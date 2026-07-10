# Runbook — Finalizar carga e validação do módulo OPEX

Estado atual: **estrutura OPEX 100% construída no servidor e versionada no repo**
(dimensões, cubos, processos, regras, feeders cross-cube). Falta apenas **carregar o
realizado de 2026/05** e **validar end-to-end**. Este runbook fecha isso.

- **Conexão:** `BR_Financeiro6` → `connection_id = bd5a09a0-8e08-4c11-b46e-91fb05295060`
- **Ambiente:** onprem. Confirme com `list_connections` antes de escrever.

## Pré-requisito — subir o CSV do realizado ao servidor

O `OPX.010.0` rejeitou porque `Base_Gastos_Realizado_2026_05.csv` **não está** no
`model_upload` do servidor (só o `Mapa_Conta_Contabil.csv` estava). O CSV já foi
gerado em `docs/contexto/Base_Gastos_Realizado_2026_05.csv` (delimitador `;`, 1
cabeçalho, 325 lançamentos + cabeçalho). Suba-o ao `model_upload` via PAX/PAW
(upload de modelo) ou pelo fluxo de blob (`zCTI.Blob.Para.ModelUpload`).

## Caminho A (oficial) — rodar o processo de carga

1. `execute_process` **`zCTI.Setup.Controle_Cargas_OPX`** (semeia os parâmetros no
   SYS.150; idempotente — já foi rodado, rode de novo se necessário).
2. `execute_process` **`OPX.010.0.CSV_para_cubo_OPX.900.Gasto_Consolidado - Carga Realizado`**
   com `{pAno:'2026', pMes:'05', pVersao:'RE'}`.
   - Esperado: **`CompletedWithMessages`** (há rejeições propositais — ver abaixo).
3. Conferir o log em `SYS.150` e as rejeições em `SYS.160` (ver Validação).

## Caminho B (fallback, sem subir arquivo) — `write_cells`

Se não for possível subir o CSV, carregue direto na fatia
`2026 / 05 / RE / <CC> / <Conta> / Realizado / $ Valor Gasto`. Regenere o payload a
partir do CSV versionado (mesma lógica de rejeição do processo — pula conta em
branco, valor não-numérico, CC/Conta inexistentes):

```python
import codecs, json, re
validCC = {'OPER0001','OPER0002','OPER0003','OPER0004','OPER0005','OPER0006',
           'SUPP0001','SUPP0002','SUPP0003','SUPP0004','SUPP0005','SUPP0006','SUPP0007'}
validConta = {'30000001','30000002','30000003'} | {str(40000000+i) for i in range(1,22)}
lines = codecs.open('docs/contexto/Base_Gastos_Realizado_2026_05.csv','r','latin-1').read().split('\n')[1:]
cells = {}
isnum = lambda s: bool(re.fullmatch(r'-?[0-9]+([.,][0-9]+)?', s))
for ln in lines:
    if not ln.strip(): continue
    cc, conta, valor, *_ = (ln.split(';') + ['',''])[:4]
    cc, conta, valor = cc.strip(), conta.strip(), valor.strip()
    if conta=='' : continue
    if valor in ('','-'): valor='0'
    if not isnum(valor) or cc not in validCC or conta not in validConta: continue
    cells[(cc,conta)] = float(valor.replace('.','').replace(',','.')) if ',' in valor else float(valor)
payload = [{'coordinates':['2026','05','RE',cc,conta,'Realizado','$ Valor Gasto'],'value':v}
           for (cc,conta),v in cells.items()]
```

Resultado esperado: **307 células**, soma **2.509.160**. Passe `payload` ao `write_cells`.

## Rejeições esperadas (armadilhas plantadas na base — não são bug)

- 1 linha com **Conta Contábil em branco** (valor 2292).
- 1 **valor não-numérico** (`e-1wq`).
- CC inexistente **`OPER0003a`** (erro de digitação).
- 1 conta fora do plano de contas.

Pelo Caminho A, essas caem em `SYS.160.Rejeitados_Cargas` (processo/usuário).

## Validação end-to-end (MDX)

1. **Total realizado por Origem** — só `Realizado` deve ter valor em RE:
   ```
   SELECT {[ALL.D.Origem_Despesas].Members} ON 0
   FROM [OPX.900.Gasto_Consolidado]
   WHERE ([ALL.D.Ano].[2026],[ALL.D.Mes].[05],[ALL.D.Versao].[RE],
          [ALL.D.Centro_Custo].[Total Centro de Custo],
          [ALL.D.Conta_Contabil].[Total Conta Contábil],
          [OPX.M.900.Gasto_Consolidado].[$ Valor Gasto])
   ```
   Esperado: `Realizado` ≈ **2.509.160** (mais o double-count, ver abaixo); demais origens = 0.

2. **Double-count no Total Conta Contábil (erro de dado a corrigir na origem):**
   `Marketing (40000020)` veio no `Mapa_Conta_Contabil.csv` com Agrupamento Nível 1 =
   `DEPRECIAÇÃO` (deveria ser `GASTOS COMERCIAIS`). Isso faz `PUBLICIDADE E MKT` ter
   **dois pais** (GASTOS COMERCIAIS e DEPRECIAÇÃO), duplicando no `Total Conta Contábil`.
   Confirme comparando `Total Conta Contábil` com a soma dos 5 Agrupamentos Nível 1:
   ```
   SELECT {[ALL.D.Conta_Contabil].[Total Conta Contábil],
           [ALL.D.Conta_Contabil].[RECEITA],[ALL.D.Conta_Contabil].[GASTOS COM PESSOAL],
           [ALL.D.Conta_Contabil].[GASTOS INDUSTRIAIS],[ALL.D.Conta_Contabil].[GASTOS COMERCIAIS],
           [ALL.D.Conta_Contabil].[DEPRECIAÇÃO]} ON 0
   FROM [OPX.900.Gasto_Consolidado]
   WHERE ([ALL.D.Ano].[2026],[ALL.D.Mes].[05],[ALL.D.Versao].[RE],
          [ALL.D.Centro_Custo].[Total Centro de Custo],[ALL.D.Origem_Despesas].[Realizado],
          [OPX.M.900.Gasto_Consolidado].[$ Valor Gasto])
   ```
   **Correção:** ajustar o `Marketing` para `GASTOS COMERCIAIS` no CSV de mapa e rodar
   `DIM.050.0` de novo (reconstrói a hierarquia sem o duplo-pai).

3. **Projeção T1 (cross-módulo)** — em um mês NÃO marcado 'Real' em `REC.000` e com
   dados de origem preenchidos, conferir que `OPX.900` T1 puxa:
   - `OPX.100` (colaborativo, mesmo CC×Conta);
   - `CAP.200` → conta `40000021` (Depreciação);
   - `FOL.100` → contas de pessoal `40000001..07` (soma Total Existente e Novo × Total Cargos);
   - `REC.100` → conta `40000018` (Frete) no CC `N/A`, com sinal invertido (custo→gasto).
   A virada Real×Orçado é herdada de REC.100/FOL.100 (que já espelham RE nos meses 'Real').

## Fora de escopo (slide 8, client-side)
Relatórios comparativos em PAX (Dynamic Report, formatação condicional Real×Orçado) e
Books de Workspace — as tools MCP do TM1 não os criam.

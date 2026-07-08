# Estilo de Rules e Feeders

Baseado em `cubes/REC.100.Receita.rules`. Toda regra nova segue este padrão.

## Cabeçalho de cada bloco (obrigatório, sem exceção)

**Toda** regra tem cabeçalho — cada bloco de cálculo **e** a seção `FEEDERS`. Regra
sem cabeçalho é tratada como defeito. O comentário é estruturado:

```
# --------------------------------------------------------------------------------------------
#Name  <nome curto do cálculo>
#Desc  <o que faz, em pt-BR — inclusive a lógica Real x Orçado quando aplicável>
#Date  <dd/mm/aaaa>
#Author <autor>
# --------------------------------------------------------------------------------------------
```

## Estrutura do arquivo

1. `SKIPCHECK;` no topo (evita varredura de células vazias; **exige** feeders).
2. Blocos de cálculo `['Medida']=N: ... ;` — sempre no nível **folha** (`N:`).
3. Seção `FEEDERS;` ao final, com um feeder para cada medida calculada.

## Convenções de lógica (padrão Real × Orçado)

- Versões dinâmicas **T1/T2**: se `REC.000.Controle_Real_Orcado` marca o mês/ano
  como `'Real'`, a célula **espelha** a versão `RE` via `DB(...)`; caso contrário,
  **calcula** pelas premissas (`REC.030`, `REC.040`, …).
- Versão `RE` e estáticas `F1..F11`: **carga direta** → use `STET` (mantém o valor
  gravado, não recalcula).

## Escopo de versão por regra de área (só dinâmicas calculam)

Em cubos multi-versão, **não** repita o gate de versão em cada medida. Coloque uma
**regra de área** no topo (logo após `SKIPCHECK;`) que restringe o cálculo às
versões dinâmicas (`T1`/`T2`, filhas do consolidado `Dynamic Versions`); `RE` e as
estáticas caem em `STET` e permanecem como **input puro**:

```
[]=IF( ELISANC( 'ALL.D.Versao', 'Dynamic Versions', !ALL.D.Versao ) <> 0,
       CONTINUE, STET );
```

- `ELISANC(dim, consolidado, elemento)` retorna `<>0` quando o elemento é
  descendente do consolidado → `CONTINUE` deixa as regras de medida abaixo agirem.
- Caso contrário (`RE` e estáticas) → `STET`: a célula é o valor gravado (carga).
- Assim as regras de medida só precisam do gate Real×Orçado (espelhar `RE` vs.
  calcular premissa), sem repetir a checagem de versão. **`RE` nunca calcula** —
  os derivados do realizado existem apenas em `T1`/`T2`.

## Referência a período anterior/seguinte — use SYS.200.Time_Travel

Contas de **saldo** e qualquer roll-forward (Saldo Inicial/Final HC, Saldos a Pagar,
Valores Unitários que herdam do mês anterior) referenciam outro período **pelo cubo
`SYS.200.Time_Travel`** — nunca por aritmética de string.

- **Proibido em regra:** `NUMBERTOSTRING` / `STRINGTONUMBER` **não existem em regras**
  (só em TI). Também não monte mês/ano com `NUMBR`/`STR`/`IF(mês='01',...)` — é frágil e
  erra a virada de ano.
- **Correto:** `SYS.200.Time_Travel` devolve o Ano/Mês resultante de um deslocamento
  (`Parametro_Mes`), já tratando a virada de ano. Ex.: para o **mês anterior**

  ```
  DB('FOL.100.Folha_Pagamento',
     DB('SYS.200.Time_Travel', !ALL.D.Ano, !ALL.D.Mes, '-1', 'Ano'),
     DB('SYS.200.Time_Travel', !ALL.D.Ano, !ALL.D.Mes, '-1', 'Mes'),
     !ALL.D.Versao, ..., 'Saldo Final HC')
  ```

  Use `'-1'` para mês anterior e `'1'` para o seguinte (nos feeders de roll). Ver
  `docs/contexto/arquitetura-modelo.md` (seção Time Travel).

## Operadores e funções idiomáticos

- `@=` compara strings; `%` é **OR** lógico; `&` é AND.
- `DB('Cubo', el1, el2, ...)` para ler outro cubo/interseção.
- `ATTRS('Dim', !Dim, 'Atributo')` para ler atributo (ex.: `Categoria` de um produto).
- `STET` mantém o valor de entrada (não sobrescreve com cálculo).
- Deduções (desconto, devolução) recebem sinal negativo explícito (`-1 * ...`).

## Feeders

- `SKIPCHECK` obriga feeders: **toda** medida `N:` que gera valor precisa alimentar
  seus destinos. Regra sem feeder = célula que não aparece em consolidação.
- Feeder deve ser tão preciso quanto possível — super-feed degrada performance;
  sub-feed causa valores faltando.
- Ao alterar um `DB()` de origem, **reveja o feeder correspondente**.
- **Feeder de versão (multi-versão):** o realizado precisa alimentar o espelho nas
  versões dinâmicas. Padrão:

  ```
  ['ALL.D.Versao':'ALL.D.Versao':'RE'] => ['ALL.D.Versao':'ALL.D.Versao':'Dynamic Versions'];
  ```

  Sem ele, as células espelhadas de `RE` em `T1`/`T2` não aparecem em consolidação.

## Antes de gravar

Rode `validate_rule` e `lint_rules` (MCP). Para regras com `DB()` cruzado entre
cubos, rode `find_circular_dependencies`. Ver `docs/guardrails.md`.

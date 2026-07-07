# Convenção de nomenclatura (fonte da verdade)

A nomenclatura é a espinha dorsal do modelo. Objetos novos **devem** segui-la;
divergências são tratadas como defeito. Derivada dos objetos existentes no repo.

## Prefixo funcional (domínio)

| Prefixo | Domínio | Exemplos |
|---|---|---|
| `ALL` | Dimensões comuns/compartilhadas | `ALL.D.Ano`, `ALL.D.Produto`, `ALL.D.Versao` |
| `CAP` | CAPEX / Investimentos | `CAP.200.Investimentos`, `CAP.M.200.Investimentos` |
| `FOL` | Folha de Pagamento | `FOL.100.Folha_Pagamento`, `FOL.010.Premissas_Gerais`, `FOL.M.100.Folha_Pagamento` |
| `MAP` | Mapas de-para (staging) | `MAP.010.Produto`, `MAP.D.Centro_Custo`, `MAP.040.Cargos` |
| `REC` | Receita e premissas | `REC.100.Receita`, `REC.030.Premissas_Produto` |
| `SYS` | Sistema / controle de carga | `SYS.150.Controle_Cargas`, `SYS.160.Rejeitados_Cargas` |

> **Prefixos de processo (TI):** além de `CUB`/`DIM` (ver seção Processos), o módulo de
> Folha usa `RH` para a carga do realizado da folha
> (`RH.020.0.CSV_para_cubo_FOL.100.Folha_Pagamento - Carga Realizado`), conforme o
> desenho do "Desafio Final – Folha".

## Tipo de dimensão

- `.D.` → dimensão de **negócio** (membros reais): `ALL.D.Estado`, `MAP.D.Produto`.
- `.M.` → dimensão de **medidas** (measures): `REC.M.100.Receita`, `CAP.M.200.Investimentos`.
- Cubos **não** levam `.D.`/`.M.` no nome; usam `PREFIXO.NNN.Nome`
  (`REC.100.Receita`, `CAP.200.Investimentos`).

## Numeração (`.NNN`)

Blocos numéricos ordenam por etapa/domínio e mantêm objetos relacionados juntos:

- `000` — controle/parametrização do domínio (ex.: `REC.000.Controle_Real_Orcado`).
- `010`–`0x0` — premissas e staging (`REC.010.Premissas_Gerais`, `MAP.010.Produto`).
- `100` — cubo transacional principal do domínio (`REC.100.Receita`).
- `150`/`160` — controle e rejeição de cargas (`SYS.150`, `SYS.160`).
- `200` — outro domínio transacional (`CAP.200.Investimentos`).

## Processos (TurboIntegrator)

Formato: **`PREFIXO.NNN.S.Descricao`**

- `PREFIXO` funcional de processo: `CUB` (carga em cubo), `DIM` (atualização de
  dimensão), `CAP`, etc.
- `NNN` — grupo/domínio; `S` — passo (ordem) dentro do grupo.
- Exemplos: `CUB.010.0.CSV_para_cubo_REC.100.Receita - Carga Realizado`,
  `DIM.010.2.cubo_MAP.010.para_dim_ALL.D.Produto`.

### Utilitários e templates — prefixo `zCTI.`

Prefixo `z` para ordenar ao fim da lista; `CTI` identifica autoria/uso interno:

- `zCTI.Template.*` — modelos reutilizáveis de carga (**reuse antes de criar**).
- `zCTI.Setup.*` — configuração de atributos/controle.
- `zCTI.Seed.*` — carga inicial de premissas.
- `zCTI.Diag.*` — diagnóstico/listagem.
- `zCTI.Blob.*` — integração de arquivos/upload de modelo.

## Elementos e medidas

- Medidas monetárias começam com `$` (`$ Receita Bruta`, `$ Desconto`).
- Percentuais começam com `%` (`% Desconto`, `% Devolução`).
- Versões: `RE` (Realizado), `T1`/`T2` (dinâmicas Real×Orçado), `F1`..`F11` (estáticas/forecast).

## Regra prática

Antes de criar um objeto, rode `validate_naming` (MCP) ou o subagente
`validador-nomenclatura`. Renomear objetos em TM1 é sensível — nomeie certo na origem.

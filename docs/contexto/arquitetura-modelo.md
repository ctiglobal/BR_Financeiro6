# Arquitetura do modelo

## Cubos

| Cubo | Papel | Medidas-chave (exemplos) |
|---|---|---|
| `REC.000.Controle_Real_Orcado` | Marca período como Real/Orçado | `Real ou Orçado` |
| `REC.010.Premissas_Gerais` | Premissas globais | — |
| `REC.020.Premissas_Estado` | Premissas por estado | — |
| `REC.030.Premissas_Produto` | Preço por produto | `$ Preço Unitário Produto` |
| `REC.040.Premissas_Canal_UF_Categ` | Deduções por canal/UF/categoria | `% Desconto`, `% Devolução` |
| `REC.100.Receita` | Receita transacional (com regras) | `Volume de Venda`, `$ Receita Bruta`, `$ Desconto`, `$ Devolução` |
| `CAP.200.Investimentos` | CAPEX e depreciação | — |
| `FOL.010.Premissas_Gerais` | Premissas de folha (encargos/reajustes) | `% Dissídio`, `% Reajuste Plano de Saúde`, `% Reajuste Vale Refeição`, `% INSS`, `% FGTS` |
| `FOL.100.Folha_Pagamento` | Folha transacional (com regras) | `Saldo Final HC`, `$ Salário`, `$ Provisão 13º Salário`, `$ Provisão Férias`, `$ INSS`, `$ FGTS`, `Total Despesas Geral` |
| `OPX.100.Gastos_Colaborativos` | OPEX colaborativo (input por CC × Conta) | `$ Valor Gasto`, `Justificativa` |
| `OPX.900.Gasto_Consolidado` | Gasto consolidado por Origem (com regras; amarra CAP/REC/FOL/OPX.100) | `$ Valor Gasto` |
| `MAP.010.Produto`, `MAP.030.Centro_Custo`, `MAP.040.Cargos`, `MAP.050.Conta_Contabil` | Staging de-para | `MAP.040`: `Descrição`, `$ Salário Base`, `$ Plano de Saúde`, `$ Vale Refeição`; `MAP.050`: `Descrição`, `Agrupamento Nível 1`, `Agrupamento Nível 2` |
| `SYS.150.Controle_Cargas` | Parâmetros de carga por processo | `Caminho do Arquivo Fonte`, `Debug (S/N)` |
| `SYS.160.Rejeitados_Cargas` | Registros rejeitados nas cargas | — |
| `SYS.200.Time_Travel` | Lookup de deslocamento de período (utilitário; ver abaixo) | `Ano`, `Mes` (string) |

## Dimensões (por prefixo)

- **`ALL.D.*`** (comuns): `Ano`, `Mes`, `Versao`, `Produto`, `Categoria`, `Estado`,
  `Canal`, `Centro_Custo`, `Cargos`, `Existente_Novo`, `Conta_Contabil`
  (hierarquia Agrupamento N1 → N2 → Conta), `Origem_Despesas`
  (`OPX.100`/`CAP.200`/`REC.100`/`FOL.100`/`Realizado`).
- **`MAP.D.*`**: dimensões de mapa com hierarquias/subsets (`Produto`, `Centro_Custo`,
  `MAP.D.040.Cargos`, `MAP.D.050.Conta_Contabil`).
- **`*.M.*`**: dimensões de medidas por cubo (`REC.M.100.Receita`,
  `CAP.M.200.Investimentos`, `OPX.M.100.Gastos_Colaborativos`,
  `OPX.M.900.Gasto_Consolidado`, `MAP.M.050.Conta_Contabil`, etc.).
- **`SYS.D.*`**: dimensões de sistema/utilitário (`SYS.D.Linha`, e do Time Travel:
  `SYS.D.Ano`, `SYS.D.Mes`, `SYS.D.Parametro_Mes`).
- **`SYS.M.*`**: medidas de controle (`SYS.M.150.Controle_Cargas`,
  `SYS.M.160.Rejeitados_Cargas`, `SYS.M.200.Time_Travel`).

## Fluxo de cálculo da Receita (regras em `REC.100`)

```
Volume de Venda ──┐
                  ├─► $ Receita Bruta = Volume × $ Preço Unitário (REC.030)
                  │
$ Desconto   = - $ Receita Bruta × % Desconto  (REC.040, por Estado/Categoria)
$ Devolução  = - $ Receita Bruta × % Devolução (REC.040, por Estado/Categoria)
```

`ATTRS('ALL.D.Produto', !produto, 'Categoria')` resolve a categoria do produto
para buscar as premissas em `REC.040`. Gate Real×Orçado em todos os cálculos via
`REC.000` (ver `docs/contexto/visao-geral.md`).

## Fluxo de cálculo da Folha (regras em `FOL.100`)

`FOL.100.Folha_Pagamento` é um único cubo usado para **Realizado** (versão `RE`,
carregado por `RH.020.0`) e **Projeção** (`T1`/`T2`, calculado). Dimensões:
`Ano × Mes × Versao × Existente_Novo × Centro_Custo × Cargos × FOL.M.100`.

- **RE (realizado):** métricas calculadas por regra — `$ Salário Base = $ Salário /
  Saldo Final HC`, Valores Unitários análogos, e `% INSS`/`% FGTS` efetivos (encargo ÷
  (Salário+13º+Férias)). Movimentação de HC travada com zero.
- **Projeção (T1/T2):** gate Real×Orçado reaproveita `REC.000`. No Orçado:
  - Salário Base / Valores Unitários: **Existentes** rolam do mês anterior; **Novos**
    partem do Mapa (`MAP.040` via atributo numérico de `ALL.D.Cargos`). Ambos aplicam
    `% Dissídio`/`% Reajuste` do mês (de `FOL.010`, sem compor).
  - `$ Salário = Salário Base × Saldo Final HC`; benefícios = Valor Unitário × HC.
  - Provisões: `13º = $ Salário/12`, `Férias = $ Salário/9`. Encargos: `INSS`/`FGTS =
    (Salário + Provisão 13º + Provisão Férias) × %` (premissa de `FOL.010`).
  - Roll de HC (`Saldo Final = Inicial + Contratações − Desligamentos`) e movimentação
    de Saldo a Pagar (`= saldo anterior + Provisão − Pagamento + Correção`).

## Utilitário Time Travel (`SYS.200.Time_Travel`)

Tabela de lookup **estática** para aritmética de período: dado um `(Ano, Mes)` base
e um deslocamento `Parametro_Mes` (de −12 a +12), devolve o `Ano`/`Mes` resultante
já tratando a virada de ano. Ambas as medidas são **string** (o `Mes` preserva o
zero à esquerda `01`, e a saída serve de coordenada em `DB(...)`).

- Dimensões: `SYS.D.Ano × SYS.D.Mes × SYS.D.Parametro_Mes × SYS.M.200.Time_Travel`.
- **Não tem regras** — os valores vêm da carga (`zCTI.Seed.Time_Travel`, do CSV em
  `model_upload/`). Re-executável (`CubeClearData` + recarga).
- Uso típico em regras de outros cubos: em vez de calcular "mês anterior" com
  `NUMBR`/`NUMBERTOSTRING`, ler o período deslocado —
  ex.: `DB('SYS.200.Time_Travel', !ALL.D.Ano, !ALL.D.Mes, '-1', 'Mes')` devolve o mês
  anterior, e `... , '-1', 'Ano')` o ano correspondente.

## Fluxo de consolidação de Gastos (regras em `OPX.900`)

`OPX.900.Gasto_Consolidado` é o cubo **agregador** do OPEX. Dimensões:
`Ano × Mes × Versao × Centro_Custo × Conta_Contabil × Origem_Despesas × OPX.M.900`.
A dimensão **Origem Despesas** dá rastreabilidade: cada gasto sabe de qual módulo veio.

- **RE (realizado):** carregado por `OPX.010.0` na origem **`Realizado`** (input; a base
  contábil não distingue origem). É a base do comparativo Real×Orçado.
- **Projeção (T1/T2):** o `$ Valor Gasto` é **calculado por regra** (não por processo),
  puxado por Origem via `DB()`:
  - **`OPX.100`** → gasto colaborativo, mesmo cruzamento CC × Conta.
  - **`CAP.200`** → `$ Depreciação` na conta **Depreciação (40000021)**, por CC.
  - **`FOL.100`** → contas de pessoal **40000001..07** (soma `Total Existente e Novo` ×
    `Total Cargos`, por CC): Salário→40000001, 13º→40000002, Férias→40000003,
    INSS→40000004, FGTS→40000005, Plano de Saúde→40000006, Vale Refeição→40000007.
  - **`REC.100`** → `$ Frete` (total Produto×Estado×Canal) na conta **Frete (40000018)**,
    no Centro de Custo **`N/A`** (REC.100 não tem CC), com **sinal invertido** (custo→gasto).
- **Feeders cross-cube:** como a projeção vem por `DB()` de outros cubos, os feeders
  moram nos **cubos de origem** (`OPX.100`/`CAP.200`/`FOL.100`/`REC.100` alimentam
  `OPX.900`) — o feeder deve nascer na célula-fonte. Para somar o Frete foram criados os
  consolidados `Total Estado` e `Total Canal` em `ALL.D.Estado`/`ALL.D.Canal`.
- **Virada Real×Orçado:** herdada dos cubos de origem — `REC.100` e `FOL.100` já espelham
  `RE` nos meses marcados 'Real' em `REC.000`; `OPX.100`/`CAP.200` são input por versão.

> Ao adicionar cubos/dimensões, atualize esta tabela na mesma mudança.

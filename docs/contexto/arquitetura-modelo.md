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
| `MAP.010.Produto`, `MAP.030.Centro_Custo`, `MAP.040.Cargos` | Staging de-para | `MAP.040`: `Descrição`, `$ Salário Base`, `$ Plano de Saúde`, `$ Vale Refeição` |
| `SYS.150.Controle_Cargas` | Parâmetros de carga por processo | `Caminho do Arquivo Fonte`, `Debug (S/N)` |
| `SYS.160.Rejeitados_Cargas` | Registros rejeitados nas cargas | — |

## Dimensões (por prefixo)

- **`ALL.D.*`** (comuns): `Ano`, `Mes`, `Versao`, `Produto`, `Categoria`, `Estado`,
  `Canal`, `Centro_Custo`, `Cargos`, `Existente_Novo`.
- **`MAP.D.*`**: dimensões de mapa com hierarquias/subsets (`Produto`, `Centro_Custo`,
  `MAP.D.040.Cargos`).
- **`*.M.*`**: dimensões de medidas por cubo (`REC.M.100.Receita`,
  `CAP.M.200.Investimentos`, etc.).
- **`SYS.M.*`**: medidas de controle (`SYS.M.150.Controle_Cargas`,
  `SYS.M.160.Rejeitados_Cargas`).

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

> Ao adicionar cubos/dimensões, atualize esta tabela na mesma mudança.

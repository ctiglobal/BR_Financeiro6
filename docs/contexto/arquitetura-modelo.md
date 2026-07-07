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
| `MAP.010.Produto`, `MAP.030.Centro_Custo` | Staging de-para | — |
| `SYS.150.Controle_Cargas` | Parâmetros de carga por processo | `Caminho do Arquivo Fonte`, `Debug (S/N)` |
| `SYS.160.Rejeitados_Cargas` | Registros rejeitados nas cargas | — |

## Dimensões (por prefixo)

- **`ALL.D.*`** (comuns): `Ano`, `Mes`, `Versao`, `Produto`, `Categoria`, `Estado`,
  `Canal`, `Centro_Custo`.
- **`MAP.D.*`**: dimensões de mapa com hierarquias/subsets (`Produto`, `Centro_Custo`).
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

> Ao adicionar cubos/dimensões, atualize esta tabela na mesma mudança.

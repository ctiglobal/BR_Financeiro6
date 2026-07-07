# Glossário

## FP&A / negócio

- **Realizado (RE)** — valores efetivamente ocorridos, carregados de arquivo.
- **Orçado** — valores planejados, calculados por premissas.
- **Virada de chave (Real × Orçado)** — mecanismo que faz o cenário mostrar
  realizado até o mês de fechamento e orçado adiante, controlado por
  `REC.000.Controle_Real_Orcado`.
- **Premissa** — parâmetro de planejamento (preço, % desconto, % devolução, etc.).
- **Fechamento gerencial** — consolidação periódica dos resultados.
- **CAPEX / OPEX** — despesa de capital / despesa operacional.
- **Receita Bruta / Deduções (Desconto, Devolução)** — bruto menos deduções = líquida.

## Versões

- `RE` — Realizado.
- `T1`, `T2` — versões dinâmicas (aplicam a virada Real × Orçado).
- `F1`..`F11` — versões estáticas (forecast/snapshots), carga direta.

## TM1 / Planning Analytics

- **Cubo** — estrutura multidimensional de dados.
- **Dimensão** — eixo do cubo; `.D.` = negócio, `.M.` = medidas.
- **Hierarquia / Subset** — agrupamento de elementos dentro de uma dimensão.
- **Regra (Rule)** — cálculo declarativo em `.rules`.
- **Feeder** — declaração que "alimenta" células calculadas para consolidação
  (necessário sob `SKIPCHECK`).
- **STET** — mantém o valor de entrada da célula (não recalcula).
- **TurboIntegrator (TI)** — linguagem de ETL do TM1 (`.ti`).
- **Chore** — agendador/orquestrador de processos.
- **Feeders órfãos / dependência circular** — patologias de regra a evitar.

## Convenções do projeto

- **`zCTI.*`** — processos utilitários/templates de autoria interna (CTI).
- **`SYS.150` / `SYS.160`** — controle de carga / rejeitados.

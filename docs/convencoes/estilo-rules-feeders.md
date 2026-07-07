# Estilo de Rules e Feeders

Baseado em `cubes/REC.100.Receita.rules`. Toda regra nova segue este padrão.

## Cabeçalho de cada bloco

Todo cálculo é precedido de um comentário estruturado:

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

## Antes de gravar

Rode `validate_rule` e `lint_rules` (MCP). Para regras com `DB()` cruzado entre
cubos, rode `find_circular_dependencies`. Ver `docs/guardrails.md`.

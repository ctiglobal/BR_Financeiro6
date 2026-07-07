# Visão geral do modelo

Modelo de planejamento de FP&A em IBM Planning Analytics / TM1, focado em
**Receita** e **CAPEX**, com virada automática entre **Realizado** e **Orçado**.

## Domínios

- **Receita (`REC.*`)** — cubo transacional `REC.100.Receita` calcula receita a
  partir de premissas (preço, desconto, devolução) e volume. A versão determina
  se o número é digitado, calculado ou espelhado do realizado.
- **CAPEX (`CAP.*`)** — `CAP.200.Investimentos` para investimentos e depreciação.
- **Premissas (`REC.010`–`REC.040`)** — parâmetros de planejamento (gerais, por
  estado, por produto, por canal/UF/categoria) que alimentam o cálculo de receita.
- **Mapas (`MAP.*`)** — staging de-para para construir/atualizar dimensões a
  partir de arquivos (Produto, Centro de Custo).
- **Sistema (`SYS.*`)** — controle de cargas (`SYS.150`) e rejeição (`SYS.160`).

## Conceito central: Real × Orçado

`REC.000.Controle_Real_Orcado` marca, por ano/mês/versão, se o período é
**'Real'** ou **'Orçado'**. As regras de `REC.100`:

- Versões dinâmicas **T1/T2**: mês marcado 'Real' → espelha a versão `RE`
  (Realizado); senão → calcula pelas premissas.
- Versão `RE` e estáticas `F1..F11`: valor de carga direta, sem recálculo.

Isso permite que o mesmo cenário mostre realizado até o mês de fechamento e
orçado/projeção daí em diante, sem intervenção manual.

## Público e idioma

Consultores e analistas de FP&A. Elementos, medidas e documentação em pt-BR;
nomes de objetos TM1 no padrão de `docs/convencoes/nomenclatura.md`.

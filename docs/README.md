# Documentação do projeto (base de conhecimento do harness)

Esta pasta é a *knowledge base* que o harness de IA consulta e mantém. O
`CLAUDE.md` na raiz aponta para cá. Mantenha estes documentos alinhados com o
código — quando divergirem, o código é a fonte da verdade e o doc deve ser
corrigido.

## Índice

- **contexto/**
  - [`visao-geral.md`](contexto/visao-geral.md) — o que o modelo faz, do ponto de vista de negócio.
  - [`arquitetura-modelo.md`](contexto/arquitetura-modelo.md) — cubos, dimensões e como se conectam.
  - [`fluxo-dados.md`](contexto/fluxo-dados.md) — ordem de cargas e cálculo.
- **convencoes/**
  - [`nomenclatura.md`](convencoes/nomenclatura.md) — padrão de nomes (fonte da verdade).
  - [`estilo-ti.md`](convencoes/estilo-ti.md) — estilo de TurboIntegrator.
  - [`estilo-rules-feeders.md`](convencoes/estilo-rules-feeders.md) — estilo de Rules/Feeders.
- **runbooks/** — procedimentos operacionais passo a passo.
- [`glossario.md`](glossario.md) — termos de FP&A e do modelo.
- [`guardrails.md`](guardrails.md) — como operar o MCP TM1 com segurança.

## Como manter

- Ao criar/alterar objetos, atualize o doc afetado na mesma mudança.
- Documentos são curtos e específicos; detalhe efêmero vai para o objeto, não aqui.

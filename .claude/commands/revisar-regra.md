---
description: Revisa um arquivo .rules específico (lógica, feeders, convenção) sem alterá-lo.
argument-hint: <caminho ou nome do cubo, ex.: REC.100.Receita>
---

Revise a regra indicada em `$ARGUMENTS`.

1. Localize o `.rules` correspondente em `cubes/` (aceite nome do cubo ou caminho).
2. Delegue ao subagente `revisor-regras-tm1`.
3. Traga o relatório de findings (bloqueante / atenção / estilo) com
   arquivo:linha e correção proposta.
4. **Não edite** o arquivo. Se eu pedir para aplicar correções depois, aí sim
   proponha o diff e siga `docs/guardrails.md` para gravar via MCP.

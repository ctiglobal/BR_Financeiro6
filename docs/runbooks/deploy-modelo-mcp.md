# Runbook — Aplicar mudança do repositório no servidor TM1 (via MCP)

> Stub guiado. O repositório Git é a fonte da verdade; o MCP aplica ao servidor vivo.

**Objetivo:** levar uma alteração de regra/processo versionada em `cubes/` ou
`processes/` para o servidor TM1, com segurança.

## Passos

1. **Prepare no Git** — faça a alteração no arquivo (`.rules`/`.ti`), revise e
   comite. O código versionado é o que será aplicado.
2. **Confirme a conexão** — `list_connections`; identifique `connection_id` e o
   **ambiente** (dev/homolog/produção). Ver [`../guardrails.md`](../guardrails.md).
3. **Capture o estado atual** — `get_cube_rules` / `get_process` do objeto alvo
   (backup do "antes").
4. **Valide** —
   - Regra: `validate_rule`, `lint_rules`, `diff_rules` (mostre o diff), e
     `find_circular_dependencies` se houver `DB()` cruzado.
   - Processo: `lint_ti`.
5. **Obtenha OK explícito** do responsável antes de escrever.
6. **Aplique** — `set_rule` / `set_process`.
7. **Verifique** — recalcule/execute em escopo pequeno; confira valores esperados
   via `execute_mdx`. Reprocessar feeders se necessário (`SDATA.ReprocessarFeeders`).
8. **Registre** — anote o que foi aplicado, em qual ambiente e por quem.

## Não fazer

- Deleções (`delete_*`) estão **negadas** por padrão — só com autorização explícita.
- Aplicar direto em produção sem o backup do passo 3.

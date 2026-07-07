# Guardrails — operando o MCP TM1 (`tm1-ide-saas`)

O MCP conecta ao **servidor TM1 ao vivo**. Editar regras, processos ou células
afeta um ambiente real, possivelmente de produção. Trate como mudança sensível.

## Sempre

1. **Comece por `list_connections`** e confirme qual `connection_id` você vai usar.
   Toda tool exige `connection_id`. Não assuma — pergunte se houver ambiguidade.
2. **Confirme o ambiente** (dev/homolog/produção) antes de qualquer escrita.
   Se não der para distinguir, **pergunte ao usuário**.
3. **Valide antes de aplicar**:
   - Regras → `validate_rule` + `lint_rules`; cheque `find_circular_dependencies`.
   - Processos → `lint_ti`.
   - Nomes → `validate_naming`.
4. **Mostre o diff** (`diff_rules` para regras) e obtenha OK explícito antes de
   `set_rule` / `set_process` / `write_cell(s)`.
5. **Versione no Git** a mudança correspondente em `cubes/`, `dimensions/` ou
   `processes/` — o repo é a fonte da verdade do modelo.

## Nunca (sem autorização explícita e ambiente confirmado)

- `delete_cube`, `delete_dimension`, `delete_hierarchy`, `delete_process`,
  `delete_rule`, `bulk_delete_elements` — **negados por padrão** em
  `.claude/settings.json`. Só habilite após confirmação inequívoca.
- Escrever em produção sem backup/versão do estado anterior.
- Executar processo/chore de carga sem saber o volume e o destino.

## Padrões operacionais

- **Leitura** (`get_*`, `list_*`, `execute_mdx`, `read_cellset`) é livre — use à
  vontade para investigar antes de propor mudanças.
- **Cargas longas**: `execute_process_async` + `poll_async`; não bloqueie.
- **Escopo de escrita**: prefira a menor mudança que resolve; nada de "refactor"
  amplo sem pedido.

## Mapa de permissões (`.claude/settings.json`)

- `allow`: leitura, lint e validação.
- `ask`: toda escrita/execução (`set_rule`, `set_process`, `execute_*`, `write_*`).
- `deny`: todas as deleções.

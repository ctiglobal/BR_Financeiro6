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
   `processes/`. O **servidor ao vivo é a fonte viva**; o repo é o **registro
   versionado e o cache de leitura** do modelo (neste projeto o objeto nasce no
   servidor via MCP e o repo o segue). Em conflito repo × servidor, **o servidor
   vence** — e isso é gatilho para re-sincronizar o repo.

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

## Ordem de consulta de metadados (repo-first, MCP autoridade)

Aplica-se a **metadados/estrutura** — regras (`.rules`), processos (`.ti`/`.json`),
dimensões e cubos. **Não** se aplica a **dados** (sempre MCP).

| Situação | Fonte |
|---|---|
| Orientação / leitura estrutural (dimensões de um cubo, lógica de uma regra, processos existentes, conferir nomes contra `nomenclatura.md`) | **Repo primeiro** (`Read`/`Glob`/`Grep`); MCP só se não achar |
| **Antes de qualquer escrita** no servidor (`set_rule`/`set_process`/`add_edges_bulk`/`upsert_*`) | **MCP** — confirmar estado atual/etag; o repo pode estar atrás |
| Validação que **decide uma mutação** (existência de elemento antes de criar edge; `validate_rule`/`lint_ti` antes de aplicar) | **MCP** (roda no servidor de qualquer forma) |
| Atributos (alias/numéricos), views, segurança, sort, subsets privados | **MCP** — o formato `tm1project` não versiona isso |
| Dados (células, saldos) | **MCP** |

Regra de ouro: **o repo é cache de leitura, não a verdade.** Se o arquivo contradiz
um retorno/erro do servidor, o **servidor vence** — e re-sincronize o repo. "Não
encontrei no repo" é seguro (cai no MCP); o caso perigoso é "encontrei porém
desatualizado", que *parece* autoritativo. Contra a defasagem silenciosa (não há
sync automático; o objeto nasce no servidor), rode a reconciliação estrutural —
ver `docs/runbooks/reconciliar-modelo.md` — antes de confiar no repo para algo
importante, ou periodicamente.

## Mapa de permissões (`.claude/settings.json`)

- `allow`: leitura, lint/validação **e escrita** (`set_rule`, `set_process`,
  `write_cell(s)`, `create_*`, `upsert_*`, `add_edges_bulk`,
  `execute_process(_async)`, `execute_chore`).
- `ask`: `delete_element`, `delete_cellset` (pedem confirmação).
- `deny`: deleções estruturais (`delete_cube`, `delete_dimension`,
  `delete_hierarchy`, `delete_process`, `delete_rule`, `bulk_delete_elements`).

> Escrita auto-aprovada no nível de permissão **não dispensa** o item 4 do
> *Sempre*: validar (`validate_rule`/`lint_ti`) e revisar o diff continua sendo
> disciplina obrigatória, ainda mais em produção.

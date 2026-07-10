# Runbook — Reconciliar modelo (repo ↔ servidor)

## Objetivo

Medir e corrigir a **defasagem estrutural** entre o repositório Git (`tm1project`)
e o servidor TM1 ao vivo. Neste projeto o objeto **nasce no servidor** (via MCP) e
o repo o segue — então o repo pode ficar atrás sem aviso. Rode isto **antes de
confiar no repo para algo importante** (política repo-first em `docs/guardrails.md`)
ou periodicamente.

## Escopo

Só **metadados/estrutura**: cubos, dimensões, hierarquias, processos (`.ti`/`.json`)
e regras (`.rules`). **Não** cobre dados (células/saldos). **Não** conta como drift o
que o formato `tm1project` não versiona: valores de atributo (alias/numéricos),
views, segurança, ordem de sort, subsets privados.

## Passos

1. **Conexão** — `list_connections` → confirme o `connection_id` e o ambiente
   (dev/homolog/produção).

2. **Inventário (existência)** — compare as listas dos dois lados:
   - Servidor: `list_cubes`, `list_dimensions`, `list_processes`.
   - Repo: `cubes/*.json`, `dimensions/*.json`, `processes/*.json` (via `Glob`).
   - **No servidor e não no repo** → faltando: versionar (ver
     `docs/runbooks/criar-modulo-mcp.md` para o formato dos artefatos).
   - **No repo e não no servidor** → órfão: investigar (objeto deletado no
     servidor? arquivo obsoleto?). Não apague nada sem confirmar.

3. **Estrutura (para os que existem nos dois lados)**:
   - **Cubo**: dimensões via `get_cube` × lista `Dimensions` do `.json`.
   - **Dimensão**: elementos/tipos via `list_elements`; edges de consolidação via
     MDX `{[dim].[consolidado].Children}` × `Elements`/`Edges` do
     `<dim>.hierarchies/<dim>.json`.
   - **Regras**: `diff_rules` (servidor × arquivo `.rules`).
   - **Processo**: `get_process` × `.ti`/`.json` (4 tabs em regiões
     `#region Prolog/Metadata/Data/Epilog`; datasource, parâmetros, variáveis).

4. **Relatório de drift** — liste por objeto: faltando / órfão / divergente, com o
   detalhe concreto (dimensão a mais, edge ausente, regra diferente).

5. **Re-sincronizar** — **o servidor vence** (é a fonte viva). Atualize os arquivos
   do repo para refletir o servidor e commite. Se, em vez disso, o **servidor**
   estiver errado, corrija no servidor via MCP (validando antes — `docs/guardrails.md`)
   e só então versione.

## Observações

- "Não encontrei no repo" é seguro (cai no MCP). O caso perigoso é
  "encontrei porém desatualizado", que *parece* autoritativo — por isso a
  reconciliação existe.
- O caminho autoritativo de sincronização contínua é o **export nativo do TM1
  IDE / integração Git**; este runbook é a checagem manual quando isso não roda.
- Pode ser promovido a comando `/reconciliar-modelo` em `.claude/commands/` se
  virar rotina.

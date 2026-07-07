# CLAUDE.md — Harness de IA do projeto

> Ponto de entrada do *Agent Harness*. O Claude Code carrega este arquivo
> automaticamente em toda sessão. Mantenha-o curto e aponte para `docs/` no que
> for detalhe. Se algo aqui divergir do código, o **código vence** — corrija este
> arquivo.

## O que é este repositório

Modelo **IBM Planning Analytics / TM1** de FP&A, versionado em Git no formato
`tm1project`. Cobre **Receita** (com virada Real × Orçado), **CAPEX/Investimentos**,
**Premissas** de planejamento e a infraestrutura de **cargas** (mapas, dimensões,
controle e rejeição).

Estrutura do modelo (não editar à toa — são artefatos gerados/versionados do TM1):

- `cubes/` — cubos (`.json`) e regras (`.rules`)
- `dimensions/` — dimensões, hierarquias (`.hierarchies`) e subsets (`.subsets`)
- `processes/` — TurboIntegrator: metadados (`.json`) + código (`.ti`)
- `tm1project.json` — manifesto do projeto (lista `Ignore` / `Files`)

## Como trabalhar aqui (regras de ouro)

1. **Leia `docs/convencoes/nomenclatura.md` antes de criar qualquer objeto.** A
   nomenclatura é a espinha dorsal do modelo e é validada.
2. **Nunca invente sintaxe TM1/MDX/TI.** Se não tiver certeza de uma função,
   assinatura ou comportamento por versão, diga que não sabe e verifique. Fabricar
   é o pior resultado possível.
3. **`.rules`** seguem `docs/convencoes/estilo-rules-feeders.md` (cabeçalho
   `#Name/#Desc/#Date/#Author`, `SKIPCHECK`/`FEEDERS`, `STET`).
4. **`.ti`** seguem `docs/convencoes/estilo-ti.md` (prefixos `c`/`p`, framework de
   carga via `SYS.150`/`SYS.160`, `ItemReject` para erros).
5. **Mudança no modelo vivo (via MCP TM1)** passa por
   `docs/guardrails.md` — sempre `validate_rule`/`lint_ti` antes de aplicar, e
   confirme `connection_id`/ambiente antes de escrever.
6. Responda e comente **em português** (pt-BR). Mantenha nomes de objetos
   TM1/SAP no idioma original.

## Mapa da documentação (`docs/`)

| Preciso de… | Arquivo |
|---|---|
| Visão geral do negócio/modelo | `docs/contexto/visao-geral.md` |
| Como os cubos se conectam | `docs/contexto/arquitetura-modelo.md` |
| Ordem de execução das cargas | `docs/contexto/fluxo-dados.md` |
| Regras de nomenclatura | `docs/convencoes/nomenclatura.md` |
| Estilo de TurboIntegrator | `docs/convencoes/estilo-ti.md` |
| Estilo de Rules/Feeders | `docs/convencoes/estilo-rules-feeders.md` |
| Termos FP&A e do modelo | `docs/glossario.md` |
| Segurança ao operar o MCP TM1 | `docs/guardrails.md` |
| Procedimentos operacionais | `docs/runbooks/` |

## Ferramentas de IA disponíveis

- **Subagentes** (`.claude/agents/`): `revisor-regras-tm1`, `autor-ti`,
  `validador-nomenclatura`.
- **Comandos** (`.claude/commands/`): `/validar-modelo`, `/revisar-regra`,
  `/documentar-processo`.
- **MCP `tm1-ide-saas`**: inspeção/edição do servidor TM1 ao vivo. Comece sempre
  por `list_connections` para obter o `connection_id`. Leia `docs/guardrails.md`.

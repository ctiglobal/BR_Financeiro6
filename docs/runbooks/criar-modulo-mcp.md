# Runbook — Criar um módulo novo end-to-end (via MCP)

> Procedimento para construir um domínio novo (dims → cubos → processos → regras →
> carga → validação) ao vivo no servidor TM1, versionando no Git. Derivado da
> construção do módulo **Folha** (`FOL.*`) em 07/07/2026. Os módulos ainda pendentes
> (Opex, Custeio/Estoques, DRE) seguem este mesmo caminho.

**Objetivo:** entregar um módulo de FP&A completo, seguindo os padrões do modelo
(nomenclatura, framework de carga `SYS.150`/`SYS.160`, gabarito de regras) sem
inventar sintaxe e sem retrabalho pelas armadilhas conhecidas do MCP.

## Pré-requisitos

- Ler [`../convencoes/nomenclatura.md`](../convencoes/nomenclatura.md) **antes** de
  nomear qualquer objeto — a nomenclatura é validada e renomear em TM1 é sensível.
- Ter o **desenho do módulo** (slides/planilha) com a lista de medidas, dimensões e
  o racional de cálculo. **Não inventar fórmulas** — extrair da fonte.
- Confirmar `connection_id` e **ambiente** (`list_connections`) — ver
  [`../guardrails.md`](../guardrails.md). `whoami` para nível de permissão.
- Reusar sempre que possível: dims comuns `ALL.D.*`, templates `zCTI.Template.Carga.*`,
  gate Real×Orçado `REC.000.Controle_Real_Orcado`.

## Ordem de construção

Construa de baixo para cima — dependências primeiro. Valide cada camada antes da
seguinte.

1. **Dimensões de negócio** (`ALL.D.*` novas e `MAP.D.*` do mapa) e **de medida**
   (`*.M.*`). Criar o pai consolidado como **stub** e ligar componentes depois
   (ver Armadilhas). Definir atributos (alias, atributos numéricos) já aqui.
2. **Cubos** — mapa (`MAP.NNN.*`), premissas (`PREFIXO.010.*`) e o transacional
   (`PREFIXO.100.*`). O transacional é **único** para Realizado (`RE`, carregado) e
   Projeção (`T1`/`T2`, calculado por regra).
3. **Processos de dimensão** (`DIM.NNN.*`): `.1` CSV→cubo mapa (valida numérico,
   rejeita ao `SYS.160`); `.2` mapa→dimensão (+ atributos + alias); `.0` orquestrador.
   Registrar em `SYS.150` via um `zCTI.Setup.Controle_Cargas_*`.
4. **Seed de premissas** (`zCTI.Seed.Premissas_*`) — popular `PREFIXO.010.*`.
5. **Processo de carga do realizado** (`PREFIXO.NNN.0`) — CSV → transacional, versão
   `RE`. A partir de `zCTI.Template.Carga.Arquivo.Para.Cubo`.
6. **Regras** do transacional — ver [`../convencoes/estilo-rules-feeders.md`](../convencoes/estilo-rules-feeders.md):
   `SKIPCHECK` (obrigatório, **nunca** omitir), regra de área de escopo de versão
   (`ELISANC`), gate Real×Orçado por medida, `FEEDERS` (incl. feeder de versão
   `RE => Dynamic Versions`).
7. **Validação end-to-end** (abaixo) e **commit** no Git.

## Validação end-to-end

1. `execute_process` do orquestrador de dimensão → conferir cubo mapa populado e a
   dimensão com atributos/alias (`get_dimension`, `read_cellset`); rejeições no
   `SYS.160`.
2. `execute_process` da carga do realizado para um `<Ano>_<Mês>` de teste → conferir
   o transacional (versão `RE`) via `execute_mdx`.
3. `execute_mdx` validando as métricas calculadas do realizado (coerência entre
   base × HC ≈ total; percentuais efetivos plausíveis).
4. Ligar uma versão a Orçado em `REC.000` e conferir a projeção `T1`/`T2`.
5. `validate_rule`, `lint_rules`, `find_circular_dependencies`, `validate_naming`.

## Armadilhas conhecidas do MCP (custaram tempo — não repetir)

| Sintoma | Causa | Contorno |
|---|---|---|
| `upsert_element`/`bulk_upsert_elements` falha ao criar consolidado (`'name'`) | tool não cria consolidado com componentes de uma vez | criar o pai como stub `Consolidated` **sem** componentes e depois `add_edges_bulk` |
| `write_cell`/`write_cells` retorna `'0' can not be found` | falha em cubo com muitas dimensões (ex.: 7) | gravar via `CellPutN` dentro de um TI (`zCTI.Diag.*` descartável) |
| Orquestrador de dimensão **aborta** e desfaz atributos já gravados | `ProcessError` no orquestrador faz *rollback* do subprocesso | **não** usar `ProcessError`; só `nRet=ExecuteProcess(...)`. `HasMinorErrors` é o resultado correto quando há rejeições |
| Regra "member not found" em medida com acento | nome da medida na regra sem acento (`% Dissidio` vs `% Dissídio`) | preservar acentos exatos dos nomes de elemento |
| lint acusa `WHILE_NO_END`; `validate_naming` embutido acusa desvio | falso-positivo neste modelo | desconsiderar (confirmar manualmente) |
| Célula de projeção fica `Null` mesmo com input e regra aplicada | feeder ausente/impreciso sob `SKIPCHECK` | revisar feeders (feeder de versão + cross-mês); validar no Architect/PAX |

## Não fazer

- Omitir `SKIPCHECK` de qualquer regra (regra do projeto — sempre com feeders).
- Inventar sintaxe/funções TM1/MDX/TI — na dúvida, verificar antes de aplicar.
- Editar o CSV-fonte para "limpar" dados corrompidos — a carga **rejeita** ao
  `SYS.160`; corrigir na origem do dado, não no arquivo.
- Deleções (`delete_*`) sem autorização explícita — ver [`../guardrails.md`](../guardrails.md).

## Ao terminar

- Atualizar na **mesma mudança**: [`../contexto/arquitetura-modelo.md`](../contexto/arquitetura-modelo.md),
  [`../contexto/fluxo-dados.md`](../contexto/fluxo-dados.md) e
  [`../convencoes/nomenclatura.md`](../convencoes/nomenclatura.md) se houver prefixo novo.
- Commitar `cubes/`, `dimensions/`, `processes/` e as docs. **Exportar os artefatos
  `.json`/`.ti`** dos objetos criados ao vivo — não deixar só a `.rules` no repo.

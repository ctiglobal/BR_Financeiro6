# Runbook — Carga de Receita Realizada

> Stub guiado. Preencha os `TODO` com os valores reais do seu ambiente na primeira
> execução assistida.

**Objetivo:** carregar receita realizada de um CSV para `REC.100.Receita` (versão `RE`).

**Processo:** `CUB.010.0.CSV_para_cubo_REC.100.Receita - Carga Realizado`

## Pré-requisitos

- Dimensões atualizadas (rodar antes o grupo `DIM.010.*` — ver
  [`../contexto/fluxo-dados.md`](../contexto/fluxo-dados.md)).
- Parâmetros configurados em `SYS.150.Controle_Cargas` para o processo:
  - `Caminho do Arquivo Fonte` = TODO
  - `Debug (S/N)` = `N` (produção)
- Arquivo CSV disponível no caminho acima. Layout esperado: TODO (colunas).

## Passos

1. Confirme o `connection_id` e o **ambiente** (`list_connections`). Ver
   [`../guardrails.md`](../guardrails.md).
2. Valide o CSV (encoding, separador, cabeçalho).
3. Execute o processo com os parâmetros `pFile` = TODO, `pMes` = `MM`.
   - Cargas grandes: `execute_process_async` + `poll_async`.
4. Verifique rejeições em `SYS.160.Rejeitados_Cargas`.
5. Confira totais em `REC.100.Receita` (versão `RE`) via `execute_mdx`.

## Rollback

- TODO: procedimento de reversão (ex.: zerar a fatia `RE`/mês antes de recarregar).

## Erros comuns

- Caminho vazio / arquivo inexistente → `ItemReject` no Prolog (ver
  [`../convencoes/estilo-ti.md`](../convencoes/estilo-ti.md)).
- Elemento inexistente na dimensão → rodar atualização de dimensão antes.

# Fluxo de dados e ordem de execução

Descreve a sequência típica de cargas. A numeração dos processos (`NNN.S`) reflete
a ordem: **grupo** (`NNN`) e **passo** dentro do grupo (`S`).

## 1. Atualização de dimensões a partir de mapas

Antes de carregar transações, as dimensões de negócio são reconstruídas a partir
dos mapas de-para:

- **Produto** — grupo `DIM.010.*`:
  1. `DIM.010.0.Inicio_Atualizacao_Mapa_e_Dimensoes_Produto` (orquestra)
  2. `DIM.010.1.CSV_para_cubo_MAP.010.Produto` (arquivo → cubo mapa)
  3. `DIM.010.2.cubo_MAP.010.para_dim_ALL.D.Produto` (mapa → dimensão)
  4. `DIM.010.3.cubo_MAP.010.para_dim_ALL.D.Categoria`
- **Centro de Custo** — grupo `DIM.030.*` (análogo, inclui hierarquias `Hier2`).
- **Cargos** — grupo `DIM.040.*`:
  1. `DIM.040.0.Inicio_Atualizacao_Mapa_e_Dimensoes_Cargos` (orquestra)
  2. `DIM.040.1.CSV_para_cubo_MAP.040.Cargos` (arquivo → cubo mapa; valida numéricos,
     rejeita linhas corrompidas para `SYS.160`)
  3. `DIM.040.2.cubo_MAP.040_para_dim_ALL.D.Cargos` (mapa → dimensão + atributos
     numéricos + alias `Código e Descrição`)

## 2. Carga de transações

- **Receita realizada** — `CUB.010.0.CSV_para_cubo_REC.100.Receita - Carga Realizado`
  (CSV → `REC.100`, versão `RE`).
- **CAPEX/Depreciação** — `CAP.020.0.CSV_para_cubo_CAP.200_Depreciacao`.
- **Folha realizada** — `RH.020.0.CSV_para_cubo_FOL.100.Folha_Pagamento - Carga Realizado`
  (CSV `Base_Folha_Pagamento_Realizada_<Ano>_<Mês>` → `FOL.100`, versão `RE`,
  `Existente_Novo = Existentes`).

Todas as cargas leem parâmetros de `SYS.150.Controle_Cargas` e gravam rejeições
em `SYS.160.Rejeitados_Cargas`.

## 3. Premissas e setup

- `zCTI.Seed.Premissas` — carga inicial de premissas de Receita.
- `zCTI.Seed.Premissas_FOL` — carga inicial de premissas de Folha (`FOL.010`;
  `% INSS`/`% FGTS` default 20%/8%).
- `zCTI.Setup.*` — atributos (Produto, Centro de Custo), controle de cargas
  (inclui `zCTI.Setup.Controle_Cargas_FOL`), controle Real/Orçado.

## 4. Cálculo e recálculo

- A receita orçada é **calculada por regras** (não por processo) a partir das
  premissas — ver `docs/contexto/arquitetura-modelo.md`.
- `SDATA.ReprocessarFeeders` — reprocessa feeders quando necessário.
- `Publicar_views_privadas` — publica views privadas.

> Confirme a ordem real olhando os chores no servidor (`list_chores` via MCP)
> antes de automatizar. Esta página descreve o padrão inferido dos nomes.

# Estilo de TurboIntegrator (.ti)

Baseado em `processes/CUB.010.0.CSV_para_cubo_REC.100.Receita - Carga Realizado.ti`
e nos templates `zCTI.Template.*`.

## Prefixos de variável

- `c*` — constantes/variáveis locais: `cCuboDestino`, `cUsuario`, `cContadorRegistro`.
- `p*` — parâmetros do processo: `pMes`, `pFile`.
- Nomes descritivos em pt-BR/CamelCase (`cCaminhoArquivoFonte`, `cMensagemProcesso`).

## Regiões

Organize o código em `#region Prolog / Metadata / Data / Epilog` (e feche com
`#endregion`). Toda a validação de entrada vai no **Prolog**.

## Framework de carga (padrão do projeto)

Cargas CSV→cubo usam a infraestrutura de controle:

- **Parâmetros** lidos de `SYS.150.Controle_Cargas` por processo:
  `Caminho do Arquivo Fonte`, `Debug (S/N)`, etc. — via `CellGetS(...)`.
- **Rejeitados** gravados em `SYS.160.Rejeitados_Cargas` (cabeçalho + registros).
- **Metadados de execução**: `cUsuario = TM1User()` (ou `'Executed by Chore'`),
  `cHorarioInicio = Now()`, `cProcesso = GetProcessName()`.
- **Contadores**: `cContadorRegistro`, `cContadorRejeitados`.

## Tratamento de erro (obrigatório no Prolog)

```
IF( cCaminhoArquivoFonte @= '' );
   cTipoErro = 1;
   cMensagemProcesso = 'O caminho do arquivo fonte não foi informado no cubo: ' | cCuboParametro;
   DataSourceType = 'NULL';
   ItemReject( cMensagemProcesso );
ENDIF;
```

- Normalize o mês: `pMes = FILL('0', 2 - LONG(Trim(pMes))) | Trim(pMes);`
- Valide existência do arquivo com `FileExists(...)` antes de processar.
- Em erro fatal: `DataSourceType = 'NULL'` + `ItemReject(msg)` com mensagem em pt-BR.
- Garanta atributos de rejeição (`AttrInsert` em `}ElementAttributes_SYS.M.160...`).

## Reuso

Antes de escrever do zero, escolha o template mais próximo:

| Caso | Template |
|---|---|
| Arquivo → cubo | `zCTI.Template.Carga.Arquivo.Para.Cubo` |
| Arquivo → cubo de mapa | `zCTI.Template.Carga.Arquivo.Para.Cubo.Mapa` |
| Cubo mapa → dimensão (balanceada) | `zCTI.Template.Carga.Cubo.Mapa.Para.Dimensao.Balanceada` |
| Cubo mapa → dimensão (desbalanceada) | `zCTI.Template.Carga.Cubo.Mapa.Para.Dimensao.Desbalanceada` |
| Cubo → cubo | `zCTI.Template.Carga.Cubo.Para.Cubo` |

## Antes de gravar

Rode `lint_ti` (MCP). Nunca invente funções TI — em dúvida sobre assinatura,
sinalize e verifique. Cargas longas: prefira `execute_process_async`.

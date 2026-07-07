---
description: Auditoria de sanidade do modelo — nomenclatura, regras, feeders e dependências circulares.
---

Faça uma auditoria de sanidade do modelo TM1 deste repositório. Execute em paralelo
onde possível e produza um relatório único, ordenado por severidade.

1. **Nomenclatura** — delegue ao subagente `validador-nomenclatura` sobre `cubes/`,
   `dimensions/` e `processes/`.
2. **Regras/feeders** — delegue ao subagente `revisor-regras-tm1` sobre todos os
   arquivos `.rules` em `cubes/`.
3. **Dependências circulares** — se houver conexão ativa, rode
   `find_circular_dependencies` (confirme o `connection_id` antes).
4. **Consistência tm1project** — confira que objetos em `Ignore` (`tm1project.json`)
   não são referenciados por processos versionados.

Ao final, resuma: nº de findings por severidade e os 3 itens mais urgentes.
Se o argumento `$ARGUMENTS` indicar um escopo (ex.: só `REC.*`), limite a auditoria a ele.

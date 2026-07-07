---
description: Gera documentação legível de um processo TI (objetivo, parâmetros, fontes, destino, regras de rejeição).
argument-hint: <nome do processo, ex.: CUB.010.0.CSV_para_cubo_REC.100.Receita>
---

Documente o processo TurboIntegrator indicado em `$ARGUMENTS`.

1. Leia o `.ti` (e o `.json` de metadados) em `processes/`.
2. Produza um resumo estruturado:
   - **Objetivo** — o que o processo faz, em 1-2 frases.
   - **Parâmetros** — cada `p*` com significado e exemplo.
   - **Fonte de dados** — arquivo/cubo de origem; parâmetros lidos de `SYS.150`.
   - **Destino** — cubo/dimensão alvo.
   - **Regras de rejeição** — condições que geram `ItemReject` / gravam em `SYS.160`.
   - **Dependências** — outros processos, cubos ou chores encadeados.
3. Escreva em pt-BR, claro para um analista de FP&A que não leu o código.
4. Se eu pedir, salve em `docs/runbooks/` no formato dos runbooks existentes.

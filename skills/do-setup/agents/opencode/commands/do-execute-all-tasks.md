---
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em cada uma, com limpeza de contexto entre elas.
agent: do-execute-all-tasks
subtask: true
---

Execute as tasks do PRD conforme os argumentos abaixo.

## Argumentos recebidos

$ARGUMENTS

Espera-se:
1. **Caminho do `tasks.md`** (obrigatório). Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** (opcional, default `all`):
   - `all` / `todas` → todas as pendentes
   - Lista de IDs separados por vírgula (ex.: `1.0,2.0,5.0`)
   - Range (ex.: `1.0-4.0`)

Se o caminho não for fornecido nos argumentos, **pergunte uma única vez** antes de prosseguir.

Siga o procedimento do agente na íntegra: descoberta inicial → montagem da fila → loop por task com `do-execute-task` → verificação de conclusão → limpeza de contexto entre tasks → resumo final.

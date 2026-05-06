---
mode: agent
description: Itera sequencialmente sobre tasks de um PBI executando do-execute-task em cada uma, com limpeza de contexto entre elas.
---

# /execute-all-tasks

Ative imediatamente o comportamento definido no chat mode **`execute-all-tasks`** (arquivo `.github/chatmodes/execute-all-tasks.chatmode.md`) e siga o procedimento dele na íntegra para esta conversa.

## Entrada do usuário

A solicitação completa está em `${input:request}` (ou no texto que acompanhou o comando). Espera-se:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `pbis/<nome-do-pbi>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se faltar o caminho do `tasks.md`, **pergunte uma única vez** antes de prosseguir.

## Procedimento

1. Leia `.github/chatmodes/execute-all-tasks.chatmode.md` na íntegra para carregar o procedimento oficial.
2. Aplique-o à entrada recebida:
   - Descoberta inicial (montagem da fila ordenada).
   - Loop de execução task a task, delegando cada uma à skill `do-execute-task` (lendo seu `SKILL.md`).
   - Verificação de conclusão a cada task.
   - **Limpeza de contexto** obrigatória entre tasks.
   - Encerramento com resumo final.
3. Respeite **todas** as regras invioláveis do chat mode (uma task por vez, ordem numérica, falha = parada, nunca editar `tasks.md` direto, etc.).

## Exemplo de uso

```
/execute-all-tasks pbis/<nome-do-pbi>/tasks/tasks.md 1.0-4.0
/execute-all-tasks pbis/<nome-do-pbi>/tasks/tasks.md all
/execute-all-tasks pbis/<nome-do-pbi>/tasks/tasks.md 1.0,3.0,5.0
```

# /execute-all-tasks

Itera sequencialmente sobre tasks de um PBI executando `do-execute-task` em cada uma, com limpeza de contexto entre elas.

## Argumentos

Espera-se na mensagem que acompanhar o comando:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `pbis/<nome-do-pbi>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se faltar o caminho do `tasks.md`, **pergunte uma única vez** antes de prosseguir.

## Procedimento

1. Carregue a regra **`execute-all-tasks`** (`.cursor/rules/execute-all-tasks.mdc`) e siga o procedimento dela na íntegra.
2. Aplique-o à entrada recebida:
   - Descoberta inicial (montagem da fila ordenada).
   - Loop de execução task a task, delegando cada uma à skill `do-execute-task` (lendo seu `SKILL.md`).
   - Verificação de conclusão a cada task.
   - **Limpeza de contexto** obrigatória entre tasks.
   - Encerramento com resumo final.
3. Respeite **todas** as regras invioláveis da rule (uma task por vez, ordem numérica, falha = parada, nunca editar `tasks.md` direto, etc.).

## Exemplos de uso

```
/execute-all-tasks pbis/<nome-do-pbi>/tasks/tasks.md 1.0-4.0
/execute-all-tasks pbis/<nome-do-pbi>/tasks/tasks.md all
/execute-all-tasks pbis/<nome-do-pbi>/tasks/tasks.md 1.0,3.0,5.0
```

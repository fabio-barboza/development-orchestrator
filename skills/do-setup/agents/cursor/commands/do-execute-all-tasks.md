# /do-execute-all-tasks

Itera sequencialmente sobre tasks de um PRD executando `do-execute-task` em cada uma, com limpeza de contexto entre elas.

## Argumentos

Espera-se na mensagem que acompanhar o comando:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se faltar o caminho do `tasks.md`, **pergunte uma única vez** antes de prosseguir.

## Procedimento

Use a **Task tool** para invocar o subagente `agent-execute-all-tasks` (definido em `.cursor/agents/agent-execute-all-tasks.md`) passando:
- O caminho exato do `tasks.md`
- O filtro de tasks
- Reforço de que deve seguir o procedimento do agente na íntegra (descoberta → loop → limpeza de contexto entre tasks → encerramento)

**Não** execute o procedimento você mesmo — apenas delegue ao subagente. O subagente é quem orquestra com contexto isolado.

Após o retorno do subagente, repasse o resumo final ao usuário sem alterações.

## Exemplos de uso

```
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0-4.0
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md all
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0,3.0,5.0
```

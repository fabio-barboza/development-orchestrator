---
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em cada uma, com isolamento de contexto entre elas via subagente.
agent: execute-all-tasks
subtask: true
---

# /execute-all-tasks

Delegue **integralmente** a execução para o subagente `execute-all-tasks` (definido em `.opencode/agents/execute-all-tasks.md`) usando o `task tool`.

## Argumentos recebidos

Espera-se:
1. **Caminho do `tasks.md`** (obrigatório). Ex.: `prds/prd-<nome>/tasks/tasks.md`.
2. **Filtro de tasks** (opcional, default `all`):
   - `all` / `todas` → todas as pendentes
   - Lista de IDs separados por vírgula (ex.: `1.0,2.0,5.0`)
   - Range (ex.: `1.0-4.0`)

Se o usuário não tiver passado um caminho válido, **pergunte uma única vez** antes de invocar o subagente.

## Procedimento

1. Faça o parse dos argumentos extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Invoque o subagente `execute-all-tasks` via `task tool` com:
   - `subagent`: `execute-all-tasks`
   - `prompt`: instrução clara contendo:
     - Caminho exato do `tasks.md`
     - Filtro de tasks
     - Reforço de que deve seguir o procedimento do agente na íntegra (descoberta → loop → limpeza de contexto entre tasks → encerramento)
3. **Não** execute o procedimento você mesmo — apenas delegue. O subagente roda em sessão filho isolada e é quem orquestra.
4. Após o retorno do subagente, repasse o resumo final ao usuário sem alterações.

## Exemplo

Usuário digita:
```
/execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0-4.0
```

Você invoca via task tool:
```
task(
  subagent="execute-all-tasks",
  prompt="Execute as tasks de prds/prd-login-google/tasks/tasks.md no range 1.0-4.0,
  seguindo integralmente o procedimento do agente: descoberta inicial, loop por task
  com do-execute-task, verificação de conclusão, limpeza de contexto entre tasks e resumo final."
)
```

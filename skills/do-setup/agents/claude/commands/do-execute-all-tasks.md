---
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em cada uma, com limpeza de contexto entre elas.
argument-hint: <caminho/para/tasks.md> [all | <ID,ID,...> | <ID-inicio>-<ID-fim>]
allowed-tools: Task, Read, Write, Edit, Glob, Grep, Bash, SlashCommand
---

# /do-execute-all-tasks

Delegue **integralmente** a execução para o subagente `agent-execute-all-tasks` (definido em `.claude/agents/agent-execute-all-tasks.md`) usando a tool **Task**.

## Argumentos recebidos

`$ARGUMENTS`

Espera-se:
1. **Caminho do `tasks.md`** (obrigatório). Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** (opcional, default `all`):
   - `all` / `todas` → todas as pendentes
   - Lista de IDs separados por vírgula (ex.: `1.0,2.0,5.0`)
   - Range (ex.: `1.0-4.0`)

Se o usuário não tiver passado um caminho válido em `$ARGUMENTS`, **pergunte uma única vez** antes de invocar o subagente.

## Procedimento

1. Faça o parse de `$ARGUMENTS` extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Invoque a tool **Task** com:
   - `subagent_type`: `agent-execute-all-tasks`
   - `description`: `Executar tasks do PRD`
   - `prompt`: instrução clara contendo:
     - Caminho exato do `tasks.md`
     - Filtro de tasks
     - Reforço de que deve seguir o procedimento do agente na íntegra (descoberta → loop → limpeza de contexto entre tasks → encerramento)
3. **Não** execute o procedimento você mesmo — apenas delegue. O subagente é quem orquestra.
4. Após o retorno do subagente, repasse o resumo final ao usuário sem alterações.

## Exemplo

Usuário digita:
```
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0-4.0
```

Você invoca:
```
Task(
  subagent_type="agent-execute-all-tasks",
  description="Executar tasks do PRD",
  prompt="Execute as tasks de prds/<nome-do-prd>/tasks/tasks.md no range 1.0-4.0, seguindo integralmente o procedimento do agente: descoberta inicial, loop por task com do-execute-task, verificação de conclusão, limpeza de contexto entre tasks e resumo final."
)
```

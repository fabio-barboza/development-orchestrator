---
mode: agent
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em cada uma, com isolamento de contexto via subagente.
tools:
  - agent
---

# /execute-all-tasks

Delegue **integralmente** a execução para o agente `execute-all-tasks` (definido em `.github/agents/execute-all-tasks.agent.md`) usando a tool `agent`.

## Entrada do usuário

A solicitação completa está em `${input:request}` (ou no texto que acompanhou o comando). Espera-se:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `prds/prd-<nome>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se faltar o caminho do `tasks.md`, **pergunte uma única vez** antes de prosseguir.

## Procedimento

1. Faça o parse dos argumentos extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Use a tool `agent` para spawnar o agente `execute-all-tasks` com contexto isolado:
   ```
   agent(
     agent: "execute-all-tasks",
     prompt: "Execute as tasks de <caminho/tasks.md> com filtro <filtro>, seguindo integralmente o procedimento do agente: descoberta inicial, loop por task com do-execute-task em contexto isolado, verificação de conclusão e resumo final."
   )
   ```
3. **Não** execute o procedimento você mesmo — apenas delegue. O subagente roda em sessão isolada.
4. Após o retorno do subagente, repasse o resumo final ao usuário sem alterações.

## Exemplos de uso

```
/execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0-4.0
/execute-all-tasks prds/prd-login-google/tasks/tasks.md all
/execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0,3.0,5.0
```

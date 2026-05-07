---
mode: agent
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em cada uma, com limpeza de contexto entre elas.
---

# /do-execute-all-tasks

Delegue **integralmente** a execução ao subagente `Do Execute All Tasks` (definido em `.github/agents/do-execute-all-tasks.agent.md`).

## Entrada do usuário

A solicitação completa está em `${input:request}` (ou no texto que acompanhou o comando). Espera-se:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se o caminho não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Procedimento

1. Faça o parse da entrada extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Invoque o subagente `Do Execute All Tasks` passando:
   - Caminho exato do `tasks.md`
   - Filtro de tasks
   - Reforço de que deve seguir o procedimento do agente na íntegra (descoberta → loop → limpeza de contexto entre tasks → encerramento)
3. **Não** execute o procedimento você mesmo — apenas delegue. O subagente é quem orquestra com contexto isolado.
4. Após o retorno do subagente, repasse o resumo final ao usuário sem alterações.

## Exemplo de uso

```
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0-4.0
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md all
```

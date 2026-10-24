---
description: Executa a fila de tasks de um PRD em sequência, uma task por subagente isolado (agent-execute-task), com a orquestração no agente primário.
---

# /do-execute-all-tasks

Carregue a skill `do-execute-all-tasks` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumentos

$ARGUMENTS

Espera-se o caminho do `tasks.md` (obrigatório) e, opcionalmente, um filtro: `all`, uma lista de IDs (`1.0,2.0,5.0`) ou um range (`1.0-4.0`). Se o caminho não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-all-tasks prds/prd-user-auth/tasks/tasks.md 1.0-4.0
```

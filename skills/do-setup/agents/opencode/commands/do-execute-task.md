---
description: Implementa uma task específica lendo o contexto do PRD e TechSpec, executando a implementação com testes e realizando uma revisão automática de código. Marca a task como concluída em tasks.md.
---

# /do-execute-task

Use a skill `do-execute-task` e execute o procedimento completo descrito nele na íntegra.

## Argumento esperado

O usuário deve fornecer o caminho do arquivo da task a implementar.

```
/do-execute-task prds/prd-<feature-slug>/tasks/task-<id>.md
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-task prds/prd-user-auth/tasks/task-1.0.md
```

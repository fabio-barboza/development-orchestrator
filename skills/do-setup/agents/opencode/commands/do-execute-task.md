---
description: Implementa uma task específica lendo o contexto do PRD e TechSpec, executando a implementação com testes e realizando uma revisão automática de código. Marca a task como concluída em tasks.md.
---

# /do-execute-task

Carregue a skill `do-execute-task` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário deve fornecer o caminho do arquivo da task a implementar.

```
/do-execute-task prds/prd-<feature-slug>/tasks/<num>_task.md
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-task prds/prd-user-auth/tasks/1_task.md
```

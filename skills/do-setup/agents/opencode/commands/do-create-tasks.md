---
description: Converte PRD e Tech Spec em uma lista detalhada e sequenciada de tasks de implementação. Cada task é um entregável funcional e incremental com sua própria suite de testes. Outputa tasks.md e arquivos individuais de task.
---

# /do-create-tasks

Use a skill `do-create-tasks` e execute o procedimento completo descrito nele na íntegra.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD disponível em `./prds/`.

## Exemplo

```
/do-create-tasks user-auth
/do-create-tasks
```

---
description: Converte PRD e Tech Spec em uma lista detalhada e sequenciada de tasks de implementação. Cada task é um entregável funcional e incremental com sua própria suite de testes. Outputa tasks.md e arquivos individuais de task.
---

# /do-create-tasks

Carregue a skill `do-create-tasks` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD disponível em `./prds/`.

## Exemplo

```
/do-create-tasks user-auth
/do-create-tasks
```

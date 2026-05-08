---
description: Mostra o progresso atual de um PRD — tasks concluídas, próxima task pendente, percentual de conclusão e artefatos faltantes. Skill read-only, não modifica arquivos.
---

# /do-status

Use a skill `do-status` e execute o procedimento completo descrito nele na íntegra.

## Argumento opcional

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD disponível em `./prds/`.

## Exemplo

```
/do-status user-auth
/do-status
```

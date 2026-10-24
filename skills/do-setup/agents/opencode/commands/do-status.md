---
description: Mostra o progresso atual de um PRD — tasks concluídas, próxima task pendente, percentual de conclusão e artefatos faltantes. Skill read-only, não modifica arquivos.
---

# /do-status

Carregue a skill `do-status` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento opcional

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD disponível em `./prds/`.

## Exemplo

```
/do-status user-auth
/do-status
```

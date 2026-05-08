---
description: Valida a implementação de uma feature contra PRD, Tech Spec e Tasks via testes E2E com MCPs disponíveis, verificação de acessibilidade (WCAG 2.2) e análise visual. Documenta todos os bugs encontrados com evidências e gera um relatório de QA abrangente.
---

# /do-execute-qa

Use a skill `do-execute-qa` e execute o procedimento completo descrito nele na íntegra.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD mais recente em `./prds/`.

## Exemplo

```
/do-execute-qa user-auth
/do-execute-qa
```

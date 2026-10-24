---
description: Valida a implementação de uma feature contra PRD, Tech Spec e Tasks via testes E2E com MCPs disponíveis, verificação de acessibilidade e análise visual. Documenta todos os bugs encontrados com evidências e gera um relatório de QA abrangente.
---

# /do-execute-qa

Carregue a skill `do-execute-qa` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD mais recente em `./prds/`.

## Exemplo

```
/do-execute-qa user-auth
/do-execute-qa
```

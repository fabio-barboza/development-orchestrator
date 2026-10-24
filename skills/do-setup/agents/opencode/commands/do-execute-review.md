---
description: Realiza revisão de código abrangente analisando git diff, verificando conformidade com as regras do projeto, validando suites de testes e aderência ao TechSpec e Tasks. Gera um relatório de code review estruturado com findings classificados por severidade.
---

# /do-execute-review

Carregue a skill `do-execute-review` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD mais recente em `./prds/`.

## Exemplo

```
/do-execute-review user-auth
/do-execute-review
```

---
description: Cria uma Especificação Técnica a partir de um PRD existente, traduzindo requisitos de produto em decisões arquiteturais e guias de implementação.
---

# /do-create-techspec

Carregue a skill `do-create-techspec` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD disponível em `./prds/`.

## Exemplo

```
/do-create-techspec user-auth
/do-create-techspec
```

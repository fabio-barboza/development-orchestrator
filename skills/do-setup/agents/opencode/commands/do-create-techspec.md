---
description: Cria uma Especificação Técnica a partir de um PRD existente, traduzindo requisitos de produto em decisões arquiteturais e guias de implementação.
---

# /do-create-techspec

Use a skill `do-create-techspec` e execute o procedimento completo descrito nele na íntegra.

## Argumento esperado

O usuário pode fornecer o slug da feature (ex.: `user-auth`). Se omitido, o procedimento detecta automaticamente o PRD disponível em `./prds/`.

## Exemplo

```
/do-create-techspec user-auth
/do-create-techspec
```

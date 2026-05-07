---
description: Recebe o caminho de um arquivo de bug em qa-bugs/, analisa a causa raiz, implementa a correção com testes de regressão, valida a suite de testes e atualiza o status do arquivo. Invoque uma vez por bug.
---

# /do-execute-qa-bugfix

Use a skill `do-execute-qa-bugfix` e execute o procedimento completo descrito nele na íntegra.

## Argumento esperado

O usuário deve fornecer o caminho do arquivo de bug gerado pelo `do-execute-qa`.

```
/do-execute-qa-bugfix prds/prd-<feature-slug>/qa-bugs/bug-<id>-<severidade>-<slug>.md
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-qa-bugfix prds/prd-user-auth/qa-bugs/bug-01-alta-formulario.md
```

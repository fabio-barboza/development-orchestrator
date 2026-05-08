---
description: Recebe o caminho de um arquivo de fix task em review-fixes/, implementa a correção, roda a suite de testes e atualiza o status do arquivo. Use após do-execute-review gerar arquivos de fix task.
---

# /do-execute-review-fix

Use a skill `do-execute-review-fix` e execute o procedimento completo descrito nele na íntegra.

## Argumento esperado

O usuário deve fornecer o caminho do arquivo de fix task gerado pelo `do-execute-review`.

```
/do-execute-review-fix prds/prd-<feature-slug>/review-fixes/fix-<id>-<severidade>-<slug>.md
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-review-fix prds/prd-user-auth/review-fixes/fix-R01-critico-validacao.md
```

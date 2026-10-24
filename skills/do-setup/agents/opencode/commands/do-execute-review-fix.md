---
description: Recebe o caminho de um arquivo de fix task em review-<N>/fixes/, implementa a correção, roda a suite de testes e atualiza o status do arquivo. Use após do-execute-review gerar arquivos de fix task.
---

# /do-execute-review-fix

Carregue a skill `do-execute-review-fix` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário deve fornecer o caminho do arquivo de fix task gerado pelo `do-execute-review`.

```
/do-execute-review-fix prds/prd-<feature-slug>/review-<N>/fixes/fix-<id>-<severidade>-<slug>.md
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-review-fix prds/prd-user-auth/review-1/fixes/fix-R01-critico-validacao.md
```

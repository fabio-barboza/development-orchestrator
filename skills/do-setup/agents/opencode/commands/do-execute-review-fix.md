---
description: Recebe o caminho de um arquivo de fix task em review/review-<N>/fixes/ (ou uma pasta de fixes, ou vários arquivos), implementa a correção, roda a suite de testes e atualiza o status. Com vários arquivos, cada fix roda em um subagente isolado. Use após do-execute-review gerar arquivos de fix task.
---

# /do-execute-review-fix

Carregue a skill `do-execute-review-fix` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário deve fornecer o caminho de um arquivo de fix task gerado pelo `do-execute-review`, **ou** de uma pasta `fixes/`, **ou** de vários arquivos separados por espaço. Com uma pasta, a opção `--min=<critico|maior|menor>` limita a fila.

```
/do-execute-review-fix prds/prd-<feature-slug>/review/review-<N>/fixes/fix-<id>-<severidade>-<slug>.md
/do-execute-review-fix prds/prd-<feature-slug>/review/review-<N>/fixes/ --min=maior
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-review-fix prds/prd-user-auth/review/review-1/fixes/fix-R01-critico-validacao.md
/do-execute-review-fix prds/prd-user-auth/review/review-1/fixes/
```

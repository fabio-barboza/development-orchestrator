---
description: Recebe o caminho de um arquivo de bug em qa/qa-<N>/bugs/ (ou uma pasta de bugs, ou vários arquivos), analisa a causa raiz, implementa a correção com testes de regressão, valida a suite de testes e atualiza o status. Com vários arquivos, cada bug roda em um subagente isolado.
---

# /do-execute-qa-bugfix

Carregue a skill `do-execute-qa-bugfix` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário deve fornecer o caminho de um arquivo de bug gerado pelo `do-execute-qa`, **ou** de uma pasta `bugs/`, **ou** de vários arquivos separados por espaço. Com uma pasta, a opção `--min=<alta|media|baixa>` limita a fila.

```
/do-execute-qa-bugfix prds/prd-<feature-slug>/qa/qa-<N>/bugs/bug-<id>-<severidade>-<slug>.md
/do-execute-qa-bugfix prds/prd-<feature-slug>/qa/qa-<N>/bugs/ --min=media
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-qa-bugfix prds/prd-user-auth/qa/qa-1/bugs/bug-01-alta-formulario.md
/do-execute-qa-bugfix prds/prd-user-auth/qa/qa-1/bugs/
```

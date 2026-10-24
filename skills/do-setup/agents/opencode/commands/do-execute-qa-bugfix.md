---
description: Recebe o caminho de um arquivo de bug em qa-<N>/bugs/, analisa a causa raiz, implementa a correção com testes de regressão, valida a suite de testes e atualiza o status do arquivo. Invoque uma vez por bug.
---

# /do-execute-qa-bugfix

Carregue a skill `do-execute-qa-bugfix` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento esperado

O usuário deve fornecer o caminho do arquivo de bug gerado pelo `do-execute-qa`.

```
/do-execute-qa-bugfix prds/prd-<feature-slug>/qa-<N>/bugs/bug-<id>-<severidade>-<slug>.md
```

Se não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Exemplo

```
/do-execute-qa-bugfix prds/prd-user-auth/qa/qa-1/bugs/bug-01-alta-formulario.md
```

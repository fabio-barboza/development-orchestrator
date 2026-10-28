---
description: Inicializa o projeto para o DO Framework, identifica skills relevantes e grava o contexto local do framework em `.do/context.md` (resumo, convenções, comandos, skills e MCPs), sem alterar um CLAUDE.md/AGENTS.md existente (se o projeto não tiver nenhum, pergunta se deve criar um só com o conteúdo do projeto). Também inventaria e informa as skills e MCPs instalados. Aceita os argumentos "agents" (instala apenas os agentes de orquestração) e "refresh" (redetecta skills e MCPs e atualiza essas seções).
---

# /do-setup

Carregue a skill `do-setup` com a tool `skill` e execute o procedimento completo descrito nela na íntegra. **Recarregue a skill mesmo que já a tenha carregado antes nesta sessão** — o texto em contexto pode ser de uma versão anterior.

## Argumento opcional

- Sem argumento: executa o setup completo (análise do projeto + instalação de agentes).
- `agents`: instala apenas os agentes e comandos de orquestração, sem rodar o setup completo.
- `refresh`: redetecta as skills e os MCPs instalados e atualiza só as seções `Available Skills`, `Uncovered Skills` e `Available MCPs` de `.do/context.md` (use depois de instalar novas skills ou MCPs).

## Exemplo

```
/do-setup
/do-setup agents
/do-setup refresh
```

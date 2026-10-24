---
description: Inicializa o projeto, identifica skills relevantes e atualiza o arquivo de configuração do projeto com resumo, convenções e skills disponíveis. Também inventaria e informa as skills e MCPs instalados. Aceita os argumentos "agents" (instala apenas os agentes de orquestração) e "refresh" (redetecta skills e MCPs e atualiza essas seções).
---

# /do-setup

Use a skill `do-setup` e execute o procedimento completo descrito nele na íntegra.

## Argumento opcional

- Sem argumento: executa o setup completo (análise do projeto + instalação de agentes).
- `agents`: instala apenas os agentes e comandos de orquestração, sem rodar o setup completo.
- `refresh`: redetecta as skills e os MCPs instalados e atualiza só as seções `Available Skills`, `Uncovered Skills` e `Available MCPs` do arquivo de configuração (use depois de instalar novas skills ou MCPs).

## Exemplo

```
/do-setup
/do-setup agents
/do-setup refresh
```

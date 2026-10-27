# Catálogo de Capacidades de MCP

Define as **capacidades** que um MCP server pode oferecer e como tratá-las. Este catálogo não lista servidores nem produtos: um MCP é classificado pelo que as suas tools **fazem**, nunca pelo nome do server. As skills de execução consultam este arquivo para decidir quais tools usar conforme o contexto da feature.

## Categorias de Capacidade

| Categoria | Descrição | Exemplos de uso |
|-----------|-----------|-----------------|
| ui-testing | Operar uma interface de usuário em execução: navegar, interagir, ler o estado da tela e capturar evidência visual | Validação ponta a ponta de fluxos de interface, validação visual, acessibilidade |
| api-testing | Enviar requisições a uma API em execução e validar as respostas | Validação ponta a ponta de serviços sem interface |
| messaging | Publicar/consumir mensagens e inspecionar filas, tópicos ou canais | Validação de fluxos orientados a eventos |
| data-store | Consultar e validar o estado de um armazenamento de dados (banco, cache, índice) | Integridade de dados, validação de migrações, comportamento de cache |
| documentation | Consultar documentação técnica de bibliotecas, frameworks ou serviços | Pesquisa técnica, verificação de uso correto de APIs |

## Como classificar um MCP server

Para cada server descoberto (ver `do-mcp-discovery-instructions.md`), olhe os **nomes e descrições das tools** que ele expõe e atribua uma ou mais categorias:

| Categoria | Sinais nas tools |
|-----------|------------------|
| ui-testing | tools que navegam, clicam, digitam, leem o estado/árvore da tela, capturam screenshot ou leem console/rede da interface |
| api-testing | tools que montam e enviam requisições e inspecionam status, cabeçalhos e corpo das respostas |
| messaging | tools que publicam, consomem, listam ou inspecionam mensagens, filas, tópicos ou assinaturas |
| data-store | tools que consultam, listam ou alteram registros, esquemas ou chaves de um armazenamento |
| documentation | tools que resolvem uma biblioteca/serviço e devolvem documentação ou exemplos |

Regras:
- Classifique pelo comportamento. Um nome de server, de produto ou de biblioteca nunca decide a categoria.
- Se as tools expostas forem genéricas (por exemplo, "buscar operação", "obter documentação da operação", "executar operação"), use-as nessa ordem para descobrir as operações disponíveis e a categoria vem do que as operações fazem.
- Se nenhuma categoria se aplicar, o server não é usado pelo framework; registre-o como "capacidade não reconhecida" e ignore-o.
- **Prefixo de tools por ferramenta de IA:** o nome de uma tool aparece como `mcp__<server>__<tool>` (Claude Code, Cursor, GitHub Copilot) ou `<server>_<tool>` (Opencode). Use sempre o nome exato que aparece na lista de tools da sessão.

## Tratamento por categoria

| Categoria | Requer alvo em execução | Se indisponível |
|-----------|-------------------------|-----------------|
| ui-testing | Sim — a aplicação precisa estar rodando (ver `do-service-readiness.md`) | Reportar o erro, **não** usar linha de comando nem scripts que acionam diretamente uma biblioteca de automação de interface; documentar a lacuna no relatório. Um erro de conexão ao navegar até a aplicação significa que a aplicação caiu, não que o MCP está indisponível: restabeleça o serviço e repita a chamada |
| api-testing | Sim — a API precisa estar rodando | Reportar o erro, documentar a lacuna e continuar com testes unitários/integração |
| messaging | Sim — o broker/serviço de mensagens precisa estar acessível | Reportar o erro, documentar a lacuna e continuar com testes unitários/integração |
| data-store | Sim — o armazenamento precisa estar acessível | Reportar o erro, documentar a lacuna e continuar com testes unitários/integração |
| documentation | Não | Prosseguir sem ele (busca na web ou leitura do código) |

Serviços externos (mensageria, armazenamento de dados, APIs de terceiros) nunca são iniciados automaticamente; apenas servidores de desenvolvimento do próprio projeto, conforme `do-service-readiness.md`.

## Como adicionar um novo MCP server

1. Adicione o server ao arquivo de configuração MCP da sua ferramenta de IA:
   - **Claude Code:** `.mcp.json` na raiz do projeto (ou `~/.mcp.json` para configuração global)
   - **GitHub Copilot:** `.vscode/mcp.json`
   - **Cursor:** `.cursor/mcp.json`
   - **Opencode:** `opencode.json` na raiz do projeto (chave `mcp`) ou o global `~/.config/opencode/opencode.json`
2. **Nenhuma skill nem este catálogo precisa ser editado.** Na próxima execução o server é descoberto e classificado pelas tools que expõe. Rode `/do-setup refresh` para atualizar o resumo gravado em `.do/context.md`.

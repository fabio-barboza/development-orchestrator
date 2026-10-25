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

## Chamadas que terminam sempre

Enquanto uma tool de MCP está rodando, você fica bloqueado: nenhuma regra escrita aqui será lida e ninguém interrompe a chamada por você. Uma espera que nunca termina trava a sessão inteira. Por isso, **quem escreve o código que a tool vai executar** (um script rodado dentro da página, uma expressão avaliada na interface, uma requisição montada à mão) é responsável por garanti-lo finito:

1. **Toda espera tem limite.** Nunca aguarde um evento, uma resposta, um elemento ou uma promessa sem prazo. Envolva a espera numa corrida contra um temporizador curto (por exemplo, de 5 a 10 segundos) que rejeita ou devolve "expirou", e trate esse resultado.
2. **Prefira a tool dedicada ao código livre.** Para esperar um texto, um elemento ou um tempo, use a tool de espera do MCP, que já tem prazo. Reserve o código livre para ler ou preparar o estado, e mantenha-o curto.
3. **Sem laços sem saída.** Nenhum laço sem contador máximo; nenhum "aguarde até que" sem prazo.
4. **Um passo por chamada.** Divida um roteiro longo em chamadas pequenas, cada uma devolvendo um resultado. Uma chamada que precisa de mais de cerca de 30 segundos está errada: divida-a.
5. **Estado que o código altera é desfeito.** Se o código substituiu algo na página (um método do navegador, uma função global), restaure-o ou recarregue a página antes de seguir.
6. **Chamada interrompida ou expirada: não repita o mesmo código.** Se a chamada foi interrompida pelo usuário, expirou ou nunca retornou, a causa está no código. Releia o que escreveu, encontre a espera sem limite, corrija e só então tente de novo; no máximo 2 tentativas do mesmo passo. Se persistir, registre a lacuna (`NÃO TESTADO`, com o motivo) e siga para o próximo item.

Serviços externos (mensageria, armazenamento de dados, APIs de terceiros) nunca são iniciados automaticamente; apenas servidores de desenvolvimento do próprio projeto, conforme `do-service-readiness.md`.

## Como adicionar um novo MCP server

1. Adicione o server ao arquivo de configuração MCP da sua ferramenta de IA:
   - **Claude Code:** `.mcp.json` na raiz do projeto (ou `~/.mcp.json` para configuração global)
   - **GitHub Copilot:** `.vscode/mcp.json`
   - **Cursor:** `.cursor/mcp.json`
   - **Opencode:** `opencode.json` na raiz do projeto (chave `mcp`) ou o global `~/.config/opencode/opencode.json`
2. **Nenhuma skill nem este catálogo precisa ser editado.** Na próxima execução o server é descoberto e classificado pelas tools que expõe. Rode `/do-setup refresh` para atualizar o resumo gravado no arquivo de configuração do projeto.

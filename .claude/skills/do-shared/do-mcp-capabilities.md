# MCP Capability Registry

Fonte de verdade sobre as capacidades de cada MCP server configurado no projeto. As skills de execução consultam este arquivo para descobrir quais tools usar conforme o contexto da feature.

## Categorias de Capacidade

| Categoria | Descricao | Exemplos de uso |
|-----------|-----------|-----------------|
| browser-testing | Navegar, interagir e capturar screenshots de UIs web | E2E frontend, validacao visual, acessibilidade |
| message-queue | Publicar/consumir mensagens, inspecionar filas e exchanges | Validacao de fluxos event-driven, RabbitMQ E2E |
| database | Consultar e validar estado do banco de dados | Integridade de dados, validacao de migrations |
| cache | Ler/escrever/inspecionar entradas de cache | Validacao de comportamento Redis/Memcached |
| api-testing | Enviar requests HTTP, validar responses | E2E de APIs REST/GraphQL sem browser |
| documentation | Consultar documentacao de frameworks/libs | Pesquisa tecnica, verificacao de API usage |

## Servidores Conhecidos

### playwright
- **Capacidades:** browser-testing
- **Prefixo de tools:** `mcp__playwright__browser_*`
- **Quando usar:** E2E de features frontend, validacao visual, verificacao de acessibilidade, captura de screenshots como evidencia
- **Requer app rodando:** Sim (iniciar dev server antes de usar)
- **Se indisponivel:** Reportar erro, NAO usar CLI (`npx playwright test`), documentar gap no relatorio
- **Tools principais:**
  | Tool | Uso |
  |------|-----|
  | `browser_navigate` | Navegar para paginas da aplicacao |
  | `browser_snapshot` | Capturar estado acessivel da pagina (preferido sobre screenshot para analise) |
  | `browser_click` | Interagir com botoes, links e elementos clicaveis |
  | `browser_type` | Preencher campos de formulario |
  | `browser_fill_form` | Preencher multiplos campos de uma vez |
  | `browser_select_option` | Selecionar opcoes em dropdowns |
  | `browser_press_key` | Simular teclas (Enter, Tab, Escape, etc.) |
  | `browser_take_screenshot` | Capturar evidencia visual |
  | `browser_console_messages` | Verificar erros no console do browser |
  | `browser_network_requests` | Verificar chamadas de API |

### context7
- **Capacidades:** documentation
- **Prefixo de tools:** `mcp__context7__`
- **Quando usar:** Verificar documentacao de frameworks/libs para uso correto de APIs, best practices e padroes recomendados
- **Requer app rodando:** Nao
- **Se indisponivel:** Prosseguir sem ele
- **Tools principais:**
  | Tool | Uso |
  |------|-----|
  | `resolve-library-id` | Resolver identificador de uma biblioteca |
  | `query-docs` | Consultar documentacao da biblioteca |

### rabbitmq
- **Capacidades:** message-queue
- **Prefixo de tools:** `mcp__rabbitmq__`
- **Quando usar:** Validar fluxos de mensageria (publish/consume), inspecionar filas e exchanges, verificar health do broker, testar E2E de features event-driven
- **Requer app rodando:** Sim (RabbitMQ broker deve estar acessivel em localhost)
- **Se indisponivel:** Reportar erro, documentar gap no relatorio, continuar com testes unitarios/integracao
- **Padrao de uso:** Este MCP usa descoberta semantica — use `search-ids` para encontrar operacoes, `get-id` para obter documentacao da operacao, e `call-id` para executar
- **Tools principais:**
  | Tool | Uso |
  |------|-----|
  | `search-ids` | Busca semantica de operacoes RabbitMQ (ex: "list queues", "publish message") |
  | `get-id` | Obter documentacao completa de uma operacao pelo ID |
  | `call-id` | Executar uma operacao RabbitMQ pelo ID e parametros |
  | `connection.health` | Verificar saude da conexao com o broker (AMQP + HTTP) |
- **Operacoes comuns (via search-ids + call-id):**
  - Listar filas e inspecionar mensagens
  - Publicar mensagens em exchanges
  - Listar exchanges e bindings
  - Verificar consumers conectados
  - Purgar filas (requer `--allow-mutative-tools`)

## Como Adicionar Novo MCP Server

1. Adicione o server ao `.mcp.json` na raiz do projeto
2. Adicione uma entrada neste arquivo sob "Servidores Conhecidos" com:
   - Capacidades (usando as categorias da tabela acima)
   - Prefixo de tools
   - Quando usar
   - Se requer app rodando
   - Handling de indisponibilidade
   - Lista de tools principais
3. **Nenhuma skill precisa ser editada** — as skills descobrem capacidades dinamicamente a partir deste registry

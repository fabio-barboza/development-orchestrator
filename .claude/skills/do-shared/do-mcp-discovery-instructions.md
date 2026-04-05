# Descoberta e Roteamento de MCPs

Procedimento padrao referenciado pelas skills de execucao (do-execute-task, do-execute-qa, do-execute-bugfix, do-execute-review). Substitui a logica hardcoded de Playwright por descoberta dinamica baseada em capacidades.

## Procedimento de Descoberta

1. Ler `.mcp.json` na raiz do projeto para listar MCP servers configurados.
2. Ler `.claude/skills/do-shared/do-mcp-capabilities.md` para mapear cada server as suas capacidades e tools.
3. Construir mapa interno de capacidades. Exemplo:
   ```
   { "browser-testing": ["playwright"], "message-queue": ["rabbitmq"], "documentation": ["context7"] }
   ```
4. Para cada necessidade de teste/validacao no procedimento atual:
   a. Identificar a capacidade necessaria (ex: `browser-testing` para frontend E2E, `message-queue` para validacao de mensageria).
   b. Buscar no mapa qual MCP server provem essa capacidade.
   c. **Se existe**: usar os tools conforme descrito na entrada do registry. Respeitar as regras de "Requer app rodando" e "Se indisponivel".
   d. **Se nao existe**: documentar o gap no relatorio ("MCP com capacidade [X] nao configurado — validacao [Y] nao executada") e continuar com testes unitarios/integracao.
5. Para MCPs que requerem app rodando: verificar se o servico esta acessivel antes de invocar tools. Se nao estiver, tentar iniciar (dev server para browser-testing, verificar broker para message-queue, etc.).
6. Se MCP configurado mas indisponivel em runtime (erro de conexao, tools nao respondem): seguir o handling de indisponibilidade descrito na entrada do registry.
7. Se um MCP em `.mcp.json` NAO esta listado no registry: tentar usar tools com prefixo `mcp__<server-name>__` e documentar no relatorio que o MCP nao esta registrado em `mcp-capabilities.md`.

## Guard de Capacidade (substitui o frontend guard binario)

O guard antigo era binario: "se backend, pular E2E". O novo guard e baseado em capacidades disponiveis.

**Como aplicar:**

1. Analisar a feature (PBI, TechSpec, Tasks) para determinar se envolve **frontend/UI**, **backend**, ou **ambos**.
2. Consultar o mapa de capacidades construido no passo 3 acima.
3. Aplicar a tabela de decisao:

| Tipo da Feature | MCP com capacidade relevante | Acao |
|-----------------|------------------------------|------|
| Frontend + `browser-testing` disponivel | Sim | Rodar E2E via browser MCP |
| Backend + `message-queue`/`database`/`cache`/`api-testing` disponivel | Sim | Rodar E2E backend via MCP correspondente |
| Frontend + Backend + ambos MCPs disponiveis | Sim | Rodar ambos E2E (browser + backend) |
| Backend + apenas `browser-testing` disponivel | Nao | Pular E2E (browser nao valida backend). Documentar gap. |
| Frontend + apenas MCPs backend disponiveis | Nao | Pular E2E browser. Documentar gap. |
| Qualquer tipo + nenhum MCP relevante | Nao | Pular E2E. Documentar gap. Prosseguir com unit/integration. |

**Regra critica**: O guard so pula E2E quando NENHUM MCP disponivel consegue validar o tipo de feature. A presenca de um MCP backend habilita E2E para features backend que antes eram puladas.

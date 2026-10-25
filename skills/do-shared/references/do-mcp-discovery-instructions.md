# Descoberta e Roteamento de MCPs

Procedimento padrão referenciado pelas skills de execução (`do-execute-task`, `do-execute-qa`, `do-execute-qa-bugfix`, `do-execute-review`) e pelo `do-setup`. A descoberta é dinâmica e baseada em **capacidades**: nenhum servidor, produto ou ferramenta específica é presumido.

## Procedimento de Descoberta

1. Identificar e ler o arquivo de configuração de MCP servers do projeto. O arquivo varia conforme a ferramenta de IA utilizada:
   - **Claude Code:** `.mcp.json` na raiz do projeto
   - **GitHub Copilot:** `.vscode/mcp.json`
   - **Cursor:** `.cursor/mcp.json`
   - **Opencode:** `opencode.json` na raiz do projeto (chave `mcp`); se ausente, o global `~/.config/opencode/opencode.json`

   Listar os MCP servers configurados nesse arquivo, **somente pelos nomes** (ver o aviso de segurança abaixo).

   **Segurança — nunca leia o arquivo inteiro para o contexto.** Esses arquivos (principalmente o global do Opencode e qualquer configuração de ferramenta) costumam conter credenciais de provedores, tokens e cabeçalhos de autorização; ler já os expõe ao modelo e ao histórico da sessão. **Não use `cat`, nem a tool de leitura de arquivos, no arquivo de configuração inteiro.** Extraia somente os **nomes dos servidores** e o **tipo** (local/remoto) do bloco de MCP (`mcpServers`, `servers` ou `mcp`, conforme a ferramenta), com um comando de extração que imprima apenas essas chaves (o arquivo pode ser JSON com comentários: remova-os ao analisar). Nunca imprima `env`, `headers`, `url` com credenciais, `apiKey`, `token` ou qualquer valor: se usar um comando que imprime o bloco de MCP (por exemplo com contexto de linhas), **filtre antes de exibir** qualquer linha que contenha `env`, `header`, `authorization`, `token`, `key`, `secret`, `password` ou uma URL, e mostre apenas os nomes dos servidores e o `type`. **Cuidado ao tirar comentários:** uma expressão simples como `//.*$` apaga o final de qualquer URL (`https://…`) e quebra o JSON; remova comentários só fora de strings ou use um parser que aceite JSON com comentários. **Remoção de comentários que preserva strings** (adapte à linguagem do seu comando de extração; o ponto é só o algoritmo): percorra o texto caractere a caractere guardando se está dentro de uma string entre aspas (tratando `\"` como escape); fora de string, `//` descarta até o fim da linha e `/* ... */` descarta até o fechamento; dentro de string, copie tudo. Depois do parse, imprima apenas as chaves do bloco de MCP (`mcp`, `mcpServers` ou `servers`); se o bloco tiver uma chave intermediária (por exemplo `servers` dentro de `mcp`), desça um nível até chegar aos nomes dos servidores, e não imprima o nome da chave do bloco como se fosse um servidor.

**Erros de análise também vazam conteúdo** (a mensagem de erro de muitos parsers inclui um trecho do arquivo): capture o erro e imprima somente uma mensagem fixa (por exemplo, `não foi possível ler a configuração de MCP`), nunca o texto do erro. Se não conseguir isolar o bloco de MCP sem expor o restante, **não leia o arquivo** e use somente a lista de tools da sessão.

   **Complemento pela lista de tools:** além do arquivo, verifique as tools disponíveis na sua sessão. Tools expostas por um server que não consta em nenhum arquivo de configuração do projeto (MCP configurado globalmente) indicam que ele existe; trate-o como configurado. **As ferramentas nativas da própria ferramenta de IA (arquivos, shell, busca, sessões, worktrees, delegação a subagentes etc.) NÃO são MCP servers** e nunca entram na lista: só conta o que vem de um server MCP configurado.
2. Ler o catálogo de capacidades em `do-shared/references/do-mcp-capabilities.md` e **classificar cada server pelo que as suas tools fazem** (nomes e descrições das tools), nunca pelo nome do server.
3. Construir um mapa interno de capacidades. Exemplo (os nomes são apenas ilustrativos):
   ```
   { "ui-testing": ["<server-a>"], "messaging": ["<server-b>"], "documentation": ["<server-c>"] }
   ```
4. Para cada necessidade de teste/validação no procedimento atual:
   a. Identificar a capacidade necessária (ex.: `ui-testing` para validar fluxos de interface, `messaging` para validar fluxos de mensageria).
   b. Buscar no mapa qual MCP server provê essa capacidade.
   c. **Se existe**: usar as tools dele. Respeitar as regras de "Requer alvo em execução" e "Se indisponível" da categoria no catálogo.
   d. **Se não existe**: documentar a lacuna no relatório ("MCP com capacidade [X] não configurado — validação [Y] não executada") e continuar com testes unitários/integração.
5. Para capacidades que requerem alvo em execução: verificar se o serviço está acessível antes de invocar as tools, pelo procedimento de `do-shared/references/do-service-readiness.md`.
6. Se um MCP está configurado mas indisponível em runtime (erro de conexão, tools não respondem): seguir o tratamento de indisponibilidade da categoria no catálogo. **Atenção:** erro de conexão do *serviço alvo* (ex.: conexão recusada ao acessar a aplicação) NÃO é indisponibilidade do MCP — o serviço caiu. Restabeleça-o pelo procedimento de `do-shared/references/do-service-readiness.md` e repita a chamada. Nunca troque o método de validação: não substitua o MCP por scripts que acionam diretamente uma biblioteca de automação nem por linha de comando.
7. Se as tools de um server não se encaixam em nenhuma categoria do catálogo, ignore-o para fins de validação e registre-o no relatório como "capacidade não reconhecida".

## Guard de Capacidade

O guard é baseado nas capacidades disponíveis, não no tipo de tecnologia do projeto.

**Como aplicar:**

1. Analisar a feature (PRD, TechSpec, Tasks) para determinar o que ela expõe: **interface de usuário**, **serviço** (API, mensageria, armazenamento de dados) ou outro tipo de artefato (biblioteca, ferramenta de linha de comando, etc.).
2. Consultar o mapa de capacidades construído no passo 3 acima.
3. Aplicar a tabela de decisão:

| Tipo da feature | MCP com capacidade relevante | Ação |
|-----------------|------------------------------|------|
| Interface de usuário + `ui-testing` disponível | Sim | Rodar validação ponta a ponta via MCP de interface |
| Serviço + `api-testing`/`messaging`/`data-store` disponível | Sim | Rodar validação ponta a ponta do serviço via MCP correspondente |
| Interface + serviço, com os dois tipos de MCP disponíveis | Sim | Rodar as duas validações |
| Serviço e apenas `ui-testing` disponível | Não | Pular a validação ponta a ponta. Documentar a lacuna. |
| Interface e apenas MCPs de serviço disponíveis | Não | Pular a validação de interface. Documentar a lacuna. |
| Qualquer tipo + nenhum MCP relevante | Não | Pular a validação ponta a ponta. Documentar a lacuna. Prosseguir com unitários/integração. |

**Regra crítica**: o guard só pula a validação ponta a ponta quando NENHUM MCP disponível consegue validar o tipo de feature.

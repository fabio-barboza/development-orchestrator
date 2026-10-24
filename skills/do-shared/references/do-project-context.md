# Skills e MCPs do Projeto

Procedimento padrão referenciado pelas skills `do-*` que implementam, revisam, testam ou especificam código. Define como usar as skills e os MCPs que o projeto tem instalados **sem assumir que nenhum deles exista**. Apenas as skills `do-*` são garantidas; qualquer outra skill ou MCP pode estar ou não instalada.

## Onde a lista fica registrada

O `do-setup` grava duas seções no arquivo de configuração do projeto da ferramenta de IA em uso:

| Ferramenta | Arquivo de configuração |
|------------|-------------------------|
| Claude Code | `CLAUDE.md` (ou `AGENTS.md`, quando não existe `CLAUDE.md`) |
| Opencode | `AGENTS.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor AI | `.cursor/rules/project.mdc` (ou `.cursorrules`) |

- `## Available Skills`: tabela `Skill | Caminho | Quando usar`.
- `## Available MCPs`: tabela `MCP | Capacidade | Quando usar`.

Essas seções são um **resumo gravado no momento do setup**. Podem estar ausentes, vazias ou desatualizadas (uma skill instalada depois do setup não aparece nelas). Por isso o procedimento abaixo sempre confirma contra o que a sessão realmente enxerga.

## Procedimento — Skills

1. Leia o arquivo de configuração do projeto (tabela acima) e extraia a seção `Available Skills`. Se o arquivo ou a seção não existir, siga para o passo 2.
2. Complete a lista com as skills que a sua ferramenta expõe nativamente (por exemplo a tool `skill` ou a lista de skills disponíveis na sessão). Ignore todas as `do-*`.
3. Escolha as skills **relevantes para a tarefa atual** pelo critério "Quando usar" (ou pela `description` do `SKILL.md`): linguagem/framework envolvido, tipo de trabalho (UI, testes, revisão, banco de dados etc.).
4. Para cada skill escolhida, leia o `SKILL.md` (pelo caminho da tabela ou pela tool `skill`) e incorpore suas regras ao trabalho.
5. Se o caminho da tabela não existir mais, descarte a entrada e registre "skill indisponível" — não é erro e não interrompe a execução.
6. Se não houver nenhuma skill relevante ou nenhuma instalada, siga sem elas. **Nunca** cite, exija ou invente uma skill pelo nome: use só as que existem de fato nesta sessão.
7. Se a task contiver uma seção `<skills>`, trate-a como acréscimo: leia também essas skills, desde que existam.

## Procedimento — MCPs

1. Execute a descoberta de `do-shared/references/do-mcp-discovery-instructions.md` (arquivo de configuração de MCP da ferramenta + tools expostas na sessão). Essa descoberta é a fonte de verdade em tempo de execução.
2. A seção `Available MCPs` do arquivo de configuração serve só como resumo para saber o que esperar; se divergir da descoberta, vale a descoberta.
3. Use um MCP somente se a capacidade correspondente foi descoberta. Sem MCP para a capacidade necessária, documente a lacuna e continue pelo caminho alternativo (testes unitários/integração, busca na web, leitura do código).
4. **Documentação de bibliotecas:** se existir um MCP com capacidade `documentation`, use-o para consultar a documentação oficial. Caso contrário, use busca na web ou o próprio código e a TechSpec. Nenhum MCP específico é obrigatório.

## Registro obrigatório

Ao final do trabalho, registre no artefato de saída (review, relatório de QA, task review ou TechSpec) uma linha curta:

```
Skills do projeto: <lidas: a, b | nenhuma aplicável | nenhuma instalada>
MCPs do projeto: <usados: x (capacidade) | nenhum necessário | lacuna: capacidade Y não disponível>
```

Isso é um **registro**, não uma exigência de usar skill ou MCP: "nenhuma instalada" é um resultado válido. O objetivo é tornar verificável que a consulta foi feita.

## Como localizar os arquivos do `do-shared` e de outras skills

- Os caminhos `do-shared/...` e `do-*/references|assets/...` citados nas skills são relativos ao **diretório de skills** (a pasta pai da skill atual, por exemplo `.agents/skills/` ou `.claude/skills/`), nunca à raiz do projeto nem à pasta do PRD.
- Ferramentas de busca por padrão (glob) **ignoram diretórios ocultos** como `.agents/`, `.claude/` e `.cursor/`. Não conclua que o arquivo não existe porque uma busca retornou vazio: leia o arquivo pelo caminho completo (`<diretório-de-skills>/do-shared/references/<arquivo>.md`) ou habilite arquivos ocultos na busca.
- Se o arquivo realmente não existir, siga a regra de erro da skill que o citou (por exemplo, parar e reportar a dependência ausente).

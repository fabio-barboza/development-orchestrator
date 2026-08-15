# Manual de Uso — DO Framework

Este manual mostra como usar o DO Framework do começo ao fim, passo a passo.

> Para detalhes completos de cada skill, consulte o [README.md](README.md).

> **Ferramentas testadas:** As skills e agents do DO Framework foram testados com **Opencode**, **Claude Code**, **Cursor** e **GitHub Copilot**. Por serem implementados como skills (instruções em arquivos de texto), possivelmente funcionam com outras ferramentas que suportem o mesmo mecanismo — mas não há garantia de compatibilidade.

## Economia de Tokens — Use o Modelo Certo em Cada Fase

Cada etapa do framework opera de forma completamente independente. O contexto de uma fase não é carregado na próxima — apenas os artefatos gerados (arquivos `.md`) são passados adiante. Isso permite uma estratégia de custo eficiente:

| Fase | Skills | Modelo recomendado | Por quê |
|------|--------|--------------------|---------|
| **Planejamento** | `do-create-prd`, `do-create-techspec`, `do-create-tasks` | Mais inteligente (ex: Opus, o1) | Requer raciocínio profundo, ambiguidade alta, decisões de arquitetura |
| **Execução** | `do-execute-task`, `do-execute-review`, `do-execute-qa`, `do-execute-qa-bugfix` | Menor e mais rápido (ex: Sonnet, Haiku) | Segue instruções precisas dos artefatos gerados, contexto já bem definido |

> Troque o modelo entre as fases sem perder contexto — os artefatos em `./prds/` são a memória do projeto.

---

## Passo 0 — Instalar as skills

Na raiz do seu projeto, execute:

```bash
npx skills add fabio-barboza/development-orchestrator
```

Isso instala todas as skills do framework (`do-setup`, `do-create-prd`, `do-create-techspec`, etc.) no seu ambiente de IA.

> ⚠️ **Reinicie a ferramenta após o `npx skills add`.** A maioria das ferramentas de IA (Claude Code, Cursor, GitHub Copilot, opencode) **não recarrega automaticamente** a lista de skills/comandos quando arquivos novos aparecem no disco. Se você abrir o autocomplete e os comandos `/do-*` não estiverem listados, **feche e reabra a ferramenta** (ou recarregue a janela do IDE / reinicie a sessão de chat). Isso vale tanto após o `npx skills add` quanto após o `do-setup` (que adiciona os slash commands em `.claude/commands/`, `.cursor/commands/`, `.github/prompts/` ou `.opencode/commands/`).

### Adicionar skills de tecnologia (opcional, mas recomendado)

Além das skills do DO Framework, você pode instalar skills específicas para a stack do seu projeto. Elas enriquecem o contexto técnico durante a criação de TechSpec e execução de tasks.

Exemplos:

```bash
# Frontend / JavaScript
npx skills add agentskills-io/javascript
npx skills add agentskills-io/typescript
npx skills add agentskills-io/react
npx skills add agentskills-io/nextjs

# Backend
npx skills add agentskills-io/golang
npx skills add agentskills-io/java
npx skills add agentskills-io/python

# Mobile
npx skills add agentskills-io/flutter
```

> Procure skills disponíveis em [skills.sh](https://skills.sh). Após instalar, o `do-setup` detecta automaticamente quais são relevantes ao projeto e as registra no arquivo de configuração.

---

## Passo 1 — Configurar MCPs (opcional, mas recomendado)

MCPs habilitam testes E2E automáticos. O mais comum é o Playwright (para browser):

**Claude Code** — adicione ao `.mcp.json` na raiz do projeto:

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    },
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

Para outras ferramentas (Cursor, Copilot), veja a tabela no [README.md](README.md#como-configurar-mcps-no-seu-projeto).

---

## Passo 2 — Inicializar o projeto

Rode **uma única vez** por projeto:

```
/do-setup
```

Isso analisa o codebase, gera o arquivo de configuração do projeto (`CLAUDE.md`, `.cursorrules`, etc.) com contexto e convenções, e **instala os agents de orquestração** nos diretórios corretos da sua ferramenta de IA.

> **Opencode (apenas na 1ª execução):** o opencode **não reconhece skills via `/<skill>`** — então a *primeira* invocação do `do-setup` precisa ser em linguagem natural:
> ```
> Execute do-setup
> ```
> O `do-setup` então copia **todos os slash commands `/do-*`** (`/do-setup`, `/do-create-prd`, `/do-create-techspec`, `/do-create-tasks`, `/do-execute-task`, `/do-execute-all-tasks`, etc.) para `.opencode/commands/`. **A partir daí, todos os comandos do framework funcionam com a sintaxe `/` normalmente no opencode** — inclusive `/do-setup` em re-execuções.

> Se você já fez o setup anteriormente e quer apenas reinstalar os agents (ex: após atualizar as skills), use:
> ```
> /do-setup agents
> ```

---

## Passo 3 — Criar o PRD

Descreva a feature que você quer construir:

```
/do-create-prd adicionar login com Google
# ou passando um arquivo de contexto:
/do-create-prd adicionar <arquivo>
```

A skill vai fazer algumas perguntas para entender o problema, objetivos e flows. Ao final, gera:

```
./prds/prd-login-google/prd.md
```

---

## Passo 4 — Criar a TechSpec

Com o PRD pronto, defina como implementar:

```
/do-create-techspec <prd>
```

A skill pesquisa documentação oficial (via Context7 MCP), explora o codebase e faz perguntas técnicas. Ao final, gera:

```
./prds/prd-login-google/techspec.md
```

---

## Passo 5 — Criar as Tasks

Com o TechSpec pronto, quebre o trabalho em tasks executáveis:

```
/do-create-tasks <techspec>
```

A skill apresenta uma lista de tasks para você aprovar antes de detalhar. Ao final, gera:

```
./prds/prd-login-google/tasks/tasks.md
./prds/prd-login-google/tasks/1_task.md
./prds/prd-login-google/tasks/2_task.md
...
```

---

## Passo 6 — Executar as Tasks

### Opção A — Uma por vez (controle manual)

Execute cada task em ordem:

```
/do-execute-task 1_task.md
/do-execute-task 2_task.md
/do-execute-task 3_task.md
... (até todas as tasks estarem [x] em tasks.md)
```

### Opção B — Em lote (agent autônomo)

Use o slash command `/do-execute-all-tasks` (a orquestração roda no agente primário da sessão e delega cada task ao subagente `agent-execute-task`) para executar um conjunto de tasks sequencialmente sem intervenção:

```
/do-execute-all-tasks prds/prd-login-google/tasks/tasks.md all
```

Você também pode executar um range ou uma lista específica:

```
/do-execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0-4.0
/do-execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0,3.0,5.0
```

> Executa **uma task por vez, estritamente sequencial** (nunca em paralelo — tasks do mesmo PRD editam os mesmos arquivos e o mesmo `tasks.md`), delegando cada execução a um subagente isolado (`agent-execute-task`) — isso evita estouro da janela de contexto em filas longas. Para automaticamente em caso de falha. Disponível no Claude Code, Cursor, GitHub Copilot e Opencode após o `do-setup`.
>
> ⚠️ Rode o command **na sessão primária**, não dentro de um subagente. Nenhuma ferramenta suportada garante subagente-de-subagente: o Claude Code proíbe, o Copilot desabilita por default, e no opencode a fila trava esperando prompts de permissão que nunca aparecem na TUI.

A skill (e o agent) implementam o código, rodam os testes e geram um review file por task. Para acompanhar o progresso a qualquer momento:

```
/do-status
```

---

## Passo 7 — Code Review (loop até APPROVED)

Com todas as tasks concluídas, rode o review geral:

```
/do-execute-review <techspec>
```

Se o resultado for `NEEDS_REVISION` ou `REJECTED`, corrija os findings e repita:

```
/do-execute-review-fix <tasks>
/do-execute-review <tasks>
```

Repita até o status ser `APPROVED`.

---

## Passo 8 — QA (loop até zero bugs HIGH)

Com o review aprovado, rode a validação final:

```
/do-execute-qa <prd>
```

Se bugs forem encontrados, corrija e revalide:

```
/do-execute-qa-bugfix <bugfix>
/do-execute-qa <prd>
```

Repita até não restar nenhum bug de severidade HIGH. Feature pronta!

---

## Resumo do Fluxo

```
instalar skills
      │
      ▼
  do-setup  (1x por projeto)
      │
      ▼
  do-create-prd
      │
      ▼
  do-create-techspec
      │
      ▼
  do-create-tasks
      │
      ▼
  do-execute-task 1..N  (uma por vez)
      │
      ▼
  do-execute-review  ──▶  do-execute-review-fix  ──▶  (repetir até APPROVED)
      │
      ▼
  do-execute-qa  ──▶  do-execute-qa-bugfix  ──▶  (repetir até zero HIGH)
      │
      ▼
   DONE ✓
```

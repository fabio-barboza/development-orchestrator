# Development Orchestrator (DO Framework)

**Framework de Skills AI-driven** que orquestra o ciclo completo de desenvolvimento de software: desde a ideia inicial até QA final com testes E2E automatizados via MCP (Model Context Protocol).

## Para Que Serve

O DO Framework elimina trabalho manual e garante qualidade consistente ao automatizar:

| Área                  | Benefício                                                                 |
|-----------------------|---------------------------------------------------------------------------|
| **Planejamento**      | PBIs padronizadas, critérios claros, sem ambiguidades                     |
| **Arquitetura**       | TechSpecs profundas com pesquisa automática em documentação oficial       |
| **Decomposição**      | Tasks atômicas e testáveis, ordenadas por dependência                     |
| **Execução**          | Implementação com testes E2E via MCP, review automático                   |
| **Review**            | Gate de qualidade com correção autônoma de findings antes do QA           |
| **QA**                | Validação sistemática, screenshots como evidência, bugs documentados      |
| **Bugfix**            | Correção com testes de regressão, priorizada por severidade               |
| **Visibilidade**      | Status de progresso em tempo real, retomada de trabalho sem fricção       |

Resultado: features entregues mais rápido, com menos retrabalho e qualidade verificável.

## Instalação

### Instalar as Skills do DO Framework

Para instalar todas as skills deste framework no seu ambiente:

```bash
npx skills add fabio-barboza/development-orchestrator
```

Este comando irá:
1. Baixar todas as skills (`do-setup`, `do-create-pbi`, `do-create-techspec`, etc.)
2. Configurar no seu ambiente de agente
3. Fazer disponível para uso imediato nos seus projetos


## Fluxo do Framework — Pipeline Completa

### FASE 1: PLANEJAMENTO

```
┌──────────┐    ┌───────────┐    ┌──────────────┐    ┌─────────────┐
│ do-setup │───▶│do-create- │───▶│do-create-    │───▶│do-create-   │
│  (1x)    │    │   pbi     │    │  techspec    │    │   tasks     │
└──────────┘    └───────────┘    └──────────────┘    └─────────────┘
                                                              │
                                                              ▼
```

### FASE 2: EXECUÇÃO (Loop por Task)

```
                    ┌─────────────────────────────────────────────┐
                    │       LOOP POR CADA TASK NA ORDEM           │
                    │                                             │
                   ◀│◀   do-execute-task [num]                    │
tasks.md            │   - Implementação + tests                   │
[15 tasks] ────────▶│   - E2E via MCP se aplicável                │
(sequencial)        │   - Gera [num]_task_review.md               │
                    │   - Marca task como [x] em tasks.md         │
                    └─────────────────────────────────────────────┘
                                    │
                          (todas tasks completas)
                                    ▼
```

### FASE 3: CODE REVIEW GERAL (Loop até Aprovado)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LOOP DE REVIEW                                           │
│                                                                             │
│   ┌──────────────────────┐                                                  │
│   │  do-execute-review   │                                                  │
│   │                      │                                                  │
│   │  - Conformidade com  │                                                  │
│   │    TechSpec          │                                                  │
│   │  - Suite de testes   │                                                  │
│   │  - E2E via MCP       │──────────────┐                                   │
│   │  - Padrões de código │              │                                   │
│   └──────────┬───────────┘              │                                   │
│              │                          │                                   │
│  (NEEDS_REVISION / REJECTED)            │ (APPROVED)                        │
│              ▼                          │                                   │
│   ┌──────────────────────┐              │                                   │
│   │ do-execute-review-fix│              │                                   │
│   │                      │              │                                   │
│   │  - Corrige CRITICAL  │              │                                   │
│   │    e MAJOR findings  │              │                                   │
│   │  - Roda testes       │              │                                   │
│   │  - Atualiza report   │              │                                   │
│   └──────────┬───────────┘              │                                   │
│              │                          │                                   │
│              └──▶ (repetir review) ◀────┘                                   │
│                   até APPROVED                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                          (review aprovado)
                                    ▼
```

### FASE 4: VALIDAÇÃO E2E + BUGFIX (Loop até Zero Bugs HIGH)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LOOP DE QUALIDADE                                        │
│                                                                             │
│   ┌──────────────────────┐                                                  │
│   │  do-execute-qa       │                                                  │
│   │                      │                                                  │
│   │  - E2E completo via  │                                                  │
│   │    MCP (Playwright)  │                                                  │
│   │  - Checklist PBI     │                                                  │
│   │  - Accessibility     │────────────┐                                     │
│   │  - Visual verification│           │                                     │
│   │  - Gera bugs.md      │            │                                     │
│   └──────────┬───────────┘            │                                     │
│              │                        │                                     │
│              │ (se houver bugs)       │ (zero bugs HIGH)                    │
│              ▼                        │                                     │
│   ┌──────────────────────┐            │                                     │
│   │  do-execute-qa-bugfix│◀───────────┘                                     │
│   │                      │                                                  │
│   │  - Corrige por       │                                                  │
│   │    severidade        │                                                  │
│   │  - Tests de regressão│                                                  │
│   │  - E2E via MCP       │                                                  │
│   │  - Atualiza bugs.md  │                                                  │
│   └──────────┬───────────┘                                                  │
│              │                                                              │
│              └─────────────▶ (repetir do-execute-qa) ◀──────────────────────┘
│                              até zero bugs HIGH
└─────────────────────────────────────────────────────────────────────────────┘
```

### Visão Geral do Fluxo

```
PLANEJAMENTO       EXECUÇÃO           REVIEW GERAL      VALIDAÇÃO
    │                   │                   │                  │
    ▼                   ▼                   ▼                  ▼
┌───────┐          ┌───────────┐       ┌─────────────┐     ┌──────────────┐
│PBI +  │─────────▶│ Loop por  │──────▶│ Review      │────▶│ QA + Bugfix  │
│Tasks  │          │ cada task │       │ (loop até   │     │ (loop até    │
│       │          │           │       │ aprovado)   │     │ zero HIGH)   │
└───────┘          └───────────┘       └─────────────┘     └──────────────┘
```

## Skills do Framework — O Que Cada Uma Faz

### `do-setup` — Inicialização do Projeto (1x)

**Quando usar:** Primeira vez no projeto ou ao reinstalar o ambiente.

**O que faz:**
1. Executa o comando de inicialização da ferramenta de IA (ex: `/init` no Claude Code), se disponível
2. Analisa profundamente o codebase (tech stack, arquitetura, padrões)
3. Verifica infraestrutura de testes — avisa se não houver test runner configurado
4. Identifica skills tecnológicas relevantes disponíveis
5. Atualiza o arquivo de configuração do projeto com summary do projeto e convenções

**Output:** Arquivo de configuração do projeto (`CLAUDE.md`, `.github/copilot-instructions.md`, ou equivalente) atualizado com contexto do projeto

---

### `do-create-pbi` — Criar Product Backlog Item

**Quando usar:** Nova feature, melhoria ou correção a ser desenvolvida.

**O que faz:**
1. **Clarification (interativo)**: Perguntas ao usuário sobre problema, objetivos, usuários, flows
2. **Planning (interativo)**: Apresenta plano de seções para aprovação
3. **Drafting**: Gera PBI padronizada com requisitos numerados e critérios de aceitação

**Output:** `./pbis/pbi-[slug]/pbi.md`

**Foco:** WHAT e WHY (nunca HOW — isso é do TechSpec)

---

### `do-create-techspec` — Especificação Técnica

**Quando usar:** PBI já criada, hora de definir arquitetura e implementação.

**O que faz:**
1. Analisa PBI profundamente
2. Explora codebase para entender contexto técnico (chamadas, interfaces, persistence)
3. **Research via MCP**: Usa Context7 para consultar documentação oficial de frameworks
4. **Clarifications (interativo)**: Perguntas técnicas focadas ao usuário
5. Gera TechSpec com arquitetura, component design, data models, endpoints, test strategy, **suposições técnicas** e riscos conhecidos

**Output:** `./pbis/pbi-[slug]/techspec.md`

**Foco:** HOW (implementação, não requisitos de negócio)

---

### `do-create-tasks` — Decomposição em Tarefas

**Quando usar:** TechSpec pronta, hora de criar tasks executáveis.

**O que faz:**
1. Analisa PBI + TechSpec
2. Research via Context7 para estimar complexidade das libs envolvidas
3. **High-level list (interativo)**: Apresenta lista de tasks para aprovação
4. Gera files com tasks detalhadas e subtasks numeradas

**Output:**
- `./pbis/pbi-[slug]/tasks/tasks.md` (index)
- `./pbis/pbi-[slug]/tasks/[num]_task.md` (cada task)

**Regra de ouro:** Cada task deve ser funcional, ter seus próprios testes, e representar ~100–200 linhas de mudança de código (para evitar context overflow na execução).

---

### `do-execute-task` — Implementar Task

**Quando usar:** Task específica para implementar (ex: "execute task 3").

**O que faz:**
1. Le PBI, TechSpec, tasks.md e `[num]_task.md`
2. **MCP Discovery**: Descobre quais MCPs estão configurados e aplicável
3. Implementa com testes unit/integration
4. **E2E via MCP**: Roda testes E2E usando tools do MCP (Playwright para browser, RabbitMQ para message queue, etc.)
5. Code review automático contra checklist de padrões
6. Marca task como `[x]` em `tasks.md`
7. **Cria review artifact obrigatório**

**Output:**
- Implementação no codebase
- `[num]_task_review.md` (MANDATÓRIO — sem isso, task NÃO está completa)

**Regras críticas:**
- TESTES DEVEM PASSAR — tarefa NUNCA completa com testes failing
- Review file MANDATÓRIO — sem artifact = não conta
- Execução AUTÔNOMA — zero pausas para confirmação do usuário

---

### `do-execute-review` — Code Review Geral (Gate antes do QA)

**Quando usar:** Todas tasks implementadas ([x] em tasks.md), obrigatório antes de ir para QA.

**O que faz:**
1. Le git diff ou lista manual de files modified durante as tasks
2. Verifica conformidade com TechSpec e padrões de código do projeto
3. Roda test suite completa (unit, integration, typecheck)
4. **E2E via MCP**: Valida features frontend/backend conforme MCPs disponíveis
5. Classifica issues: CRITICAL / MAJOR / MINOR / POSITIVE / **DESVIO JUSTIFICADO** (divergência técnica válida da TechSpec — não reprova, mas recomenda atualizar a TechSpec)
6. Gera review report com status final

**Loop de Correção:**
- Se status = **NEEDS_REVISION** ou **REJECTED**: rodar `do-execute-review-fix` → depois `do-execute-review` novamente
- Se status = **APPROVED**: prossegue para FASE 4 (QA)

**Output:** `./pbis/pbi-[slug]/review-report.md`

**Status:** ACCEPTED / NEEDS_REVISION / REJECTED

---

### `do-execute-review-fix` — Corrigir Findings do Code Review

**Quando usar:** Após `do-execute-review` retornar NEEDS_REVISION ou REJECTED.

**O que faz:**
1. Lê `review-report.md` e extrai todos os findings por severidade
2. Implementa correções em ordem: CRITICAL → MAJOR → MINOR (opcional)
3. Roda suite completa de testes — task nunca completa com testes failing
4. Atualiza `review-report.md` com status de cada finding (Fixed / Unresolved)
5. Gera fix report com resumo das correções aplicadas

**Output:**
- Correções no codebase
- `review-report.md` atualizado
- `./pbis/pbi-[slug]/fix-report.md`

---

### `do-execute-qa` — Quality Assurance E2E

**Quando usar:** Implementação completa, hora de validação final.

**O que faz:**
1. Cria checklist baseado nos requisitos do PBI
2. **MCP Discovery**: Identifica MCPs disponíveis e aplica capability guard
3. **E2E Tests via MCP**: Roda flows completos capturando screenshots
4. **Accessibility**: Verifica WCAG 2.2 (keyboard nav, labels, contrast, etc.)
5. **Visual verification**: Screenshots de estados diferentes
6. Documenta bugs encontrados com evidência visual

**Output:**
- `./pbis/pbi-[slug]/qa-report.md`
- `./pbis/pbi-[slug]/bugs.md` (se houver bugs)

**Status:** APPROVED apenas se TODOS requisitos verificados e funcionando

---

### `do-status` — Progresso do PBI

**Quando usar:** Para verificar o estado atual de um PBI, retomar trabalho após interrupção, ou saber qual é a próxima task.

**O que faz:**
1. Identifica o PBI alvo (pelo slug ou por seleção automática)
2. Lê `tasks.md` e calcula progresso (X/Y tasks, Z%)
3. Identifica a próxima task pendente e verifica dependências
4. Detecta tasks marcadas como `[x]` mas sem review file (incompletas)

**Output:** Relatório de progresso em tela (não gera arquivos)

**Skill somente leitura** — não modifica nenhum arquivo.

---

### `do-execute-qa-bugfix` — Correção de Bugs

**Quando usar:** Bugs documentados em `bugs.md` precisam ser corrigidos.

**O que faz:**
1. Le todos bugs de `bugs.md`
2. Analisa causa raiz de cada um
3. Implementa correções por severidade (HIGH → MEDIUM → LOW)
4. Cria testes de regressão para cada bug
5. **E2E via MCP**: Valida correção no fluxo completo
6. Atualiza `bugs.md` com status "Fixed" e descrição do fix
7. Gera report final

**Output:**
- Correções no codebase
- `bugs.md` atualizado
- `./pbis/pbi-[slug]/bugfix-report.md`

---

## MCP Integration — Testes E2E Dinâmicos

O DO Framework usa **Model Context Protocol (MCP)** para descoberta dinâmica de ferramentas de teste. Não é hardcoded — tudo configurável via registry.

### Como Funciona a Descoberta

1. Le o arquivo de configuração MCP da ferramenta de IA (`.mcp.json`, `.vscode/mcp.json`, etc.) → lista MCP servers configurados
2. Le `do-mcp-capabilities.md` → mapeia cada server a capacidades e tools
3. Constroi mapa interno: `{ "browser-testing": ["playwright"], "message-queue": ["rabbitmq"] }`
4. Aplica **capability guard**: usa MCP disponível conforme tipo da feature

### Capability Guard — Tabela de Decisão

| Tipo Feature | MCP Disponível | Ação |
|--------------|----------------|------|
| Frontend + `browser-testing` | Sim | Rodar E2E browser via Playwright MCP |
| Backend + `message-queue`/`database`/`cache` | Sim | Rodar E2E backend via MCP correspondente |
| Frontend + Backend + ambos | Sim | Rodar ambos E2Es |
| Feature + nenhum MCP relevante | Não | Pular E2E, documentar gap, continuar com unit/integration |

### Como Configurar MCPs no Seu Projeto

A localização e o formato do arquivo de configuração de MCPs **varia conforme a ferramenta de IA** utilizada:

| Ferramenta | Arquivo de Configuração | Chave principal |
|-----------|------------------------|----------------|
| **Claude Code** | `.mcp.json` (raiz do projeto) ou `~/.mcp.json` (global) | `mcpServers` |
| **GitHub Copilot** | `.vscode/mcp.json` | `servers` |
| **Cursor** | `.cursor/mcp.json` | `mcpServers` |

Os MCPs disponíveis para o DO Framework estão documentados em `skills/do-shared/do-mcp-capabilities.md`. Você pode adicioná-los no arquivo correspondente à sua ferramenta.

Obs: Você pode adicionar novos MCPs e configurá-los no `skills/do-shared/do-mcp-capabilities.md`

**Claude Code — `.mcp.json` (raiz do projeto):**

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "env": {}
    },
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"],
      "env": {}
    },
    "rabbitmq": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "mcp-server-rabbitmq@latest",
        "--rabbitmq-host", "localhost",
        "--port", "5672",
        "--username", "guest",
        "--password", "guest",
        "--api-port", "15672"
      ]
    }
  }
}
```

**GitHub Copilot — `.vscode/mcp.json`:**

```json
{
  "servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    },
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "rabbitmq": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "mcp-server-rabbitmq@latest",
        "--rabbitmq-host", "localhost",
        "--port", "5672",
        "--username", "guest",
        "--password", "guest",
        "--api-port", "15672"
      ]
    }
  }
}
```

**MCPs Disponíveis (já documentados em `do-mcp-capabilities.md`):**

| MCP | Capacidade | Quando Usar |
|-----|------------|-------------|
| `playwright` | browser-testing | E2E frontend, validação visual, accessibility |
| `context7` | documentation research | Pesquisa de documentação oficial durante TechSpec/Tasks |
| `rabbitmq` | message-queue validation | Validação de filas e mensagens em background jobs |

### Registry de Capacidades

**Arquivo:** `skills/do-shared/do-mcp-capabilities.md`

Documenta cada MCP com:
- Capacidades (browser-testing, documentation, message-queue, etc.)
- Prefixo de tools (`mcp__playwright__browser_*`)
- Quando usar
- Requer app rodando? (Sim/Não)
- Handling se indisponível
- Lista de tools principais

### Adicionar Novo MCP (Zero Code Changes)

1. Configurar no arquivo MCP da sua ferramenta (`.mcp.json`, `.vscode/mcp.json`, etc.)
2. Adicionar entrada em `do-mcp-capabilities.md`
3. **Pronto** — skills descobrem automaticamente, nenhuma edição nas skills necessária

## Prerequisitos do Ambiente

### Ferramentas Obrigatórias (Comuns a Todas as Stacks)

```bash
# Node.js + npx (necessário para MCPs)
node --version  # 18+
npm --version   # 9+

# Git (para versionamento e diff analysis)
git --version
```

### Ferramentas Específicas por Stack

```bash
# Para projetos React/Node.js
npm install -g supabase  # Supabase CLI (opcional, para DB operations)

# Para projetos Flutter
flutter --version

# Para projetos Python
python3 --version

# Escolha conforme sua stack
```

### Ferramentas Opcionais (para MCPs)

```bash
# uv (para MCPs Python-based como RabbitMQ)
uvx --version
# Install: curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Configurar um MCP

**Exemplo: Playwright (browser-testing)**

1. Adicionar no arquivo de configuração MCP da sua ferramenta (veja a tabela acima):
   ```json
   {
     "playwright": {
       "type": "stdio",
       "command": "npx",
       "args": ["@playwright/mcp@latest"],
       "env": {}
     }
   }
   ```

2. Adicionar entrada em `do-mcp-capabilities.md`:
   ```markdown
   ### playwright
   - **Capacidades:** browser-testing
   - **Prefixo de tools:** mcp__playwright__browser_*
   - **Quando usar:** E2E frontend, validação visual
   - **Requer app rodando:** Sim
   - **Se indisponível:** Reportar erro, documentar gap no relatório
   ```

3. Instalar dependências se necessário:
   ```bash
   npx @playwright/mcp@latest --help  # Verifica install
   ```

## Estrutura de Pastas Gerada

```
./pbis/
└── pbi-[feature-slug]/
    ├── pbi.md                  # Product Backlog Item (do-create-pbi)
    ├── techspec.md             # Tech Spec (do-create-techspec)
    ├── bugs.md                 # Bugs encontrados (do-execute-qa/bugfix)
    ├── qa-report.md            # Relatório QA (do-execute-qa)
    ├── review-report.md        # Relatório Review (do-execute-review)
    ├── fix-report.md           # Relatório de correções do review (do-execute-review-fix)
    ├── bugfix-report.md        # Relatório Bugfix (do-execute-qa-bugfix)
    └── tasks/
        ├── tasks.md            # Index de tasks (do-create-tasks)
        ├── 1_task.md           # Task 1 detalhada
        ├── 1_task_review.md    # Review artifact task 1
        ├── 2_task.md
        ├── 2_task_review.md
        └── ...
```

## Convenções do Framework

| Convenção | Descrição |
|-----------|-----------|
| **Prefixo `do-`** | Todas skills usam `do-` para identificar como parte do framework |
| **Slug kebab-case** | Pastas usam kebab-case: `pbi-projeto-exemplo`, `pbi-user-auth` |
| **Numeração de tasks** | Tasks numeradas sequencialmente: `1_task.md`, `2_task.md` |
| **Artefatos obrigatórios** | Sem artifact = task incompleta (review files, reports) |
| **Testes devem passar** | Task NUNCA completa com failing tests |
| **Execução autônoma** | Skills executam sem pausas para confirmação (exceto steps interativos explícitos) |
| **Língua dos artifacts** | PT-BR para documentos, English para code/tech terms |

## Quando Usar Cada Skill — Guia Rápido

> Os exemplos abaixo usam a sintaxe do **Claude Code** (`/skill-name`). Consulte a documentação da sua ferramenta para a forma de invocar skills (GitHub Copilot, Cursor, etc. possuem sua própria sintaxe de invocação).

| Cenário | Skill | Loop? |
|---------|-------|-------|
| Novo projeto, primeira vez | `do-setup` | Não (1x) |
| Nova feature (ideia vaga) | `do-create-pbi` | Não |
| PBI criada, definir arquitetura | `do-create-techspec` | Não |
| TechSpec pronta, criar tasks | `do-create-tasks` | Não |
| Verificar progresso / retomar trabalho | `do-status` | Não |
| Implementar task específica | `do-execute-task 1` | Sim (por cada task) |
| **Review geral (OBRIGATÓRIO antes do QA)** | `do-execute-review` | **Sim (até APPROVED)** |
| Corrigir findings do review | `do-execute-review-fix` | **Sim (com do-execute-review)** |
| QA E2E da feature completa | `do-execute-qa` | Sim (com bugfix) |
| Corrigir bugs encontrados no QA | `do-execute-qa-bugfix` | **Sim (até zero HIGH)** |

## Fluxo Completo — Exemplo Prático

**Feature:** "Modo escuro no app"

> Os comandos abaixo usam a sintaxe do **Claude Code** (prefixo `/`). Para outras ferramentas, adapte a invocação conforme necessário.

```bash
# FASE 1: PLANEJAMENTO
# ------------------------------------------------
# 1. Criar PBI
/do-create-pbi *descrição ou arquivo de contexto*
> Feature: Projeto Exemplo
> [responde perguntas sobre problema, usuários, flows]

# 2. Criar TechSpec
/do-create-techspec projeto-exemplo
> [pesquisa docs via Context7 MCP + responde clarifications técnicas]

# 3. Criar Tasks
/do-create-tasks projeto-exemplo
> [aprova high-level task list]

# FASE 2: EXECUÇÃO
# ------------------------------------------------
# 4. Verificar progresso a qualquer momento (opcional, mas recomendado)
/do-status  # mostra X/Y tasks, próxima task, artefatos ausentes

# 5. Executar cada task em sequência (loop)
/do-execute-task 1  # implementa + testa + review file
/do-execute-task 2  # implementa + testa + review file
/do-execute-task 3  # implementa + testa + review file
... (até todas tasks [x] em tasks.md)

# FASE 3: CODE REVIEW GERAL (LOOP até APPROVED)
# ------------------------------------------------
# 5. Review geral do PBI (OBRIGATÓRIO antes do QA)
/do-execute-review projeto-exemplo
> Status: NEEDS_REVISION / REJECTED? → /do-execute-review-fix projeto-exemplo → /do-execute-review projeto-exemplo
> Status: APPROVED? → prossegue para QA

# FASE 4: VALIDAÇÃO E2E + BUGFIX (LOOP até zero bugs HIGH)
# ------------------------------------------------
# 6. QA final com E2E completo
/do-execute-qa projeto-exemplo
> [roda E2E via Playwright MCP, accessibility, visual verification]
> [gera qa-report.md + bugs.md se houver problemas]

# 7. Se houver bugs HIGH/MEDIUM: loop de correção
[se bugs.md tem entries] → /do-execute-qa-bugfix projeto-exemplo
> [corrige por severidade + tests de regressão]

# 8. Revalida com QA novamente
/do-execute-qa projeto-exemplo
> [reteste apenas areas corrigidas ou E2E completo]

# 9. Repetir steps 7-8 até zero bugs HIGH
[se ainda tem bugs HIGH] → /do-execute-qa-bugfix projeto-exemplo → /do-execute-qa projeto-exemplo
[zero bugs HIGH] → FEATURE READY! ✅
```

## Referências

| Arquivo | Propósito |
|---------|-----------|
| `do-mcp-capabilities.md` | Registry central de MCPs e suas capacidades |
| `do-mcp-discovery-instructions.md` | Procedimento de descoberta dinâmica de MCPs |
| Arquivo de configuração MCP (`.mcp.json`, `.vscode/mcp.json`, etc.) | Configuração de MCP servers ativos no projeto — localização varia por ferramenta de IA |
| Arquivo de configuração do projeto (`CLAUDE.md`, `.github/copilot-instructions.md`, etc.) | Contexto do projeto gerado por `do-setup` — nome varia por ferramenta de IA |

## Compatibilidade com Ferramentas de IA

O DO Framework é agnóstico à ferramenta de IA. Os conceitos, o fluxo de trabalho e os artefatos gerados funcionam com qualquer ferramenta que suporte skills/instruções customizadas.

**Convenções de caminho**: Os arquivos de skill referenciam internamente o diretório `.claude/skills/` (convenção do Claude Code). Se você usa outra ferramenta, os arquivos estarão no diretório equivalente dessa ferramenta (ex: `.github/` para GitHub Copilot).

| Ferramenta | Diretório de Skills | Config do Projeto |
|-----------|--------------------|--------------------|
| **Claude Code** | `.claude/skills/` | `CLAUDE.md` |
| **GitHub Copilot** | `.github/` (instructions) | `.github/copilot-instructions.md` |
| **Cursor** | `.cursor/rules/` | `.cursorrules` |

---

## Dicas Importantes

1. **Não pule steps**: PBI → TechSpec → Tasks → Execute (ordem é importante)
2. **MCPs são opcionais mas recomendados**: Sem MCP, perde-se E2E automatizado
3. **Review files são obrigatórios**: Sem `[num]_task_review.md`, a task NÃO está completa
4. **Testes failing bloqueiam progresso**: Não prossiga até todos passarem
5. **Bugs HIGH devem ser corrigidos antes de considerar feature done**
6. **Use `do-status` para retomar trabalho**: Após qualquer interrupção, rode `do-status` para saber exatamente onde parou
7. **Documente suposições na TechSpec**: Suposições não documentadas viram surpresas na task 7 de 12

---

## Sobre o Autor

**Fabio Barboza de Oliveira** — Desenvolvedor Senior especializado em arquitetura de software e desenvolvimento de ferramentas que aumentam a produtividade de equipes de engenharia.

### Sobre mim

Desenvolvedor com experiência em criar soluções que automatizam processos e melhoram a qualidade de código. Este framework (DO - Development Orchestrator) representa minha abordagem para orquestrar o ciclo completo de desenvolvimento de software, desde o planejamento até a validação final.

### Contato

- **LinkedIn:** [linkedin.com/in/fabio-oliveira-20a977a1](https://www.linkedin.com/in/fabio-oliveira-20a977a1/)
- **Email:** barboza.oliveira@gmail.com

---

## Licença

Este projeto é software de código aberto.

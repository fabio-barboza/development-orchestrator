# Development Orchestrator (DO Framework)

**Framework de Skills AI-driven** que orquestra o ciclo completo de desenvolvimento de software seguindo o padrão **Spec Driven Development (SDD)**: desde a ideia inicial até QA final com testes E2E automatizados via MCP (Model Context Protocol).

O SDD garante que cada feature seja especificada antes de implementada — PRD → TechSpec → Tasks — criando uma cadeia de artefatos rastreáveis que eliminam ambiguidade e orientam a execução com contexto completo.

> **Novo por aqui?** Veja o [Manual de Uso](MANUAL.md) para um guia passo a passo simplificado.

## Para Que Serve

O DO Framework elimina trabalho manual e garante qualidade consistente ao automatizar:

| Área                  | Benefício                                                                 |
|-----------------------|---------------------------------------------------------------------------|
| **Planejamento**      | PRDs padronizados, critérios claros, sem ambiguidades                     |
| **Arquitetura**       | TechSpecs profundas com pesquisa automática em documentação oficial       |
| **Decomposição**      | Tasks atômicas e testáveis, ordenadas por dependência                     |
| **Execução**          | Implementação com testes de unidade e integração (e fumaça via MCP), review automático |
| **Review**            | Gate de qualidade só de código; findings corrigidos em lote por subagentes isolados, no lote que você escolher |
| **QA**                | Único dono do E2E, acessibilidade e análise visual; screenshots como evidência, bugs documentados e corrigidos em lote |
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
1. Baixar todas as skills (`do-setup`, `do-create-prd`, `do-create-techspec`, etc.)
2. Configurar no seu ambiente de agente
3. Fazer disponível para uso imediato nos seus projetos

> ⚠️ **Se as skills ou os comandos `/do-*` não aparecerem após o `npx skills add` ou o `do-setup`, recarregue a ferramenta.** O jeito depende dela:
>
> | Ferramenta | Como recarregar |
> |-----------|-----------------|
> | **Opencode** | `/reload` |
> | **Claude Code** | Skills novas entram sozinhas; se a pasta `.claude/skills/` foi criada agora, rode `/reload-skills`. A pasta `.claude/agents/` criada pelo `do-setup` só é detectada **ao reiniciar a sessão** |
> | **Cursor** e **GitHub Copilot** | A documentação não especifica. Se não aparecerem, recarregue a janela do IDE ou abra uma nova sessão de chat |


## Fluxo do Framework — Pipeline Completa

### FASE 1: PLANEJAMENTO

```
┌──────────┐    ┌───────────┐    ┌──────────────┐    ┌─────────────┐
│ do-setup │───▶│do-create- │───▶│do-create-    │───▶│do-create-   │
│  (1x)    │    │   prd     │    │  techspec    │    │   tasks     │
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
[N tasks]  ────────▶│   - fumaça via MCP se aplicável             │
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
│   │  do-execute-review   │◀─────────────┐                                   │
│   │                      │              │                                   │
│   │  - Conformidade com  │              │                                   │
│   │    TechSpec          │              │                                   │
│   │  - Suite de testes   │              │                                   │
│   │  - Sem E2E (é do QA) │              │ (repetir review)                  │
│   │  - Padrões de código │              │                                   │
│   └──────────┬───────────┘              │                                   │
│              │                          │                                   │
│              ├─── (APROVADO) ─────────────────────────────────▶ sai do loop │
│              │                          │                                   │
│  (REPROVADO / COM RESSALVAS)            │                                   │
│              ▼                          │                                   │
│   ┌──────────────────────┐              │                                   │
│   │ do-execute-review-fix│              │                                   │
│   │                      │              │                                   │
│   │  - Corrige CRITICAL  │              │                                   │
│   │    e MAJOR findings  │              │                                   │
│   │  - Roda testes       │              │                                   │
│   │  - Atualiza report   │              │                                   │
│   └──────────┬───────────┘              │                                   │
│              └──────────────────────────┘                                   │
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
│   │  do-execute-qa       │◀─────────────┐                                   │
│   │                      │              │                                   │
│   │  - E2E completo via  │              │                                   │
│   │    MCP (ui-testing)  │              │                                   │
│   │  - Checklist PRD     │              │ (repetir do-execute-qa)           │
│   │  - Accessibility     │              │                                   │
│   │  - Verificação visual│              │                                   │
│   │  - Gera qa/qa-N/bugs/│              │                                   │
│   └──────────┬───────────┘              │                                   │
│              │                          │                                   │
│              ├─── (zero bugs HIGH) ───────────────────────────▶ sai do loop │
│              │                          │                                   │
│       (se houver bugs)                  │                                   │
│              ▼                          │                                   │
│   ┌──────────────────────┐              │                                   │
│   │  do-execute-qa-fix   │              │                                   │
│   │                      │              │                                   │
│   │  - Corrige por       │              │                                   │
│   │    severidade        │              │                                   │
│   │  - Tests de regressão│              │                                   │
│   │  - E2E via MCP       │              │                                   │
│   │  - Atualiza o bug    │              │                                   │
│   └──────────┬───────────┘              │                                   │
│              └──────────────────────────┘                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Visão Geral do Fluxo

```
PLANEJAMENTO       EXECUÇÃO           REVIEW GERAL      VALIDAÇÃO
    │                   │                   │                  │
    ▼                   ▼                   ▼                  ▼
┌───────┐          ┌───────────┐       ┌─────────────┐     ┌──────────────┐
│PRD +  │─────────▶│ Loop por  │──────▶│ Review      │────▶│ QA + Bugfix  │
│Tasks  │          │ cada task │       │ (loop até   │     │ (loop até    │
│       │          │           │       │ aprovado)   │     │ zero HIGH)   │
└───────┘          └───────────┘       └─────────────┘     └──────────────┘
```

## Skills do Framework — O Que Cada Uma Faz

### `do-setup` — Inicialização do Projeto (1x)

**Quando usar:** Primeira vez no projeto ou ao reinstalar o ambiente.

**O que faz:**
1. Cria a pasta `.do/` e, se o projeto é um repositório git, acrescenta `.do/` ao `.gitignore`, sem tocar em mais nada
2. Analisa profundamente o codebase (tech stack, arquitetura, padrões), lendo o arquivo de instruções do projeto (`CLAUDE.md`, `AGENTS.md` etc.) se existir
3. Verifica infraestrutura de testes — avisa se não houver test runner configurado
4. Identifica skills tecnológicas relevantes disponíveis e inventaria os MCPs
5. Grava `.do/context.md` com resumo do projeto, convenções, comandos, skills e MCPs
6. Se o projeto não tem nenhum arquivo de instruções, pergunta se deve criar um (`CLAUDE.md` no Claude Code, `AGENTS.md` nas demais) só com resumo, comandos e convenções, sem referências ao framework
7. **Instala os agents e commands de orquestração** nos diretórios corretos da ferramenta detectada

**Output:** `.do/context.md`, local a cada desenvolvedor e ignorado pelo git. O `do-setup` **nunca** altera um `CLAUDE.md`, `AGENTS.md` ou equivalente que já existe: esses arquivos são do projeto e de todo o time, e a lista de skills e MCPs só vale para a máquina de quem rodou o setup. Quando o projeto não tem nenhum, ele pergunta e, se você aceitar, cria o arquivo com o conteúdo que um `/init` geraria (resumo, comandos, convenções, em inglês, como o `/init`), sem nada do framework e sem adicioná-lo ao git.

**Argumento opcional:** `/do-setup agents` — pula a análise do projeto e instala apenas os agents/commands (útil quando o setup já foi feito anteriormente)

**Argumento opcional:** `/do-setup refresh` — redetecta as skills e os MCPs instalados (depois de instalar novos) e atualiza só as seções `Available Skills`, `Uncovered Skills` e `Available MCPs` de `.do/context.md`. O `do-setup` também **informa** no relatório final as skills e MCPs detectados e as lacunas (por exemplo, sem MCP de interface o E2E de interface é pulado). Nenhuma skill além das `do-*` é presumida: se não houver nenhuma, o resultado é só um aviso.

> **Opencode (apenas na 1ª execução):** ao contrário das outras ferramentas (Claude Code, Cursor, GitHub Copilot), o opencode **não reconhece skills via `/<skill>`**. Por isso, a *primeira* invocação do `do-setup` precisa ser feita em linguagem natural:
> ```
> Execute do-setup
> ```
> O `do-setup` então copia **todos os slash commands `/do-*`** do framework (`/do-setup`, `/do-create-prd`, `/do-create-techspec`, `/do-create-tasks`, `/do-execute-task`, `/do-execute-all-tasks`, etc.) para `.opencode/commands/`. **A partir daí, todos os comandos passam a funcionar normalmente com a sintaxe `/` no opencode** — inclusive `/do-setup` em re-execuções e `/do-setup agents`.

---

### `do-create-prd` — Criar Product Requirements Document

**Quando usar:** Nova feature, melhoria ou correção a ser desenvolvida.

**O que faz:**
1. **Clarification (interativo)**: Perguntas ao usuário sobre problema, objetivos, usuários, flows
2. **Planning (interativo)**: Apresenta plano de seções para aprovação
3. **Drafting**: Gera PRD padronizado com requisitos numerados e critérios de aceitação

**Output:** `./prds/prd-[slug]/prd.md`

**Foco:** WHAT e WHY (nunca HOW — isso é do TechSpec)

---

### `do-create-techspec` — Especificação Técnica

**Quando usar:** PRD já criado, hora de definir arquitetura e implementação.

**O que faz:**
1. Analisa PRD profundamente
2. Explora codebase para entender contexto técnico (chamadas, interfaces, persistence)
3. **Research via MCP**: Usa um MCP de documentação (capacidade `documentation`), se disponível, para consultar documentação oficial; sem ele, usa busca na web
4. **Clarifications (interativo)**: Perguntas técnicas focadas ao usuário
5. Gera TechSpec com arquitetura, component design, data models, endpoints, test strategy, **cenários de aceite** (os fluxos que o `do-execute-qa` valida depois; não viram tasks), **suposições técnicas** e riscos conhecidos

**Output:** `./prds/prd-[slug]/techspec.md`

**Foco:** HOW (implementação, não requisitos de negócio)

---

### `do-create-tasks` — Decomposição em Tarefas

**Quando usar:** TechSpec pronta, hora de criar tasks executáveis.

**O que faz:**
1. Analisa PRD + TechSpec
2. Pesquisa de documentação (MCP `documentation`, se disponível) para estimar complexidade das libs envolvidas
3. **High-level list (interativo)**: Apresenta lista de tasks para aprovação
4. Gera files com tasks detalhadas e subtasks numeradas

**Sem task de E2E:** o framework não cria task de validação de ponta a ponta (nem final, nem por área). Cada task valida só o seu entregável, com testes de unidade e integração e, se entrega interface ou serviço, um teste de fumaça. A fumaça exercita o fluxo principal de sucesso com o sistema no ar (nunca com mocks), nunca só um caminho de erro, e usa o serviço externo quando ele está acessível. Uma task sem nada executável omite a fumaça, e a task que torna aquilo executável a cobre. A validação completa fica no `do-execute-qa`.

**Output:**
- `./prds/prd-[slug]/tasks/tasks.md` (index)
- `./prds/prd-[slug]/tasks/[num]_task.md` (cada task)

**Regra de ouro:** Cada task deve ser funcional, ter seus próprios testes, e representar ~100–200 linhas de mudança de código (para evitar context overflow na execução).

---

### `do-execute-task` — Implementar Task

**Quando usar:** Task específica para implementar (ex: "execute task 3").

**O que faz:**
1. Le PRD, TechSpec, tasks.md e `[num]_task.md`
2. **MCP Discovery**: Descobre quais MCPs estão configurados e aplicável
3. Implementa com testes unit/integration
4. **Teste de fumaça via MCP**: Valida só o fluxo principal de sucesso do que a task entregou (nunca só um caminho de erro), usando as tools dos MCPs disponíveis, conforme a capacidade (`ui-testing`, `api-testing`, `messaging`, `data-store`). O E2E completo fica no `do-execute-qa`. Teste com mock nunca conta como fumaça. Um defeito de outra task (achado pela fumaça ou por um teste que já falhava) é corrigido com a menor mudança possível e registrado no review; só quando a correção exige decisão de escopo, design ou contrato a task sai como `MUDANÇAS SOLICITADAS` e a fila para. Nenhuma task é aprovada com teste falhando
5. Code review automático contra checklist de padrões
6. Marca task como `[x]` em `tasks.md`
7. **Cria review artifact obrigatório**

**Output:**
- Implementação no codebase
- `[num]_task_review.md` (MANDATÓRIO — sem isso, task NÃO está completa)

**Regras críticas:**
- TESTES DEVEM PASSAR — tarefa NUNCA completa com testes failing
- Review file MANDATÓRIO — sem artifact = não conta
- Teste com mock de uma dependência externa valida o contrato completo do que é enviado (a requisição inteira), não só pedaços
- Skills relevantes são lidas antes de implementar; o review registra se foram lidas antes ou depois
- Execução AUTÔNOMA — zero pausas para confirmação do usuário

---

### `do-execute-review` — Code Review Geral (Gate antes do QA)

**Quando usar:** Todas tasks implementadas ([x] em tasks.md), obrigatório antes de ir para QA. É um review **de código**: não sobe serviços nem navegador, e testes exploratórios, responsividade e acessibilidade ficam para o QA.

**O que faz:**
1. Le git diff ou lista manual de files modified durante as tasks
2. Verifica conformidade com TechSpec e padrões de código do projeto
3. Roda test suite completa (unit, integration, typecheck)
4. **Sem E2E**: o review julga o código e roda a suíte do projeto; o E2E é do `do-execute-qa`
5. Classifica issues: CRITICAL / MAJOR / MINOR / POSITIVE / **DESVIO JUSTIFICADO** (divergência técnica válida da TechSpec — não reprova, mas recomenda atualizar a TechSpec)
6. Gera review report com status final

**Loop de Correção:**
- Se status = **REPROVADO** (ou **APROVADO COM RESSALVAS** com itens a corrigir): rodar `do-execute-review-fix` → depois `do-execute-review` novamente
- Se status = **APROVADO**: prossegue para FASE 4 (QA)

**Output:** `./prds/prd-[slug]/review/review-[N]/review-report.md` (uma pasta nova a cada rodada: `review/review-1/`, `review/review-2/`…), com os fix tasks em `review/review-[N]/fixes/`. O relatório inclui os itens ainda pendentes da rodada anterior.

**Status:** APROVADO / APROVADO COM RESSALVAS / REPROVADO

---

### `do-execute-review-fix` — Corrigir Findings do Code Review

**Quando usar:** Após `do-execute-review` retornar REPROVADO ou APROVADO COM RESSALVAS.

**O que faz:**
1. Recebe **um** fix task (`review/review-[N]/fixes/fix-R..md`), **vários** arquivos ou a **pasta** `fixes/` (com `--min=<critico|maior|menor>` para limitar). Com mais de um arquivo, cada fix roda em um subagente isolado, um por vez, dos críticos para os menores; se um falhar, os outros seguem (a fila só para se a suíte ficar vermelha). Você escolhe o lote: o framework não decide sozinho o que corrigir
2. Lê o `review-report.md` da mesma rodada para contexto e implementa a correção
3. Roda suite completa de testes — o fix nunca é concluído com testes failing
4. Atualiza o fix task (`corrigido` / `não-resolvido`) e o finding correspondente no `review-report.md` da rodada

**Output:**
- Correção no codebase
- Fix task e `review-report.md` da rodada atualizados

---

### `do-execute-qa` — Quality Assurance E2E

**Quando usar:** Implementação completa e review aprovado, hora de validação final. É o **único** lugar do framework onde o E2E completo, a acessibilidade e a análise visual acontecem; tasks e review não os repetem.

**O que faz:**
1. Cria checklist baseado nos requisitos do PRD
2. **MCP Discovery**: Identifica MCPs disponíveis e aplica capability guard
3. **E2E Tests via MCP**: Roda flows completos capturando screenshots
4. **Accessibility**: Verifica acessibilidade da interface (navegação por teclado, rótulos, contraste, etc.)
5. **Visual verification**: Screenshots de estados diferentes
6. Documenta bugs encontrados com evidência visual

**Output:**
- `./prds/prd-[slug]/qa/qa-[N]/qa-report.md` (uma pasta nova a cada rodada: `qa/qa-1/`, `qa/qa-2/`…)
- `./prds/prd-[slug]/qa/qa-[N]/bugs/bug-[XX]-....md` (se houver bugs) e `qa/qa-[N]/screenshots/`
- O relatório inclui os bugs ainda abertos da rodada anterior
- Cada requisito leva um de quatro status conforme a evidência: `PASSOU` (visto rodando), `FALHOU`, `VERIFICADO NO CÓDIGO` (só leitura de código, nunca conta como aprovado) ou `NÃO TESTADO` (com o motivo e o que foi tentado). Antes de marcar não testado, o QA tenta o que a MCP de interface oferece (permissões, localização simulada, interceptar rede, rodar script na página)
- Ao terminar, encerra a árvore inteira de processos dos serviços que subiu (wrapper, supervisor e servidor), não só o que segura a porta

**Status:** APROVADO se todos os requisitos passaram e não há bug aberto; APROVADO COM RESSALVAS se só restam bugs de severidade baixa (ou requisitos não testados que você aceitar); REPROVADO se há bug aberto de severidade média ou alta

---

### `do-status` — Progresso do PRD

**Quando usar:** Para verificar o estado atual de um PRD, retomar trabalho após interrupção, ou saber qual é a próxima task.

**O que faz:**
1. Identifica o PRD alvo (pelo slug ou por seleção automática)
2. Lê `tasks.md` e calcula progresso (X/Y tasks, Z%)
3. Identifica a próxima task pendente e verifica dependências
4. Detecta tasks marcadas como `[x]` mas sem review file (incompletas)

**Output:** Relatório de progresso em tela (não gera arquivos)

**Skill somente leitura** — não modifica nenhum arquivo.

---

### `do-execute-qa-fix` — Correção de Bugs

**Quando usar:** Após `do-execute-qa` registrar bugs em `qa/qa-[N]/bugs/`.

**O que faz:**
1. Recebe **um** bug (`qa/qa-[N]/bugs/bug-XX-....md`), **vários** arquivos ou a **pasta** `bugs/` (com `--min=<alta|media|baixa>` para limitar). Com mais de um arquivo, cada bug roda em um subagente isolado, um por vez, dos mais graves para os menos graves; se um falhar, os outros seguem (a fila só para se a suíte ficar vermelha)
2. Analisa a causa raiz e implementa a correção
3. Cria testes de regressão e roda a suíte completa
4. **E2E via MCP**: valida a correção no fluxo, quando há MCP de interface ou serviço
5. Atualiza o arquivo do bug (`corrigido` / `não-resolvido`) na mesma rodada

**Output:**
- Correções no codebase
- Arquivos de bug da rodada atualizados, screenshots em `qa/qa-[N]/screenshots/`

---

## Agents de Orquestração

Além das skills (invocadas explicitamente), o DO Framework inclui **agents** — componentes que expõem comandos dedicados para fluxos de trabalho mais complexos. Os agents são instalados automaticamente pelo `do-setup` nos diretórios corretos de cada ferramenta.

### `do-execute-all-tasks` — Executar Tasks em Sequência

Orquestra a execução sequencial de todas (ou um subconjunto de) tasks de um PRD. A skill `do-execute-all-tasks` roda **no agente primário da sessão** e delega cada task a um **subagente isolado** (`agent-execute-task`) — um subagente novo por task — para garantir contexto totalmente distinto entre execuções e evitar estouro da janela de contexto em filas longas.

> ⚠️ **Não existe agente orquestrador.** Nenhum harness suportado garante que um subagente possa spawnar outro subagente: Claude Code permite até 3 níveis desde a v2.1, mas prompts de permissão e o isolamento ficam mais simples com um nível só; Copilot desabilita por default (`chat.subagents.allowInvocationsFromSubagents: false`); Cursor permite só um nível desde a 2.5 (um neto não lança mais nada); opencode limita via `subagent_depth` (default `1`) e, mesmo elevando o limite, prompts de permissão de sessões-neto podem não aparecer na TUI — a fila trava esperando indefinidamente. Se um install antigo deixou `agent-execute-all-tasks.md` em `.claude/agents/`, `.cursor/agents/`, `.github/agents/` ou `.opencode/agents/`, **apague o arquivo** e rode `/do-setup` novamente.

> **Por que o laço é uma skill e não um comando:** a lógica fica em arquivo lido do disco na hora (a skill e o contrato do subagente), então uma cópia antiga em cache de comando ou de agente nunca impõe regras antigas. Os arquivos instalados pelo `do-setup` são só ponteiros finos.

> **O mesmo worker corrige itens:** o `agent-execute-task` também executa um fix task (`do-execute-review-fix`) ou um bug (`do-execute-qa-fix`), conforme a skill indicada no prompt. É o que permite o modo fila dessas duas skills, sem instalar outro agente.

> Convenção de nomes: skills e commands usam o prefixo `do-`; agents usam o prefixo `agent-` para não colidir com a skill `do-execute-task` que é executada dentro do worker.

**Quando usar:** Ao invés de rodar `/do-execute-task` manualmente task por task, use esta skill para executar um lote de forma autônoma.

**Regras de execução:**
- Uma task por vez — nunca executa em paralelo
- Ordem numérica estrita
- Para imediatamente em caso de falha e reporta o problema
- **Isolamento real de contexto via subagente** (`Agent` no Claude Code, `Task` no Cursor, `subagent` no opencode, `#runSubagent` no Copilot) — não é apenas instrução textual ao modelo

**Disponível nas ferramentas:**

| Ferramenta | O que o `do-setup` instala | Invocação |
|---|---|---|
| **Claude Code** | Agent (`.claude/agents/agent-execute-task.md`). A skill já é invocável; não há arquivo de comando | `/do-execute-all-tasks <tasks.md> [filtro]` |
| **Cursor AI** | Agent (`.cursor/agents/agent-execute-task.md`). Sem arquivo de comando | invocar a skill `do-execute-all-tasks` com `<tasks.md> [filtro]` |
| **GitHub Copilot** | Agent (`.github/agents/agent-execute-task.agent.md`). Sem arquivo de prompt | invocar a skill `do-execute-all-tasks` com `<tasks.md> [filtro]` |
| **Opencode** | Agent (`.opencode/agents/agent-execute-task.md`) + comandos finos em `.opencode/commands/` | `/do-execute-all-tasks <tasks.md> [filtro]` |

**Filtros suportados:**

| Filtro | Exemplo | Comportamento |
|---|---|---|
| `all` (default) | `/do-execute-all-tasks tasks.md all` | Todas as tasks pendentes (não marcadas `[x]`) |
| Range | `/do-execute-all-tasks tasks.md 1.0-4.0` | Tasks no intervalo especificado |
| Lista | `/do-execute-all-tasks tasks.md 1.0,3.0,5.0` | Tasks específicas por ID |

**Instalação:** Feita automaticamente pelo `do-setup`. Para reinstalar apenas os agents sem refazer o setup completo:

```
/do-setup agents
```

---

## MCP Integration — Testes E2E Dinâmicos

O DO Framework usa **Model Context Protocol (MCP)** para descoberta dinâmica de ferramentas de teste. Nenhum servidor, produto ou stack é presumido: o framework classifica cada MCP pelo que as suas tools **fazem**.

### Como Funciona a Descoberta

1. Lê o arquivo de configuração MCP da ferramenta de IA (`.mcp.json`, `.vscode/mcp.json`, etc.) e as tools expostas na sessão → lista os MCP servers disponíveis
2. Lê `do-shared/references/do-mcp-capabilities.md` → classifica cada server em uma **capacidade** (`ui-testing`, `api-testing`, `messaging`, `data-store`, `documentation`) pelos nomes e descrições das suas tools
3. Constrói um mapa interno de capacidades: `{ "<capacidade>": ["<server>"] }`
4. Aplica **capability guard**: usa o MCP disponível conforme o tipo da feature

### Capability Guard — Tabela de Decisão

| Tipo da feature | MCP disponível | Ação |
|-----------------|----------------|------|
| Interface de usuário + `ui-testing` | Sim | Rodar E2E de interface via MCP |
| Serviço + `api-testing`/`messaging`/`data-store` | Sim | Rodar E2E do serviço via MCP correspondente |
| Interface + serviço, com ambos | Sim | Rodar os dois E2Es |
| Feature + nenhum MCP relevante | Não | Pular E2E, documentar lacuna, continuar com unitários/integração |

### Como Configurar MCPs no Seu Projeto

A localização e o formato do arquivo de configuração de MCPs **varia conforme a ferramenta de IA** utilizada:

| Ferramenta | Arquivo de Configuração | Chave principal |
|-----------|------------------------|----------------|
| **Claude Code** | `.mcp.json` (raiz do projeto) ou `~/.mcp.json` (global) | `mcpServers` |
| **GitHub Copilot** | `.vscode/mcp.json` | `servers` |
| **Cursor** | `.cursor/mcp.json` | `mcpServers` |
| **Opencode** | `opencode.json` (raiz do projeto) ou `~/.config/opencode/opencode.json` (global) | `mcp` |

Estrutura mínima de uma entrada (os valores entre `<>` vêm da documentação do MCP que você escolher):

```json
{
  "<chave-principal>": {
    "<nome-do-mcp>": {
      "type": "stdio",
      "command": "<comando>",
      "args": ["<argumentos>"]
    }
  }
}
```

> No **Opencode** a entrada usa `"type": "local"`, `"command": ["<comando>", "<argumentos>"]` (ou `"type": "remote"` com `"url"`) e `"enabled": true`, e os servidores ficam sob a chave `mcp`: direto (`mcp → <nome>`) ou agrupados em `mcp → servers → <nome>`; o schema oficial aceita os dois. Consulte a documentação da sua ferramenta para o formato exato.

### Catálogo de Capacidades

**Arquivo:** `skills/do-shared/references/do-mcp-capabilities.md`

Define, para cada capacidade:
- Descrição e sinais para reconhecê-la nas tools de um server
- Se requer alvo em execução (aplicação, API, broker, armazenamento)
- Tratamento se indisponível

### Adicionar Novo MCP (Zero Code Changes)

1. Configurar no arquivo MCP da sua ferramenta (`.mcp.json`, `.vscode/mcp.json`, etc.)
2. **Pronto** — o servidor é descoberto e classificado automaticamente. Nenhuma edição nas skills nem no catálogo é necessária. Rode `/do-setup refresh` para atualizar o resumo gravado em `.do/context.md`.

## Prerequisitos do Ambiente

O framework não exige nenhuma linguagem ou stack. O que você precisa:

- A **ferramenta de IA** escolhida (Claude Code, GitHub Copilot, Cursor ou Opencode).
- **Git** (para versionamento e análise de diff).
- Os **pré-requisitos do seu próprio projeto** (compilador/interpretador, gerenciador de dependências, runner de testes). O `do-setup` registra os comandos do projeto na seção `Project Commands` de `.do/context.md` e as skills de execução os usam.
- Os **pré-requisitos dos MCPs** que você optar por configurar (cada MCP documenta o que precisa).

## Estrutura de Pastas Gerada

```
./.do/                          # local a cada desenvolvedor, ignorado pelo git
├── context.md                  # contexto do framework (do-setup): resumo, convenções, comandos, skills, MCPs
└── tmp/                        # rascunhos das skills de execução, apagados ao fim de cada execução

./CLAUDE.md ou ./AGENTS.md      # só se o projeto não tinha nenhum arquivo de instruções e você aceitou criar (do-setup);
                                # em inglês, sem referências ao framework, não adicionado ao git

./prds/
└── prd-[feature-slug]/
    ├── prd.md                  # Product Requirements Document (do-create-prd)
    ├── techspec.md             # Tech Spec (do-create-techspec)
    ├── qa/
    │   └── qa-1/               # 1ª rodada de QA (do-execute-qa); a próxima será qa-2/...
    │       ├── qa-report.md
    │       ├── bugs/bug-01-alta-....md
    │       └── screenshots/
    ├── review/
    │   └── review-1/           # 1ª rodada de review (do-execute-review); a próxima será review-2/...
    │       ├── review-report.md
    │       └── fixes/fix-R-01-critico-....md
    ├── review-report.md        # (legado) relatório solto na raiz, de PRDs anteriores às pastas de rodada
    ├── qa-report.md            # (legado) idem para o QA
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
| **Slug kebab-case** | Pastas usam kebab-case: `prd-projeto-exemplo`, `prd-user-auth` |
| **Numeração de tasks** | Tasks numeradas sequencialmente: `1_task.md`, `2_task.md` |
| **Artefatos obrigatórios** | Sem artifact = task incompleta (review files, reports) |
| **Testes devem passar** | Task NUNCA completa com failing tests |
| **Execução autônoma** | Skills executam sem pausas para confirmação (exceto steps interativos explícitos) |
| **Língua dos artifacts** | PT-BR para documentos, English para code/tech terms |
| **Pastas de rodada** | Review e QA gravam em `review/review-N/` e `qa/qa-N/` dentro do PRD; nada sobrescreve uma rodada anterior |
| **Raiz do projeto é o limite** | Rascunhos só em `.do/tmp/` da raiz do projeto, por caminho absoluto quando o comando roda num subdiretório (apagado ao fim); saída de comando é filtrada no pipe, nunca redirecionada para fora do projeto; serviços que a execução subiu são encerrados ao final, a árvore inteira, pelo handle ou PID guardado; só se apaga o que a própria execução criou |
| **Estado do git é do desenvolvedor** | Nenhuma skill de execução altera o git (stage, commit, reset, checkout, stash, branch); só lê `status`, `diff`, `log` e `show` |
| **Fila de correções** | Com pasta ou vários arquivos, `review-fix` e `qa-fix` rodam um subagente por item; a falha de um item não derruba os outros. Um fix só altera código e testes: se exigir mudar PRD/TechSpec ou uma decisão do usuário, o worker devolve `NÃO RESOLVIDO` sem editar nada, e a decisão aparece no resumo |

## Quando Usar Cada Skill — Guia Rápido

> Os exemplos abaixo usam a sintaxe do **Claude Code** (`/skill-name`). Consulte a documentação da sua ferramenta para a forma de invocar skills (GitHub Copilot, Cursor, etc. possuem sua própria sintaxe de invocação).

| Cenário | Skill | Loop? |
|---------|-------|-------|
| Novo projeto, primeira vez | `do-setup` | Não (1x) |
| Nova feature (ideia vaga) | `do-create-prd` | Não |
| PRD criado, definir arquitetura | `do-create-techspec` | Não |
| TechSpec pronta, criar tasks | `do-create-tasks` | Não |
| Verificar progresso / retomar trabalho | `do-status` | Não |
| Implementar task específica | `do-execute-task 1` | Sim (por cada task) |
| **Review geral (OBRIGATÓRIO antes do QA)** | `do-execute-review` | **Sim (até APROVADO)** |
| Corrigir findings do review | `do-execute-review-fix` | **Sim (com do-execute-review)** |
| QA E2E da feature completa | `do-execute-qa` | Sim (com `do-execute-qa-fix`) |
| Corrigir bugs encontrados no QA | `do-execute-qa-fix` | **Sim (até zero HIGH)** |

## Fluxo Completo — Exemplo Prático

**Feature:** "Modo escuro no app"

> Os comandos abaixo usam a sintaxe do **Claude Code** (prefixo `/`). Para outras ferramentas, adapte a invocação conforme necessário.

```bash
# FASE 1: PLANEJAMENTO
# ------------------------------------------------
# 1. Criar PRD
/do-create-prd *descrição ou arquivo de contexto*
> Feature: Projeto Exemplo
> [responde perguntas sobre problema, usuários, flows]

# 2. Criar TechSpec
/do-create-techspec projeto-exemplo
> [pesquisa documentação (MCP, se disponível) + responde clarifications técnicas]

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

# FASE 3: CODE REVIEW GERAL (LOOP até APROVADO)
# ------------------------------------------------
# 5. Review geral do PRD (OBRIGATÓRIO antes do QA)
/do-execute-review projeto-exemplo
> Status: REPROVADO / COM RESSALVAS? → /do-execute-review-fix prds/prd-projeto-exemplo/review/review-1/fixes/ --min=maior → /do-execute-review projeto-exemplo
> Status: APROVADO? → prossegue para QA

# FASE 4: VALIDAÇÃO E2E + BUGFIX (LOOP até zero bugs HIGH)
# ------------------------------------------------
# 6. QA final com E2E completo
/do-execute-qa projeto-exemplo
> [roda E2E via MCP disponível, accessibility, visual verification]
> [gera qa/qa-1/qa-report.md + qa/qa-1/bugs/ se houver problemas]

# 7. Se houver bugs HIGH/MEDIUM: loop de correção
[se qa/qa-1/bugs/ tem bugs abertos] → /do-execute-qa-fix prds/prd-projeto-exemplo/qa/qa-1/bugs/<bug>.md
> [corrige por severidade + tests de regressão]

# 8. Revalida com QA novamente
/do-execute-qa projeto-exemplo
> [reteste apenas areas corrigidas ou E2E completo]

# 9. Repetir steps 7-8 até zero bugs HIGH
[se ainda tem bugs HIGH] → /do-execute-qa-fix projeto-exemplo → /do-execute-qa projeto-exemplo
[zero bugs HIGH] → FEATURE READY! ✅
```

## Referências

| Arquivo | Propósito |
|---------|-----------|
| `do-shared/references/do-mcp-capabilities.md` | Registry central de MCPs e suas capacidades |
| `do-shared/references/do-mcp-discovery-instructions.md` | Procedimento de descoberta dinâmica de MCPs (nunca lê o arquivo de configuração inteiro) |
| `do-shared/references/do-project-context.md` | Como usar as skills e MCPs instalados no projeto |
| `do-shared/references/do-project-commands.md` | Como obter os comandos do próprio projeto (testes, cobertura, checagem estática, serviço) |
| `do-shared/references/do-service-readiness.md` | Checar, reaproveitar, iniciar e encerrar os serviços necessários |
| `do-shared/references/do-run-folders.md` | Pastas de rodada `review/review-N` e `qa/qa-N`, pendências da rodada anterior e layout legado |
| `do-shared/references/do-fix-queue.md` | Modo fila de `review-fix` e `qa-fix` (um subagente por item) |
| Arquivo de configuração MCP (`.mcp.json`, `.vscode/mcp.json`, etc.) | Configuração de MCP servers ativos no projeto — localização varia por ferramenta de IA |
| `.do/context.md` | Contexto do framework gerado por `do-setup` (local, ignorado pelo git) |
| Arquivo de instruções do projeto (`CLAUDE.md`, `AGENTS.md`, etc.) | Lido pelo framework para as convenções e nunca alterado; criado pelo `do-setup` só se não existir nenhum e você aceitar, sem referências ao framework — nome varia por ferramenta de IA |

## Compatibilidade com Ferramentas de IA

O DO Framework é agnóstico à ferramenta de IA. Os conceitos, o fluxo de trabalho e os artefatos gerados funcionam com qualquer ferramenta que suporte skills/instruções customizadas.

> **Ferramentas testadas:** As skills e agents do DO Framework foram testados com **Opencode**, **Claude Code**, **Cursor** e **GitHub Copilot**. Por serem implementados como skills (instruções em arquivos de texto), possivelmente funcionam com outras ferramentas que suportem o mesmo mecanismo — mas não há garantia de compatibilidade.

**Convenções de caminho**: Os arquivos de skill referenciam internamente o diretório `.claude/skills/` (convenção do Claude Code). Se você usa outra ferramenta, os arquivos estarão no diretório equivalente dessa ferramenta (ex: `.github/` para GitHub Copilot).

| Ferramenta | Diretório de Skills | Instruções do Projeto (só leitura) | Criado se não houver nenhum | Config MCP |
|-----------|--------------------|--------------------|--------------------|------------|
| **Claude Code** | `.claude/skills/` | `CLAUDE.md` | `CLAUDE.md` | `.mcp.json` |
| **GitHub Copilot** | `.github/` (instructions) | `.github/copilot-instructions.md` ou `AGENTS.md` | `AGENTS.md` | `.vscode/mcp.json` |
| **Cursor** | `.agents/skills/` ou `.cursor/skills/` (regras em `.cursor/rules/`) | `.cursor/rules/*.mdc`, `.cursorrules` ou `AGENTS.md` | `AGENTS.md` | `.cursor/mcp.json` |
| **Opencode** | `.agents/skills/` | `AGENTS.md` | `AGENTS.md` | `opencode.json` (chave `mcp`) |

---

## Dicas Importantes

1. **Não pule steps**: PRD → TechSpec → Tasks → Execute (ordem é importante)
2. **MCPs são opcionais mas recomendados**: Sem MCP, perde-se o E2E automatizado do QA e os testes de fumaça das tasks
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

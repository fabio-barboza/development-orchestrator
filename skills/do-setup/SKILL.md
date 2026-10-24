---
name: do-setup
description: "Initializes the project: analyzes it, inventories installed skills and MCP servers (reporting gaps), and writes the project configuration file (CLAUDE.md, AGENTS.md, .github/copilot-instructions.md or .cursor rules) with summary, conventions, commands, skills and MCPs. Installs the orchestration agents and commands. Arguments: \"agents\" (install agents/commands only), \"refresh\" (re-detect skills/MCPs only). Use to initialize or configure the agent-assisted environment. Not for PRDs, implementation, review or QA."
---

# Project Setup

## Role
You are a senior developer advocate responsible for project initialization, tooling configuration, and ensuring the agent-assisted development environment is correctly set up.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reasons to stop and ask the user are:
1. **The mandatory initial AI tool selection question (Step 0).** This is the FIRST action the skill performs and is always required.
2. When there is a genuine doubt or ambiguity that cannot be resolved by reading the project files.

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to analyze the project structure and produce the configuration document. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

## Procedures

**Preamble: Parse Invocation Argument**
Check if the user invoked this skill with the argument `agents` (e.g., `/do-setup agents`).

- If the argument **is** `agents`: set mode = `agents-only`. Skip Steps 1–4 and 6. Execute only Step 0 (AI tool selection) and Step 5 (install agents).
- If the argument **is** `refresh`: set mode = `refresh`. Execute only Step 0 (AI tool selection), Step 3 (discover skills), Step 3B (inventory MCPs), Step 4 restricted to the `## Available Skills`, `## Uncovered Skills` and `## Available MCPs` sections, and Step 6. Skip Steps 1, 2 and 5 — except that, for **GitHub Copilot**, you must also re-sync the `<server-name>/*` MCP entries of the `tools:` list in `.github/agents/agent-execute-task.agent.md` with the MCPs found in Step 3B (add the new ones, keep the existing ones), since Copilot only exposes listed tools to the worker. The project configuration file must already exist; if it does not, stop and tell the user to run `/do-setup` (full) first. In this mode derive the project technologies from the existing `## Project Summary` section instead of re-analysing the project.
- If no argument or any other argument: set mode = `full`. Execute all steps in order.

**Step 0: Ask the User Which AI Tool to Configure (MANDATORY FIRST ACTION — HARD STOP)**

> ⛔ **HARD STOP — DO NOT SKIP THIS QUESTION UNDER ANY CIRCUMSTANCE.**
>
> This question is the **very first action** of the skill. It is **MANDATORY** even when:
> - you (the model) are running inside OpenCode, Claude Code, Cursor, or GitHub Copilot — the host you are running in is **IRRELEVANT** to the answer; the user may want to configure THIS project for a DIFFERENT tool;
> - the project already contains `.claude/`, `.opencode/`, `AGENTS.md`, `.cursor/`, `.github/`, `opencode.json`, `CLAUDE.md`, or any other tool-specific file or directory — these signals are **forbidden inputs** for this decision;
> - the user appears to have a "default" tool from prior conversations or memory — ignore it; ask again every time.
>
> ❌ **DO NOT** auto-detect, infer, guess, default, pre-fill, or assume the answer.
> ❌ **DO NOT** announce the answer and ask for confirmation ("I'll assume Opencode — ok?"). That is still skipping the question.
> ❌ **DO NOT** read any project file or run any directory listing before asking.
> ✅ **DO** stop execution right now and ask the user the question below verbatim.

### The question to ask (verbatim)

Present these four options to the user, exactly:

> **Para qual ferramenta agêntica o projeto deve ser configurado?**
>
> 1. **Claude Code** — gera `CLAUDE.md`, usa `.claude/skills/`, instala agentes em `.claude/agents/` e comandos em `.claude/commands/`.
> 2. **GitHub Copilot** — gera `.github/copilot-instructions.md`, usa `.github/` para instruções, instala agentes em `.github/agents/` e prompts em `.github/prompts/`.
> 3. **Cursor AI** — gera `.cursor/rules/project.mdc` (ou `.cursorrules` legado), usa `.agents/skills/` e `.cursor/skills/` (além das regras em `.cursor/rules/`), instala agentes em `.cursor/agents/` e comandos em `.cursor/commands/`.
> 4. **Opencode** — gera `AGENTS.md`, usa `.agents/skills/`, instala agentes em `.opencode/agents/` e comandos em `.opencode/commands/`.

### How to ask

- If the host provides a structured question tool (e.g., `AskUserQuestion` in Claude Code), use it with these four options.
- Otherwise, emit the four numbered options as plain text in your reply and **end the turn immediately**. Do not call any other tool, do not read files, do not write anything. Yield control to the user and wait for their reply.
- After yielding, **the next user message is the answer**. Accept the option number (1–4) or the tool name. If the answer is ambiguous, ask again — never guess.

### Acceptance criteria (you must satisfy ALL before proceeding to Step 1)

- [ ] The four options were presented to the user in this turn or a previous turn of the current invocation.
- [ ] The user has explicitly replied with one of: `1`, `2`, `3`, `4`, `Claude Code`, `GitHub Copilot`, `Cursor AI`, or `Opencode`.
- [ ] You have NOT read any project file, listed any directory, or run any Bash command yet.

If any criterion is unmet, **return to the top of Step 0** and ask again.

### Resolution table

Once the user answers, store these values internally for use in every subsequent step:

| Option | AI Tool | Config file | Skills directory | Agents directory | Commands directory |
|--------|---------|-------------|------------------|------------------|--------------------|
| 1 | Claude Code | `CLAUDE.md` | `.claude/skills/` | `.claude/agents/` | `.claude/commands/` |
| 2 | GitHub Copilot | `.github/copilot-instructions.md` | `.github/` | `.github/agents/` | `.github/prompts/` |
| 3 | Cursor AI | `.cursor/rules/project.mdc` (or `.cursorrules` if the user explicitly asks for the legacy format) | `.agents/skills/`, `.cursor/skills/` | `.cursor/agents/` | `.cursor/commands/` |
| 4 | Opencode | `AGENTS.md` | `.agents/skills/` | `.opencode/agents/` | `.opencode/commands/` |

Only after the user has explicitly selected one option, continue to Step 1 using the selected tool's paths and conventions consistently.

**Step 1: Initialize Project Configuration**
The initialization strategy depends on the AI tool selected by the user in Step 0:

- **Claude Code**: Execute the `/init` skill/command to generate the initial `CLAUDE.md`. Wait for it to complete before proceeding.
- **Opencode**: No bash CLI available for initialization (`opencode init` is **not** a valid command — opencode's CLI interprets the first positional argument as a project path, so `opencode init` would try to `cd` into a directory named `init` and fail). Create `AGENTS.md` at the project root directly using the `Write` tool if it doesn't exist. Do **not** invoke `opencode init` via Bash under any circumstances.
- **GitHub Copilot**: No built-in init command. Create `.github/copilot-instructions.md` if it doesn't exist.
- **Cursor AI**: No built-in init command. Create the config file as determined in Step 0:
  - If using `.cursor/rules/project.mdc`: create the `.cursor/rules/` directory if needed, then create the file with the following frontmatter header:
    ```
    ---
    description: Project instructions and conventions for the development orchestrator
    globs:
    alwaysApply: true
    ---

    ```
  - If using `.cursorrules` (legacy): create the file at the project root.

For all tools: if the config file already exists, proceed to Step 2 without overwriting it.

**Counterpart file check (warn only — never modify the other file):** if the tool selected is **Opencode** and a `CLAUDE.md` exists, or the tool selected is **Claude Code** and an `AGENTS.md` exists, remember it for the Step 6 report. Do not merge, convert or edit the other file.

**Step 2: Deep Project Analysis**
1. Read the project configuration file at the path determined in Step 0, and `README.md` if it exists.
2. List the project root and read the root-level files that declare dependencies, build, test and run commands, versions or infrastructure (manifests, build descriptors, task runners, container/orchestration files, version pins, compiler/type configuration) — whatever the project uses. Do not assume a particular language or tool: identify them from what is actually there.
3. Scan directory structure recursively, ignoring:
   - What the project itself excludes (read its version-control ignore file): dependency directories, build outputs, caches and generated artifacts
   - Compiled and packaged artifacts, and binaries/media (images, documents, archives)
   - Hidden: any path starting with `.` (except `.claude/`, `.github/`, and `.cursor/`)
4. Read representative files from each layer or module (entry points, business logic, data access, tests) to understand adopted patterns.
5. Build an internal summary with:
   - Main stack and versions
   - Adopted architecture and design patterns
   - Naming and organization patterns
   - System purpose
   - External integrations (queues, databases, APIs)
6. Check test infrastructure:
   - Look for how the project declares its test command (see `do-shared/references/do-project-commands.md`).
   - Scan for test files and directories following the project's own naming conventions.
7. Extract the project commands (install, run all tests, run one test, static checks, start the service locally and its port/health address) from what the project itself declares, following `do-shared/references/do-project-commands.md`. Record only commands that exist; use `—` for the rest. They are written to `## Project Commands` in Step 4.
   - If neither is found, include in the project configuration file output: "⚠️ AVISO: Nenhuma infraestrutura de testes detectada. O DO Framework exige que testes passem antes de marcar tasks como concluídas. Configure um test runner antes de usar `do-execute-task`."

**Step 3: Discover Installed Skills**
**No skill other than the `do-*` ones is assumed to exist.** Zero skills found is a valid result, not an error.
1. Build the list of installed skills from both sources and merge them (de-duplicate by skill name; a project-level skill wins over a global one):
   a. **Native list:** the skills your AI tool already exposes in the session (for example the list shown by the `skill` tool). Use name, description and location when provided.
   b. **Directory scan:** every directory below that exists, each skill being a subfolder containing a `SKILL.md`:
      - Project: `.agents/skills/`, `.claude/skills/`, `.cursor/skills/`, `.opencode/skills/`, `.github/skills/`
      - Global (home directory): `~/.agents/skills/`, `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.cursor/skills/`
      - Extra for **GitHub Copilot**: instruction files in `.github/`. Extra for **Cursor AI**: all `.mdc` files in `.cursor/rules/`.
   Ignore folders without a `SKILL.md`.
2. **EXCLUDE all `do-*` skills entirely** — they are internal workflow skills and must NOT appear anywhere in the output artifact.
3. For each remaining skill, read its descriptor (`SKILL.md`, or the `.mdc` content for Cursor rules).
4. Based on the Step 2 summary, keep the skills that are relevant: they cover at least one of the project's primary language or framework, the adopted architecture, or an identified pattern or integration (queues, database, API, tests, UI, etc.).
5. For each kept skill record: name, **path** (project-relative when inside the project, absolute when global — confirm the file exists before writing it) and "when to use".
6. Technologies identified in Step 2 with no matching skill go to `## Uncovered Skills`.

**Step 3B: Inventory MCP Servers (read-only)**
Skip in `agents-only` mode. This step only **reads** configuration — never start, call or test an MCP server, and never make network calls.
1. Follow the discovery procedure in `do-shared/references/do-mcp-discovery-instructions.md` (project MCP config file for the selected tool, global config, and the tools exposed in the session) and map each server with `do-shared/references/do-mcp-capabilities.md`. Both live in the skills directory, next to `do-setup` (the project-local `<skill-dir>` parent from Step 5): read them by full path, since glob tools skip hidden directories such as `.agents/`.
2. For each MCP found record: name, source (project config, global config or session tools), type (local/remote) and capability. A server absent from the registry is listed as `não registrado`. **Never copy tokens, headers, URLs with credentials or environment values** from the config into any output.
3. Compute informational **gaps** from the Step 2 summary (never errors):
   - project has a user interface and no `ui-testing` capability → UI E2E will be skipped by `do-execute-task`/`do-execute-qa`;
   - project integrates with a service (messaging, data store, API) and no matching capability (`messaging`, `data-store`, `api-testing`) → service E2E will be skipped;
   - no `documentation` capability → library docs will come from Web Search or the codebase.
4. Zero MCPs found is a valid result.

**Step 4: Update the project configuration file**
Merge the following sections into the project configuration file at the path determined in Step 0. Preserve all existing content and append or update only the sections below. In `refresh` mode update only `## Available Skills`, `## Uncovered Skills` and `## Available MCPs`.

> **Instrução ao agente (não incluir no arquivo gerado):** Se o Step 3 não encontrou nenhuma skill de tecnologia, a tabela de `## Available Skills` recebe uma única linha `_(nenhuma)_ | — | Nenhuma skill de tecnologia detectada` e a linha "Exemplo:" é omitida. Se o Step 3B não encontrou MCPs, a tabela de `## Available MCPs` recebe uma única linha `_(nenhum)_ | — | Nenhum MCP detectado`. Nunca escreva tokens, headers ou valores de ambiente. Na seção `## Available Skills`, escreva SOMENTE o parágrafo IMPORTANTE, o exemplo, a observação e a tabela. Nunca inclua skills `do-*` (do-setup, do-create-prd, do-create-techspec, do-create-tasks, do-execute-task, do-execute-review, do-execute-qa, etc.) na tabela. O parágrafo IMPORTANTE é **obrigatório** — não omita, não parafraseie.

```markdown
## Project Summary
- **Purpose:** [system description]
- **Stack:** [main technologies and versions]
- **Architecture:** [adopted pattern]
- **Integrations:** [external services]

## Available Skills
**IMPORTANTE: Antes de iniciar qualquer tarefa, leia o arquivo da skill correspondente à tecnologia ou padrão envolvido usando a ferramenta de leitura de arquivos disponível na sua plataforma. As skills contêm convenções, restrições e padrões obrigatórios para este projeto — não prossiga sem lê-las.**

Exemplo: para trabalhar com [primeira-tecnologia], leia o arquivo `[caminho-da-primeira-skill]` antes de qualquer implementação.

> Lista gravada pelo `/do-setup`. Skills instaladas depois podem não constar aqui: consulte também a lista de skills da sua ferramenta ou rode `/do-setup refresh`.

| Skill | Caminho | Quando usar |
|-------|---------|-------------|
| [name] | [path found in Step 3] | [usage context] |

## Uncovered Skills
| Tecnologia | Observação |
|------------|------------|
| [technology identified in Step 2] | Nenhuma skill disponível localmente — adicione manualmente ao diretório de skills |

## Available MCPs
> Resumo gravado pelo `/do-setup`; a descoberta em tempo de execução (`do-shared/references/do-mcp-discovery-instructions.md`) prevalece. Para atualizar após instalar MCPs, rode `/do-setup refresh`.

| MCP | Capacidade | Quando usar |
|-----|------------|-------------|
| [name] | [capability from the registry, or "não registrado"] | [usage context] |

## Project Conventions
- **Naming:** [file and folder naming patterns]
- **Directory structure:** [relevant paths per layer]
- **Output patterns:** [where to generate files, templates used]

## Project Commands
| Finalidade | Comando | Observação |
|------------|---------|------------|
| Instalar dependências | [command or —] | [prerequisites] |
| Rodar todos os testes | [command or —] | [scope] |
| Rodar um teste isolado | [command with placeholder or —] | [how to filter] |
| Checagem estática | [command(s) or —] | [type checking, lint, formatting — whatever the project defines] |
| Iniciar o serviço local | [command or —] | [port or health address] |
```

**Step 5: Install Orchestration Agents & Commands**
Based on the AI tool selected by the user in Step 0, install the `agent-execute-task` worker subagent and the slash command for orchestration. The orchestration logic of the queue lives in different places depending on the AI tool's subagent nesting policy:

**In every supported AI tool the orchestration loop lives inside the slash command/prompt itself** (running on the main/primary agent), and only `agent-execute-task` is installed as a subagent. Each task is delegated via the platform's task tool — exactly **one** nesting level, which every tool allows.

- **Claude Code**: since v2.1 subagents can spawn subagents (up to three levels, `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`; [docs](https://code.claude.com/docs/en/sub-agents)), but the queue still runs on the main agent so each task stays one level deep and permission prompts surface in the main session. The delegation tool is `Agent` (renamed from `Task` in v2.1.63; `Task` remains an alias). `run_in_background` is offered only in some modes (present in `claude -p`, absent in interactive sessions with fork mode on): subagents may start in the background and report back through a completion notification, so the orchestrator must wait for it before the next task. `agent-execute-task` blocks delegation with `disallowedTools: Agent` instead of a `tools` allowlist, so it keeps MCP tools (needed for E2E tests) and preloads the `do-execute-task` skill via `skills`.
- **Cursor, GitHub Copilot**: subagents **cannot** spawn other subagents by default (Copilot requires `chat.subagents.allowInvocationsFromSubagents=true`, off by default; Cursor allows one nesting level since 2.5, but a grandchild cannot launch further subagents).
- **Opencode**: nesting is capped by `subagent_depth` (**default `1`**), and a subagent only receives the `subagent` tool (named `task` in older releases) if it declares `permission.task`. Do not mark `agent-execute-task` as `hidden: true`: since opencode 2.1, hidden agents are omitted from the `subagent` tool's "Available subagents" list, so the orchestrator cannot see the worker. Even with `subagent_depth: 2`, permission prompts raised inside a grandchild session may never render in the TUI — the queue then waits forever. So no `agent-execute-all-tasks` orchestrator is installed here either.

> Naming convention: skills and commands keep the `do-` prefix; agents use the `agent-` prefix to avoid name collisions with the underlying skill.

1. Resolve `<skill-dir>` **inside the current project only**. The templates were copied alongside the `SKILL.md` when the DO Framework skills were installed in this project.
   - Check these paths, in order, relative to the project root, and use the first that exists: `.agents/skills/do-setup`, `.claude/skills/do-setup`. If neither exists, use the `do-setup` directory that the installation created **inside the project root**.
   - **NEVER** use a `do-setup` copy outside the project root (parent directories such as `~/projects/.agents/`, global/home skill directories, other projects). Those copies can be stale and install an obsolete orchestrator. Do not run a recursive Glob from a parent directory to find it.
   - Sanity check: `<skill-dir>/agents/<tool>/agents/` must contain `agent-execute-task.md` (`agent-execute-task.agent.md` for GitHub Copilot) and must **not** contain any `*execute-all-tasks*` file. If the check fails, the copy is stale: **STOP** and tell the user to reinstall the DO Framework skills in this project (see the installation instructions in the README).
2. Execute **only** the installation block that matches the AI tool selected by the user in Step 0. Do not install assets for other tools.

   **Claude Code** (if the user selected option 1):
   - Run `mkdir -p .claude/agents .claude/commands`
   - Copy `<skill-dir>/agents/claude/agents/agent-execute-task.md` → `.claude/agents/agent-execute-task.md`
   - Copy `<skill-dir>/agents/claude/commands/do-execute-all-tasks.md` → `.claude/commands/do-execute-all-tasks.md`
   - **Do NOT** copy any `agent-execute-all-tasks.md` — orchestration is embedded in the slash command, so each task stays one subagent level deep.
   - Delete any orchestrator agent left by previous installs (all legacy names): `rm -f .claude/agents/agent-execute-all-tasks.md .claude/agents/do-execute-all-tasks.md .claude/agents/execute-all-tasks.md`.

   **Cursor AI** (if the user selected option 3):
   - Run `mkdir -p .cursor/agents .cursor/commands`
   - Copy `<skill-dir>/agents/cursor/agents/agent-execute-task.md` → `.cursor/agents/agent-execute-task.md`
   - Copy `<skill-dir>/agents/cursor/commands/do-execute-all-tasks.md` → `.cursor/commands/do-execute-all-tasks.md`
   - **Do NOT** copy any `agent-execute-all-tasks.md` — orchestration is embedded in the slash command.
   - Delete any orchestrator agent left by previous installs (all legacy names): `rm -f .cursor/agents/agent-execute-all-tasks.md .cursor/agents/do-execute-all-tasks.md .cursor/agents/execute-all-tasks.md`.

   **GitHub Copilot** (if the user selected option 2):
   - Run `mkdir -p .github/agents .github/prompts`
   - Copy `<skill-dir>/agents/github/agents/agent-execute-task.agent.md` → `.github/agents/agent-execute-task.agent.md`
   - Copy `<skill-dir>/agents/github/prompts/do-execute-all-tasks.prompt.md` → `.github/prompts/do-execute-all-tasks.prompt.md`
   - **MCP tools for the worker:** the template's `tools:` list has no MCP entries (GitHub Copilot only exposes to a custom agent the tools it lists). After copying, append one `  - <server-name>/*` line to the `tools:` list of `.github/agents/agent-execute-task.agent.md` for **every MCP server** found by the Step 3B discovery (in `agents-only` mode run just the read-only discovery of Step 3B, item 1, for this purpose), using the server names as written in `.vscode/mcp.json`. If no MCP server is found, append nothing. This keeps Copilot equivalent to the other tools, where the worker sees whatever MCPs the project has.
   - **Do NOT** copy any `agent-execute-all-tasks.agent.md` — orchestration is embedded in the prompt.
   - Delete any orchestrator agent left by previous installs (all legacy names): `rm -f .github/agents/agent-execute-all-tasks.agent.md .github/agents/do-execute-all-tasks.agent.md .github/agents/execute-all-tasks.agent.md`.

   **Opencode** (if the user selected option 4):
   - Run `mkdir -p .opencode/agents .opencode/commands`
   - Copy `<skill-dir>/agents/opencode/agents/agent-execute-task.md` → `.opencode/agents/agent-execute-task.md`
   - Copy all `<skill-dir>/agents/opencode/commands/*.md` → `.opencode/commands/`
   - **Do NOT** copy any `agent-execute-all-tasks.md` — orchestration is embedded in the `/do-execute-all-tasks` command and must run in the **primary** session. Um command com `subtask: true` delegando a um orquestrador subagente cria dois níveis de aninhamento: falha com `Subagent depth limit reached` no default (`subagent_depth: 1`) e, mesmo com `subagent_depth: 2`, prompts de permissão do neto podem não aparecer na TUI e a fila trava indefinidamente.
   - Delete any orchestrator agent left by previous installs (all legacy names): `rm -f .opencode/agents/agent-execute-all-tasks.md .opencode/agents/do-execute-all-tasks.md .opencode/agents/execute-all-tasks.md`.

3. Verify the installation before reporting:
   - The agents directory of the selected tool contains `agent-execute-task` and **no** file matching `*execute-all-tasks*`.
   - The installed `do-execute-all-tasks` command/prompt is byte-identical to the template in `<skill-dir>` (compare with `cmp`). For Opencode, it must **not** contain `subtask: true` nor an `agent:` field in the frontmatter — that would route the queue to a subagent orchestrator.
   - If any check fails, fix it (re-copy / delete) and verify again. Never report success with a legacy orchestrator installed.
4. Confirm to the user which files were installed, for which tool, and the `<skill-dir>` used as source.
5. **Tell the user how to load what was just installed, by tool, before running `/do-execute-all-tasks`** (or any newly installed command): **Opencode** — run `/reload` (if commands look outdated, e.g. a project re-cloned at the same path, `opencode service restart`); **Claude Code** — skills are picked up live (use `/reload-skills` only if `.claude/skills/` was just created), but a `.claude/agents/` directory created during this setup is only detected after restarting the session; **Cursor** and **GitHub Copilot** — the documentation does not say, so tell the user to reload the IDE window or open a new chat session if the commands/agents do not appear. Include this notice in the final report of Step 6.

> Use Bash to run `cp` commands. `<skill-dir>` is the project-local `do-setup` directory resolved in item 1 (e.g. `.agents/skills/do-setup`).

**Step 6: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the project configuration file is updated, use the `TaskUpdate` tool to mark all corresponding items in your internal task tracking as `completed`.
2. **ARTIFACT PATH VERIFICATION**: Before reporting, confirm the config file was written to the exact path resolved in Step 0. Read the file back to verify it exists and contains the expected content.
3. Provide a summary of the setup performed. It MUST include this inventory (omit empty lines only if the corresponding list is empty, and say so):
   - **Skills detectadas:** name and path of each relevant technology skill, or "nenhuma skill de tecnologia detectada".
   - **MCPs detectados:** name, capability and source of each server (mark `não registrado` when applicable), or "nenhum MCP detectado".
   - **Lacunas:** the informational gaps computed in Step 3B (e.g. "sem MCP de interface: o E2E de interface será pulado") and, when no skills/MCPs were found, a hint to install them and run `/do-setup refresh`.
   - **Arquivo de configuração duplicado (apenas se o Counterpart file check encontrou):** "Existem `CLAUDE.md` e `AGENTS.md`. O Claude Code lê só o `CLAUDE.md` quando os dois existem e o Opencode lê o `AGENTS.md`; as seções geradas aqui só chegam à ferramenta escolhida. Para unificar, mantenha o conteúdo no `AGENTS.md` e deixe no `CLAUDE.md` a linha `@AGENTS.md`." Do not change the other file.
   End with the load notice from Step 5 item 5 (how to load the installed agents/commands in the selected tool) — except in `refresh` mode, where no agents were installed and the notice is omitted.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the project configuration file saved at the correct path (resolved in Step 0)?
    - Did you accurately identify the project stack and skills?
    - Leia a seção `## Available Skills` no arquivo gerado e confirme que o parágrafo `**IMPORTANTE:**` está presente antes da tabela. Se não estiver, insira-o agora antes de responder ao usuário.
    - Confirme que a seção `## Available MCPs` existe no arquivo gerado e que nenhum token, header ou valor de ambiente foi copiado para ele.

## Output Language
Todos os artefatos gerados (seções do arquivo de configuração do projeto, resumos) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If the project root has no files declaring dependencies, build or run commands, warn the user that the project may not be initialized and ask for clarification about the stack.
- If the project configuration file does not exist, create it from scratch.
- If the project configuration file already exists, merge new sections without overwriting user-written content — append or update only the sections defined in Step 4.
- If no skills (other than `do-*`) or no MCP servers are found, this is not an error: write the empty-state rows, report it in the inventory and suggest installing them and running `/do-setup refresh`.
- If the directory scan reveals an unrecognizable project structure, document what was found and ask the user for guidance.

## References
- Output: Project configuration file (e.g., `CLAUDE.md` for Claude Code, `AGENTS.md` for Opencode, `.github/copilot-instructions.md` for GitHub Copilot, `.cursor/rules/project.mdc` or `.cursorrules` for Cursor AI)
- Skills discovery: the tool's native skill list plus the directories scanned in Step 3. Conventional locations by tool:
  - Claude Code: `.claude/skills/` (each skill has a `SKILL.md`)
  - Opencode: `.agents/skills/` (each skill has a `SKILL.md`)
  - GitHub Copilot: `.github/` (instruction files)
  - Cursor AI: `.agents/skills/` and `.cursor/skills/` (each skill has a `SKILL.md`); rules in `.cursor/rules/` (`.mdc` files with frontmatter)

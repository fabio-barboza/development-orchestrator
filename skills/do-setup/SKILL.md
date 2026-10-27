---
name: do-setup
description: "Initializes the project for the DO Framework: analyzes it, inventories installed skills and MCP servers (reporting gaps), and writes the framework's local context file `.do/context.md` (summary, conventions, commands, skills and MCPs). Never edits an existing project instructions file (CLAUDE.md, AGENTS.md, .github/copilot-instructions.md, .cursor rules); when the project has none, offers to create one with project content only (no framework, skill or MCP references). Installs the orchestration agents and commands. Arguments: \"agents\" (install agents/commands only), \"refresh\" (re-detect skills/MCPs only). Use to initialize or configure the agent-assisted environment. Not for PRDs, implementation, review or QA."
---

# Project Setup

## Role
You are a senior developer advocate responsible for project initialization, tooling configuration, and ensuring the agent-assisted development environment is correctly set up.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reasons to stop and ask the user are:
1. **The mandatory initial AI tool selection question (Step 0).** This is the FIRST action the skill performs and is always required.
2. **The instructions file question (Step 1 item 4)**, asked only in `full` mode and only when the project has no instructions file at all.
3. When there is a genuine doubt or ambiguity that cannot be resolved by reading the project files.

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to analyze the project structure and produce the framework's context file. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

**An existing instructions file is read-only.** `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, `.cursor/rules/*.mdc` and `.cursorrules` belong to the project and are shared with people who may not use this framework, or who have other skills and MCPs installed. If any of them exists, never edit or append to it. Only when **none** of them exists may this skill create one (Step 4B), and only with the user's consent: the created file holds project content only — summary, commands, conventions, like a `/init` would produce — and never mentions this framework, its skills, MCPs or the `.do/` folder. Do not invoke the tool's own `/init` (not every tool lets the agent run it): write the file yourself. Everything about the framework goes into the `.do/` folder at the project root:

```
.do/
  context.md   # written by this skill: summary, conventions, commands, skills, MCPs
  tmp/         # scratch files of the execution skills, deleted at the end of each run
```

`.do/` is local to each developer (skills and MCPs differ per machine) and is ignored by git (Step 1).

## Procedures

**Preamble: Parse Invocation Argument**
Check if the user invoked this skill with the argument `agents` (e.g., `/do-setup agents`).

- If the argument **is** `agents`: set mode = `agents-only`. Skip Steps 1–4 and 6. Execute only Step 0 (AI tool selection) and Step 5 (install agents).
- If the argument **is** `refresh`: set mode = `refresh`. Execute only Step 0 (AI tool selection), Step 3 (discover skills), Step 3B (inventory MCPs), Step 4 restricted to the `## Available Skills`, `## Uncovered Skills` and `## Available MCPs` sections of `.do/context.md`, and Step 6. Skip Steps 1, 2 and 5 — except that, for **GitHub Copilot**, you must also re-sync the `<server-name>/*` MCP entries of the `tools:` list in `.github/agents/agent-execute-task.agent.md` with the MCPs found in Step 3B (add the new ones, keep the existing ones), since Copilot only exposes listed tools to the worker. `.do/context.md` must already exist; if it does not, stop and tell the user to run `/do-setup` (full) first. In this mode derive the project technologies from its `## Project Summary` section instead of re-analysing the project.
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
> 1. **Claude Code** — usa `.claude/skills/` e instala o agente em `.claude/agents/`.
> 2. **GitHub Copilot** — usa `.github/` para instruções e instala o agente em `.github/agents/`.
> 3. **Cursor AI** — usa `.agents/skills/` e `.cursor/skills/` (além das regras em `.cursor/rules/`) e instala o agente em `.cursor/agents/`.
> 4. **Opencode** — usa `.agents/skills/` e instala o agente em `.opencode/agents/` e os comandos em `.opencode/commands/`.
>
> Em todas, o contexto do framework fica em `.do/context.md` (local, ignorado pelo git). Um arquivo de instruções do projeto que já existe (`CLAUDE.md`, `AGENTS.md` etc.) nunca é alterado; se o projeto não tiver nenhum, o setup pergunta se deve criar um, só com o conteúdo do projeto.

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

| Option | AI Tool | Instructions file (read-only) | Created when none exists | Skills directory | Agents directory | Commands directory |
|--------|---------|-------------------------------|--------------------------|------------------|------------------|--------------------|
| 1 | Claude Code | `CLAUDE.md` (or `AGENTS.md` when there is no `CLAUDE.md`) | `CLAUDE.md` | `.claude/skills/` | `.claude/agents/` | `.claude/commands/` |
| 2 | GitHub Copilot | `.github/copilot-instructions.md` (or `AGENTS.md`) | `AGENTS.md` | `.github/` | `.github/agents/` | `.github/prompts/` |
| 3 | Cursor AI | `.cursor/rules/*.mdc`, `.cursorrules` (or `AGENTS.md`) | `AGENTS.md` | `.agents/skills/`, `.cursor/skills/` | `.cursor/agents/` | `.cursor/commands/` |
| 4 | Opencode | `AGENTS.md` | `AGENTS.md` | `.agents/skills/` | `.opencode/agents/` | `.opencode/commands/` |

`AGENTS.md` is the file created for every tool except Claude Code: it is plain markdown and GitHub Copilot, Cursor and Opencode all read it, so it serves any developer of the project.

Only after the user has explicitly selected one option, continue to Step 1 using the selected tool's paths and conventions consistently.

**Step 1: Prepare the `.do/` Folder**
1. Create `.do/` at the project root (`mkdir -p .do`). Never edit an existing instructions file, whichever tool was selected.
2. If the project root is a git repository (it has a `.git`) and `.gitignore` does not already ignore `.do` (with or without the trailing slash), append `.do/` once, creating `.gitignore` if it does not exist. Add nothing else to `.gitignore` and remove nothing from it. Mention in the final report whether the entry was added or already present.
3. **Legacy sections check (warn only — never edit the file):** previous versions of this skill wrote their output into the instructions file. If the instructions file of the selected tool (resolution table) exists and contains any of the headings `## Available Skills`, `## Uncovered Skills`, `## Available MCPs` or `## Project Commands`, remember which ones for the Step 6 report. Also remember if `.gitignore` still has a `.do-tmp` entry. Do not change either file.
4. **Instructions file question (`full` mode only):** check whether the project root has **any** instructions file — `CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, a `.mdc` file in `.cursor/rules/` or `.cursorrules` — regardless of the selected tool. If at least one exists, do not ask: the project already has its instructions and nothing is created. If none exists, ask the user once (structured question tool when available; otherwise plain text, ending the turn):

   > **O projeto não tem arquivo de instruções para agentes. Criar `<arquivo>` com resumo, comandos e convenções do projeto?** O arquivo vai para o repositório e serve a qualquer desenvolvedor: não terá nenhuma referência ao DO Framework, a skills ou a MCPs (isso fica em `.do/context.md`). O setup cria o arquivo, mas não o adiciona ao git.
   >
   > 1. Sim, criar `<arquivo>`
   > 2. Não, seguir sem ele

   `<arquivo>` is the "Created when none exists" column of the resolution table. Remember the answer for Step 4B and continue autonomously.

**Step 2: Deep Project Analysis**
1. Read the project's instructions files that exist (the ones checked in Step 1 item 4, read-only), and `README.md` if it exists. Ignore any section a previous version of this skill wrote there (legacy sections check, Step 1): this run rebuilds that content from the project itself.
2. List the project root and read the root-level files that declare dependencies, build, test and run commands, versions or infrastructure (manifests, build descriptors, task runners, container/orchestration files, version pins, compiler/type configuration) — whatever the project uses. Do not assume a particular language or tool: identify them from what is actually there. **Never read the AI tool's own configuration files** (`opencode.json`, `.mcp.json`, `.vscode/mcp.json`, `.cursor/mcp.json`, the tool's settings files) **nor secret files** (`.env` and similar, key/credential files) in this step: they may hold credentials and are handled only by the MCP inventory in Step 3B, which extracts names only. An example file such as `.env.example` is fine.
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
7. Extract the project commands (install, run all tests, run one test, run tests with coverage, static checks, start the service locally and its port/health address) from what the project itself declares, following `do-shared/references/do-project-commands.md`. Record only commands that exist; use `—` for the rest. They are written to `## Project Commands` in Step 4.
   - If neither is found, include in `.do/context.md`: "⚠️ AVISO: Nenhuma infraestrutura de testes detectada. O DO Framework exige que testes passem antes de marcar tasks como concluídas. Configure um test runner antes de usar `do-execute-task`."

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
5. For each kept skill record: name, **path of its `SKILL.md` file** (project-relative when inside the project, absolute when global — the path must end in `SKILL.md` and the file must exist before you write it; never a bare directory) and "when to use".
6. Technologies identified in Step 2 with no matching skill go to `## Uncovered Skills`.
7. **Global copies of DO skills (warn only):** while scanning the global directories, note every `do-*` skill found **outside the project** (`~/.agents/skills/`, `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.cursor/skills/`). For each one, compare its `SKILL.md` with the project copy (`cmp`) and record `idêntica` or `diferente` (or `sem cópia no projeto`). Do not list them in `.do/context.md` and do not delete or edit anything: this is only reported in Step 6. Reason: when the same skill name exists globally and in the project, the tool decides which one loads — Claude Code documents that the **personal/global copy wins over the project copy**; for Opencode, Cursor and GitHub Copilot the documentation does not say.

**Step 3B: Inventory MCP Servers (read-only)**
Skip in `agents-only` mode. This step only **reads** configuration — never start, call or test an MCP server, and never make network calls.
1. Follow the discovery procedure in `do-shared/references/do-mcp-discovery-instructions.md` (project MCP config file for the selected tool, global config, and the tools exposed in the session). **Security: never `cat` or read a whole configuration file** — global configs hold provider credentials. Extract only the server names and types, as the discovery instructions require; if you cannot isolate the MCP block, skip the file and rely on the tools exposed in the session and map each server with `do-shared/references/do-mcp-capabilities.md`. Both live in the skills directory, next to `do-setup` (the project-local `<skill-dir>` parent from Step 5): read them by full path, since glob tools skip hidden directories such as `.agents/`.
2. The AI tool's own native tools (file, shell, search, session, worktree or subagent tools) are **not** MCP servers: never list them. For each MCP server found record: name, source (project config, global config or session tools), type (local/remote) and capability. A server absent from the registry is listed as `não registrado`. **Never copy tokens, headers, URLs with credentials or environment values** from the config into any output.
3. Compute informational **gaps** from the Step 2 summary (never errors):
   - project has a user interface and no `ui-testing` capability → UI E2E will be skipped by `do-execute-task`/`do-execute-qa`;
   - project integrates with a service (messaging, data store, API) and no matching capability (`messaging`, `data-store`, `api-testing`) → service E2E will be skipped;
   - no `documentation` capability → library docs will come from Web Search or the codebase.
4. Zero MCPs found is a valid result.

**Step 4: Write `.do/context.md`**
Write `.do/context.md` with the content below. The file belongs to the framework: in `full` mode write it whole (overwrite any previous version); in `refresh` mode rewrite only `## Available Skills`, `## Uncovered Skills` and `## Available MCPs` and keep the other sections as they are. Framework content (skills, MCPs, `do-*`, `.do/`) goes only here, never into an instructions file.

> **Instrução ao agente (não incluir no arquivo gerado):** Se o Step 3 não encontrou nenhuma skill de tecnologia, a tabela de `## Available Skills` recebe uma única linha `_(nenhuma)_ | — | Nenhuma skill de tecnologia detectada` e a linha "Exemplo:" é omitida. Se o Step 3B não encontrou MCPs, a tabela de `## Available MCPs` recebe uma única linha `_(nenhum)_ | — | Nenhum MCP detectado`. Nunca escreva tokens, headers ou valores de ambiente. Nunca inclua skills `do-*` (do-setup, do-create-prd, do-create-techspec, do-create-tasks, do-execute-task, do-execute-review, do-execute-qa, etc.) na tabela.

```markdown
# Contexto do DO Framework

> Gerado pelo `/do-setup` para as skills `do-*`. Local a cada desenvolvedor e ignorado pelo git: as skills e os MCPs listados são os desta máquina. Para atualizar depois de instalar skills ou MCPs, rode `/do-setup refresh`.

## Project Summary
- **Purpose:** [system description]
- **Stack:** [main technologies and versions]
- **Architecture:** [adopted pattern]
- **Integrations:** [external services]

## Available Skills
> Skills instaladas depois do setup podem não constar aqui: as skills `do-*` completam a lista com a da sua ferramenta (`do-shared/references/do-project-context.md`).

| Skill | Caminho | Quando usar |
|-------|---------|-------------|
| [name] | [path of the skill's SKILL.md file] | [usage context] |

## Uncovered Skills
| Tecnologia | Observação |
|------------|------------|
| [technology identified in Step 2] | Nenhuma skill disponível localmente — adicione manualmente ao diretório de skills |

## Available MCPs
> Resumo; a descoberta em tempo de execução (`do-shared/references/do-mcp-discovery-instructions.md`) prevalece.

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
| Rodar testes com cobertura | [command or —] | [only if the project declares one] |
| Checagem estática | [command(s) or —] | [type checking, lint, formatting — whatever the project defines] |
| Iniciar o serviço local | [command or —] | [port or health address] |
```

**Step 4B: Create the Instructions File (only if the user accepted in Step 1 item 4)**
Skip this step in `agents-only` and `refresh` modes, when an instructions file already exists, or when the user declined.
1. Check again that none of the files listed in Step 1 item 4 exists; if one appeared, do not create anything.
2. Write `<arquivo>` (resolution table, "Created when none exists") at the project root, built from the Step 2 analysis, with the content below. It is project documentation for any developer and any agent: **never** mention the DO Framework, `do-*` skills, other skills, MCP servers, `.do/` or anything specific to this machine (absolute paths, home directory, installed tools).
3. Do not run `git add` or commit it: the developer decides whether it goes to the repository.

> **Agent instruction (do not include in the generated file):** Write this file in **English**, the default language of the agent tools' own `/init` — not in the framework's output language. Include only what the Step 2 analysis confirmed in the project; omit a section that does not apply instead of filling it with guesses. The commands are the same as in `## Project Commands`, without the `—` rows.

```markdown
# [Project name]

## Overview
[System purpose in two or three sentences]

## Stack
- [main technology and version]

## Architecture
[Adopted pattern and how layers or modules are organized]

## Directory Structure
- `[path]` — [what it contains]

## Commands
| Purpose | Command |
|---------|---------|
| [purpose] | `[command]` |

## Conventions
- **Naming:** [file, folder and symbol naming patterns]
- **Tests:** [where they live and how they are named]
- **Other:** [patterns observed in the code that a contributor must follow]
```

**Step 5: Install the Worker Agent (and, for Opencode, the command wrappers)**
The logic of the queue lives in the skill `do-execute-all-tasks` and the contract of the worker lives in `do-execute-task/references/subagent-mode.md`; both are read from disk at run time. What this step installs is only thin, tool-specific glue, so a cached or outdated copy of it can never impose old rules:

- **All tools:** the `agent-execute-task` worker subagent (permissions + a pointer to the skill and the contract). The same worker runs one task, one review fix or one QA bug, depending on the skill named in the prompt.
- **Opencode only:** one-line command wrappers (`/do-create-prd`, `/do-execute-all-tasks`, …) in `.opencode/commands/`, because Opencode does not list skills under `/`.
- **Claude Code, Cursor, GitHub Copilot:** **no command or prompt files.** These tools already expose each skill (including `do-execute-all-tasks`) for invocation, and a command with the same name would appear duplicated.

The orchestration runs on the primary agent and each task is delegated through the platform's delegation tool — exactly **one** nesting level, which every tool allows:

- **Claude Code**: since v2.1 subagents can spawn subagents (up to three levels, `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`; [docs](https://code.claude.com/docs/en/sub-agents)), but the queue still runs on the main agent so each task stays one level deep and permission prompts surface in the main session. `agent-execute-task` blocks delegation with `disallowedTools: Agent` instead of a `tools` allowlist, so it keeps MCP tools (needed for E2E tests) and preloads the `do-execute-task` skill via `skills`.
- **Cursor, GitHub Copilot**: subagents **cannot** spawn other subagents by default (Copilot requires `chat.subagents.allowInvocationsFromSubagents=true`, off by default; Cursor allows one nesting level since 2.5, but a grandchild cannot launch further subagents).
- **Opencode**: nesting is capped by `subagent_depth` (**default `1`**), and a subagent only receives the delegation tool if it declares `permission.task`. Do not mark `agent-execute-task` as `hidden: true`: since opencode 2.1, hidden agents are omitted from the "Available subagents" list, so the orchestrator cannot see the worker. Even with `subagent_depth: 2`, permission prompts raised inside a grandchild session may never render in the TUI — the queue then waits forever. So no orchestrator subagent is installed.

> Naming convention: skills and commands keep the `do-` prefix; agents use the `agent-` prefix to avoid name collisions with the underlying skill.

1. Resolve `<skill-dir>` **inside the current project only**. The templates were copied alongside the `SKILL.md` when the DO Framework skills were installed in this project.
   - Check these paths, in order, relative to the project root, and use the first that exists: `.agents/skills/do-setup`, `.claude/skills/do-setup`. If neither exists, use the `do-setup` directory that the installation created **inside the project root**.
   - **NEVER** use a `do-setup` copy outside the project root (parent directories such as `~/projects/.agents/`, global/home skill directories, other projects). Those copies can be stale. Do not run a recursive Glob from a parent directory to find it.
   - Sanity check: `<skill-dir>/agents/<tool>/agents/` must contain `agent-execute-task.md` (`agent-execute-task.agent.md` for GitHub Copilot) and must **not** contain any `*execute-all-tasks*` file; the skill `do-execute-all-tasks` must exist next to `do-setup`. If a check fails, the copy is stale: **STOP** and tell the user to reinstall the DO Framework skills in this project (see the installation instructions in the README).
2. Execute **only** the installation block that matches the AI tool selected by the user in Step 0. Do not install assets for other tools.

   **Claude Code** (if the user selected option 1):
   - Run `mkdir -p .claude/agents`
   - Copy `<skill-dir>/agents/claude/agents/agent-execute-task.md` → `.claude/agents/agent-execute-task.md`
   - Delete leftovers of previous installs (legacy orchestrator agents and the old loop command, which would now duplicate the skill): `rm -f .claude/agents/agent-execute-all-tasks.md .claude/agents/do-execute-all-tasks.md .claude/agents/execute-all-tasks.md .claude/commands/do-execute-all-tasks.md`.

   **Cursor AI** (if the user selected option 3):
   - Run `mkdir -p .cursor/agents`
   - Copy `<skill-dir>/agents/cursor/agents/agent-execute-task.md` → `.cursor/agents/agent-execute-task.md`
   - Delete leftovers of previous installs: `rm -f .cursor/agents/agent-execute-all-tasks.md .cursor/agents/do-execute-all-tasks.md .cursor/agents/execute-all-tasks.md .cursor/commands/do-execute-all-tasks.md`.

   **GitHub Copilot** (if the user selected option 2):
   - Run `mkdir -p .github/agents`
   - Copy `<skill-dir>/agents/github/agents/agent-execute-task.agent.md` → `.github/agents/agent-execute-task.agent.md`
   - **MCP tools for the worker:** the template's `tools:` list has no MCP entries (GitHub Copilot only exposes to a custom agent the tools it lists). After copying, append one `  - <server-name>/*` line to the `tools:` list of `.github/agents/agent-execute-task.agent.md` for **every MCP server** found by the Step 3B discovery (in `agents-only` mode run just the read-only discovery of Step 3B, item 1, for this purpose), using the server names as written in `.vscode/mcp.json`. If no MCP server is found, append nothing. This keeps Copilot equivalent to the other tools, where the worker sees whatever MCPs the project has.
   - Delete leftovers of previous installs: `rm -f .github/agents/agent-execute-all-tasks.agent.md .github/agents/do-execute-all-tasks.agent.md .github/agents/execute-all-tasks.agent.md .github/prompts/do-execute-all-tasks.prompt.md`.

   **Opencode** (if the user selected option 4):
   - Run `mkdir -p .opencode/agents .opencode/commands`
   - Copy `<skill-dir>/agents/opencode/agents/agent-execute-task.md` → `.opencode/agents/agent-execute-task.md`
   - Copy all `<skill-dir>/agents/opencode/commands/*.md` → `.opencode/commands/`
   - **Do NOT** copy any orchestrator agent — the queue runs in the **primary** session via the skill. A command with `subtask: true` delegating to an orchestrator subagent creates two nesting levels: it fails with `Subagent depth limit reached` at the default (`subagent_depth: 1`) and, even with `subagent_depth: 2`, grandchild permission prompts may not show in the TUI and the queue hangs.
   - Delete any orchestrator agent left by previous installs (all legacy names): `rm -f .opencode/agents/agent-execute-all-tasks.md .opencode/agents/do-execute-all-tasks.md .opencode/agents/execute-all-tasks.md`.

3. Verify the installation before reporting:
   - The agents directory of the selected tool contains `agent-execute-task` and **no** file matching `*execute-all-tasks*`.
   - For Opencode: the installed `.opencode/commands/do-execute-all-tasks.md` is byte-identical to the template in `<skill-dir>` (compare with `cmp`) and contains neither `subtask: true` nor an `agent:` field in the frontmatter. For the other tools: no `do-execute-all-tasks` command or prompt file exists (the skill is the only entry point).
   - If any check fails, fix it (re-copy / delete) and verify again. Never report success with a legacy orchestrator or a duplicate loop command installed.
4. Confirm to the user which files were installed, for which tool, and the `<skill-dir>` used as source.
5. **Tell the user how to load what was just installed, by tool** (this applies to the worker agent and, on Opencode, the command wrappers): **Opencode** — run `/reload`; **Claude Code** — skills are picked up live (use `/reload-skills` only if `.claude/skills/` was just created), but a `.claude/agents/` directory created during this setup is only detected after restarting the session; **Cursor** and **GitHub Copilot** — the documentation does not say, so tell the user to reload the IDE window or open a new chat session if the agent does not appear. Include this notice in the final report of Step 6.

6. **Ignore the framework folder** (`agents-only` mode only; the other modes already did it in Step 1): apply Step 1 item 2, so the execution skills' scratch folder `./.do/tmp/` is never committed.

> Use Bash to run `cp` commands. `<skill-dir>` is the project-local `do-setup` directory resolved in item 1 (e.g. `.agents/skills/do-setup`).

**Step 6: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once `.do/context.md` is written, use the `TaskUpdate` tool to mark all corresponding items in your internal task tracking as `completed`.
2. **ARTIFACT PATH VERIFICATION**: Before reporting, read `.do/context.md` back to verify it exists and contains the expected content, and confirm that no instructions file that existed before this run was modified. If Step 4B created one, read it back and confirm it does not mention the DO Framework, `do-*`, skills, MCPs or `.do/`; remove any such line.
3. Provide a summary of the setup performed. It MUST include this inventory (omit empty lines only if the corresponding list is empty, and say so):
   - **Skills detectadas:** name and path of each relevant technology skill, or "nenhuma skill de tecnologia detectada".
   - **MCPs detectados:** name, capability and source of each server (mark `não registrado` when applicable), or "nenhum MCP detectado".
   - **Lacunas:** the informational gaps computed in Step 3B (e.g. "sem MCP de interface: o E2E de interface será pulado") and, when no skills/MCPs were found, a hint to install them and run `/do-setup refresh`.
   - **Cópias globais de skills `do-*` (apenas se o passo 7 do Step 3 encontrou):** list each one with its path and whether it is `idêntica` or `diferente` from the project copy, then warn: "Existe cópia global de `do-x`. Se o mesmo nome existe no projeto e na máquina, a ferramenta escolhe qual carregar: no Claude Code a global vence a do projeto; nas demais a documentação não especifica. Mantenha só uma das cópias para evitar rodar uma versão diferente da esperada." Do not remove anything.
   - **Arquivo de instruções:** one of: "`<arquivo>` já existia e não foi alterado"; "`<arquivo>` criado em inglês com resumo, comandos e convenções do projeto, sem referências ao framework. Ele não foi adicionado ao git: revise e faça o commit se quiser compartilhá-lo com o time"; or "o projeto não tem arquivo de instruções (criação recusada)".
   - **Seções antigas no arquivo de instruções (apenas se o legacy sections check do Step 1 encontrou):** "O `<arquivo>` ainda tem as seções `<lista>`, gravadas por uma versão anterior do `do-setup`. O framework agora lê tudo de `.do/context.md` e não altera esse arquivo: remova essas seções manualmente se foram geradas pelo framework." Se o `.gitignore` ainda tiver `.do-tmp`, acrescente: "A entrada `.do-tmp/` do `.gitignore` não é mais usada (os rascunhos ficam em `.do/tmp/`) e pode ser removida." Do not change either file.
   End with the load notice from Step 5 item 5 (how to load the installed agents/commands in the selected tool) — except in `refresh` mode, where no agents were installed and the notice is omitted.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is `.do/context.md` saved at the project root, and is `.do/` ignored by git (when the project is a git repository)?
    - Did you accurately identify the project stack and skills?
    - Confirme que a seção `## Available MCPs` existe em `.do/context.md` e que nenhum token, header ou valor de ambiente foi copiado para ele.
    - Confirme que nenhum arquivo de instruções que já existia (`CLAUDE.md`, `AGENTS.md` etc.) foi alterado e que, se um foi criado, foi com o consentimento do usuário e sem referências ao framework, a skills ou a MCPs.

## Output Language
Todos os artefatos gerados (`.do/context.md`, resumos) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês. **Exceção:** o arquivo de instruções criado no Step 4B (`CLAUDE.md`/`AGENTS.md`) é escrito em inglês, o idioma padrão do `/init` das ferramentas agênticas.

## Error Handling
- If the project root has no files declaring dependencies, build or run commands, warn the user that the project may not be initialized and ask for clarification about the stack.
- If the project has no instructions file, this is not an error: ask once (Step 1 item 4) and, if the user declines, proceed without it.
- If no skills (other than `do-*`) or no MCP servers are found, this is not an error: write the empty-state rows, report it in the inventory and suggest installing them and running `/do-setup refresh`.
- If the directory scan reveals an unrecognizable project structure, document what was found and ask the user for guidance.

## References
- Output: `.do/context.md` (local, ignored by git); `CLAUDE.md` (Claude Code) or `AGENTS.md` (other tools) only when the project has no instructions file and the user accepts (Step 4B). Input only: an existing instructions file (`CLAUDE.md` or `AGENTS.md` for Claude Code, `AGENTS.md` for Opencode, `.github/copilot-instructions.md` or `AGENTS.md` for GitHub Copilot, `.cursor/rules/*.mdc`, `.cursorrules` or `AGENTS.md` for Cursor AI)
- Skills discovery: the tool's native skill list plus the directories scanned in Step 3. Conventional locations by tool:
  - Claude Code: `.claude/skills/` (each skill has a `SKILL.md`)
  - Opencode: `.agents/skills/` (each skill has a `SKILL.md`)
  - GitHub Copilot: `.github/` (instruction files)
  - Cursor AI: `.agents/skills/` and `.cursor/skills/` (each skill has a `SKILL.md`); rules in `.cursor/rules/` (`.mdc` files with frontmatter)

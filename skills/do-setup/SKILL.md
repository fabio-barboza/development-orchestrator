---
name: do-setup
description: Initializes the project, identifies relevant skills, and updates the project configuration file (CLAUDE.md, .github/copilot-instructions.md, .cursor/rules/project.mdc, or .cursorrules) with project summary, conventions, and available skills. Accepts an optional argument "agents" to only install orchestration agents and commands without running the full setup. Use when the user asks to initialize the project, configure the agent-assisted development environment, or install agents with "/do-setup agents". Do not use for PRD creation, task implementation, code review, or QA testing.
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
> 3. **Cursor AI** — gera `.cursor/rules/project.mdc` (ou `.cursorrules` legado), usa `.cursor/rules/`, instala agentes em `.cursor/agents/` e comandos em `.cursor/commands/`.
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
| 3 | Cursor AI | `.cursor/rules/project.mdc` (or `.cursorrules` if the user explicitly asks for the legacy format) | `.cursor/rules/` | `.cursor/agents/` | `.cursor/commands/` |
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

**Step 2: Deep Project Analysis**
1. Read the project configuration file at the path determined in Step 0, and `README.md` if it exists.
2. Read root config files if they exist: `package.json`, `go.mod`, `pom.xml`, `build.gradle`, `build.gradle.kts`, `docker-compose.yml`, `tsconfig.json`, `settings.gradle`, `.nvmrc`, `Makefile`, `Dockerfile`.
3. Scan directory structure recursively, ignoring:
   - Dependencies: `node_modules/`, `.venv/`, `venv/`, `vendor/`, `.gradle/`, `.m2/`
   - Build: `target/`, `build/`, `dist/`, `out/`, `.next/`, `__pycache__/`
   - Hidden: any path starting with `.` (except `.claude/`, `.github/`, and `.cursor/`)
   - Binaries/media: `*.jar`, `*.class`, `*.png`, `*.jpg`, `*.pdf`
4. Read representative files from each layer (e.g., a controller, a use case, a repository) to understand adopted patterns.
5. Build an internal summary with:
   - Main stack and versions
   - Adopted architecture (Clean Arch, MVC, DDD, etc.)
   - Naming and organization patterns
   - System purpose
   - External integrations (queues, databases, APIs)
6. Check test infrastructure:
   - Look for a `test` script in `package.json` (or equivalent for the stack).
   - Scan for test files (`*.test.*`, `*.spec.*`, `__tests__/`, `test/`, `tests/`).
   - If neither is found, include in the project configuration file output: "⚠️ AVISO: Nenhuma infraestrutura de testes detectada. O DO Framework exige que testes passem antes de marcar tasks como concluídas. Configure um test runner antes de usar `do-execute-task`."

**Step 3: Identify Relevant Skills**
1. List all available skills by scanning the skills directory corresponding to the AI tool the user selected in Step 0:
   - **Claude Code**: `.claude/skills/`
   - **Opencode**: `.agents/skills/`
   - **GitHub Copilot**: `.github/` (look for instruction files)
   - **Cursor AI**: `.cursor/rules/` (scan all `.mdc` files)
   List every skill/rule file found and read its content.
2. **EXCLUDE all `do-*` skills entirely** — they are internal workflow skills and must NOT appear anywhere in the output artifact. Only evaluate technology/library skills (e.g., `claude-api`, `find-skills`).
3. For each remaining (non-`do-*`) skill, read its descriptor file (`SKILL.md` for Claude Code, the `.mdc` content for Cursor AI).
4. Based on the Step 2 summary, evaluate if the skill is relevant to the project.
5. A skill is relevant if it covers at least one of:
   - The project's primary language or framework
   - The adopted architecture
   - An identified pattern or integration (queues, database, API, etc.)

**Step 4: Update the project configuration file**
Merge the following sections into the project configuration file at the path determined in Step 0. Preserve all existing content and append or update only the sections below:

```markdown
## Project Summary
- **Purpose:** [system description]
- **Stack:** [main technologies and versions]
- **Architecture:** [adopted pattern]
- **Integrations:** [external services]

## Available Skills
**NOTE: Only list non-`do-*` skills here. Never include workflow skills (do-setup, do-create-prd, do-create-techspec, do-create-tasks, do-execute-task, do-execute-review, do-execute-qa, etc.) in this table.**
| Skill | Path | When to use |
|-------|------|-------------|
| [name] | [skills-dir]/[skill]/SKILL.md | [usage context] |

## Uncovered Skills
| Technology | Note |
|------------|------|
| [e.g., Java/Quarkus] | No skill available locally — add manually to the skills directory |

## Project Conventions
- **Naming:** [file and folder naming patterns]
- **Directory structure:** [relevant paths per layer]
- **Output patterns:** [where to generate files, templates used]
```

**Step 5: Install Orchestration Agents & Commands**
Based on the AI tool selected by the user in Step 0, install the `agent-execute-task` worker subagent and the slash command for orchestration. The orchestration logic of the queue lives in different places depending on the AI tool's subagent nesting policy:

- **Claude Code, Cursor, GitHub Copilot**: subagents **cannot** spawn other subagents by default ([Claude Code docs](https://code.claude.com/docs/en/sub-agents); Copilot requires `chat.subagents.allowInvocationsFromSubagents=true`, which is off by default). The orchestration loop lives **inside the slash command/prompt itself** (runs on the main agent), and only `agent-execute-task` is installed as a subagent. Each task is delegated via the platform's task tool (one nesting level — allowed).
- **Opencode**: subagent nesting is allowed natively. Both `agent-execute-all-tasks` (orchestrator) and `agent-execute-task` (worker) are installed as subagents.

> Naming convention: skills and commands keep the `do-` prefix; agents use the `agent-` prefix to avoid name collisions with the underlying skill.

1. Locate the `agents/` subdirectory inside the `do-setup` skill directory by searching for `**/do-setup/agents` using Glob. This directory was copied alongside the `SKILL.md` when the user ran `npx skills add`.
2. Execute **only** the installation block that matches the AI tool selected by the user in Step 0. Do not install assets for other tools.

   **Claude Code** (if the user selected option 1):
   - Run `mkdir -p .claude/agents .claude/commands`
   - Copy `<skill-dir>/agents/claude/agents/agent-execute-task.md` → `.claude/agents/agent-execute-task.md`
   - Copy `<skill-dir>/agents/claude/commands/do-execute-all-tasks.md` → `.claude/commands/do-execute-all-tasks.md`
   - **Do NOT** copy any `agent-execute-all-tasks.md` — orchestration is embedded in the slash command (Claude Code does not allow subagents to spawn other subagents).
   - If a previous install left a stale `.claude/agents/agent-execute-all-tasks.md` in the project, delete it (`rm -f .claude/agents/agent-execute-all-tasks.md`).

   **Cursor AI** (if the user selected option 3):
   - Run `mkdir -p .cursor/agents .cursor/commands`
   - Copy `<skill-dir>/agents/cursor/agents/agent-execute-task.md` → `.cursor/agents/agent-execute-task.md`
   - Copy `<skill-dir>/agents/cursor/commands/do-execute-all-tasks.md` → `.cursor/commands/do-execute-all-tasks.md`
   - **Do NOT** copy any `agent-execute-all-tasks.md` — orchestration is embedded in the slash command.
   - If a previous install left a stale `.cursor/agents/agent-execute-all-tasks.md`, delete it (`rm -f .cursor/agents/agent-execute-all-tasks.md`).

   **GitHub Copilot** (if the user selected option 2):
   - Run `mkdir -p .github/agents .github/prompts`
   - Copy `<skill-dir>/agents/github/agents/agent-execute-task.agent.md` → `.github/agents/agent-execute-task.agent.md`
   - Copy `<skill-dir>/agents/github/prompts/do-execute-all-tasks.prompt.md` → `.github/prompts/do-execute-all-tasks.prompt.md`
   - **Do NOT** copy any `agent-execute-all-tasks.agent.md` — orchestration is embedded in the prompt.
   - If a previous install left a stale `.github/agents/agent-execute-all-tasks.agent.md`, delete it (`rm -f .github/agents/agent-execute-all-tasks.agent.md`).

   **Opencode** (if the user selected option 4):
   - Run `mkdir -p .opencode/agents .opencode/commands`
   - Copy `<skill-dir>/agents/opencode/agents/agent-execute-all-tasks.md` → `.opencode/agents/agent-execute-all-tasks.md`
   - Copy `<skill-dir>/agents/opencode/agents/agent-execute-task.md` → `.opencode/agents/agent-execute-task.md`
   - Copy all `<skill-dir>/agents/opencode/commands/*.md` → `.opencode/commands/`

3. Confirm to the user which files were installed and for which tool.

> Use Bash to run `cp` commands. `<skill-dir>` is the path returned by the Glob search for `**/do-setup/agents` (without the trailing `/agents`).

**Step 6: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the project configuration file is updated, use the `TaskUpdate` tool to mark all corresponding items in your internal task tracking as `completed`.
2. **ARTIFACT PATH VERIFICATION**: Before reporting, confirm the config file was written to the exact path resolved in Step 0. Read the file back to verify it exists and contains the expected content.
3. Provide a summary of the setup performed.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the project configuration file saved at the correct path (resolved in Step 0)?
    - Did you accurately identify the project stack and skills?

## Output Language
Todos os artefatos gerados (seções do arquivo de configuração do projeto, resumos) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If no config files are found (no `package.json`, `go.mod`, etc.), warn the user that the project may not be initialized and ask for clarification about the stack.
- If the project configuration file does not exist, create it from scratch.
- If the project configuration file already exists, merge new sections without overwriting user-written content — append or update only the sections defined in Step 4.
- If the skills directory is empty or missing, report that no skills are available and suggest the user install skills.
- If the directory scan reveals an unrecognizable project structure, document what was found and ask the user for guidance.

## References
- Output: Project configuration file (e.g., `CLAUDE.md` for Claude Code, `AGENTS.md` for Opencode, `.github/copilot-instructions.md` for GitHub Copilot, `.cursor/rules/project.mdc` or `.cursorrules` for Cursor AI)
- Skills directories:
  - Claude Code: `.claude/skills/` (each skill has a `SKILL.md`)
  - Opencode: `.agents/skills/` (each skill has a `SKILL.md`)
  - GitHub Copilot: `.github/` (instruction files)
  - Cursor AI: `.cursor/rules/` (`.mdc` files with frontmatter)

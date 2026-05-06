---
name: do-setup
description: Initializes the project, identifies relevant skills, and updates the project configuration file (CLAUDE.md, .github/copilot-instructions.md, .cursor/rules/project.mdc, or .cursorrules) with project summary, conventions, and available skills. Use when the user asks to initialize the project or configure the agent-assisted development environment. Do not use for PRD creation, task implementation, code review, or QA testing.
---

# Project Setup

## Role
You are a senior developer advocate responsible for project initialization, tooling configuration, and ensuring the agent-assisted development environment is correctly set up.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reason to stop and ask the user is when there is a genuine doubt or ambiguity that cannot be resolved by reading the project files.

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to analyze the project structure and produce the configuration document. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

## Procedures

**Step 0: Detect AI Tool Environment**
Before doing anything else, determine which AI tool is executing this skill:
1. Check for `.claude/` directory in the project root → **Claude Code** → config file: `CLAUDE.md`
2. Check if `.github/copilot-instructions.md` already exists → **GitHub Copilot** → config file: `.github/copilot-instructions.md`
3. Check if `.github/` directory exists but `copilot-instructions.md` does not → likely **GitHub Copilot** → config file: `.github/copilot-instructions.md`
4. Check for any of the following Cursor AI indicators → **Cursor AI**:
   - `.cursor/rules/` directory exists → config file: `.cursor/rules/project.mdc`, skills dirs: `.cursor/rules/`
   - `.cursor/mcp.json` exists → confirms Cursor AI (use together with rules detection)
   - `.cursorrules` file exists → config file: `.cursorrules`, skills dirs: none (legacy format)
5. If none of the above, infer from the current tool context. When in doubt, default to `CLAUDE.md`.

Store the resolved config file path, the detected AI tool name, and the skills directories internally. Use them consistently throughout all remaining steps.

**Step 1: Initialize Project Configuration**
The initialization strategy depends on the detected AI tool:

- **Claude Code**: Execute the `/init` skill to generate the initial `CLAUDE.md`. Wait for it to complete before proceeding.
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
1. List all available skills by scanning the AI tool's skills directories:
   - **Claude Code**: `.claude/skills/`
   - **GitHub Copilot**: `.github/` (look for instruction files)
   - **Cursor AI**: `.cursor/rules/` (scan all `.mdc` files)
   For each directory, list every skill/rule file found and read its content.
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

**Step 5: Report Results & Sync Progress (Mandatory)**
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
- Output: Project configuration file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, `.cursor/rules/project.mdc` or `.cursorrules` for Cursor AI)
- Skills directories:
  - Claude Code: `.claude/skills/` (each skill has a `SKILL.md`)
  - GitHub Copilot: `.github/` (instruction files)
  - Cursor AI: `.cursor/rules/` (`.mdc` files with frontmatter)

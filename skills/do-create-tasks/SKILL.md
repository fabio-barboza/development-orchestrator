---
name: do-create-tasks
description: Converts PRD and Tech Spec into a detailed, sequenced list of implementation tasks. Each task is a functional, incremental deliverable with its own test suite. Outputs tasks.md and individual task files. Use when the user asks to create tasks, break down work, or plan implementation from an existing PRD and Tech Spec. Do not use for PRD creation, tech spec creation, or actual code implementation.
---

# Task Creation

## Role
You are a senior project manager specialized in breaking down features into incremental, independently deliverable tasks.

## Interactive Execution Policy
**This skill is interactive by design.** It requires user approval at Step 3 (high-level task list) before generating files. Do NOT proceed past Step 3 without explicit user confirmation.

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to produce the task breakdown documents. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** create or reference a path like `./prds/user-auth/` (without the `prd-` prefix). The `tasks/` subdirectory is always inside this prefixed folder: `./prds/prd-[feature-slug]/tasks/`.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code**
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot**
3. Check for `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI**
4. Check for `opencode.json`, `.opencode/` directory, or `AGENTS.md` → **Opencode**
5. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, Cursor, and Opencode, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Skill assets/references are loaded via your AI tool's native skill resolver — do not hard-code paths. Store the detected tool and capability flags internally.

**Step 1: Validate Prerequisites**
1. Confirm the feature slug has been provided.
2. Verify the PRD exists at `./prds/prd-[feature-slug]/prd.md`. The directory MUST be `prd-[feature-slug]` — never `[feature-slug]` alone. If missing, halt.
3. Verify the Tech Spec exists at `./prds/prd-[feature-slug]/techspec.md`. If missing, halt.
4. **Path check**: Before creating any file, confirm you are writing to `./prds/prd-[feature-slug]/tasks/` — not `./prds/[feature-slug]/tasks/`.
5. **Detect project surface (silent)**: set `project_surface = visual | backend`. Force `visual` if the PRD has an "Identidade Visual" section OR the Tech Spec has an "Implementação Visual" section. Otherwise inspect frontend signals (deps, file extensions, directories). Default to `backend` when uncertain. Visual-coverage rules below apply ONLY when `project_surface = visual` AND the specific task touches UI files.

**Step 2: Analyze PRD and Tech Spec (Mandatory)**
1. Read the PRD completely to extract requirements.
2. Read the Tech Spec completely to extract technical decisions.
3. Use Context7 MCP (`resolve-library-id` → `query-docs`) to check documentation of frameworks/libraries involved — this helps estimate task complexity and define accurate implementation steps. If Context7 MCP is unavailable, proceed without it.
4. Identify main components and their dependencies.

**Step 3: Generate High-Level Task List (Mandatory)**
1. Present the high-level task list to the user for approval BEFORE generating any files.
2. Organize tasks by logical deliverable.
3. Order tasks logically: dependencies before dependents (e.g., backend before frontend, both before E2E tests).
4. Each task MUST be a functional, incremental deliverable.
5. Each task MUST have its own set of unit and integration tests.
6. Limit to a maximum of 15 tasks (group as needed).
7. **Scope guideline**: each task should represent approximately 100–200 lines of production code change. Tasks estimated to exceed this should be split — this prevents context overflow during `do-execute-task`.
8. Wait for user approval before proceeding to Step 4.

**Step 4: Generate Task Files (Mandatory)**
1. Read the tasks summary template at `do-create-tasks/assets/tasks-template.md`.
2. Read the individual task template at `do-create-tasks/assets/task-template.md`.
3. **PATH VERIFICATION**: Before creating any file, confirm the target directory is exactly `./prds/prd-[feature-slug]/tasks/`. Verify the parent directory name starts with `prd-`. Never write to `./prds/[feature-slug]/tasks/` (missing `prd-` prefix).
4. Create the directory `./prds/prd-[feature-slug]/tasks/` if it does not exist.
5. Create the summary file: `./prds/prd-[feature-slug]/tasks/tasks.md`.
6. Create individual task files: `./prds/prd-[feature-slug]/tasks/[num]_task.md`.
   **MANDATORY FILENAME RULE — ABSOLUTE, NON-NEGOTIABLE:**
   - The filename MUST be exactly `<integer>_task.md` — e.g., `1_task.md`, `2_task.md`, `10_task.md`, `11_task.md`.
   - **PROHIBITED**: descriptive slugs (`11_weekly_forecast.md` ❌), decimal numbering in filename (`1.0_task.md` ❌), title fragments (`1_task_login.md` ❌), uppercase (`1_Task.md` ❌), prefixes (`task_1.md` ❌).
   - The task title (e.g., "Tarefa 1.0: Weekly Forecast") goes INSIDE the file content — NEVER in the filename.
   - The integer matches the main task number from `tasks.md` (the X in `X.0`). For task `11.0`, the file is `11_task.md`.
7. Use format X.0 for main tasks, X.Y for subtasks **inside the file content only** — never in the filename.
8. Do NOT repeat implementation details already in the Tech Spec — reference it instead.
9. **VISUAL IDENTITY COVERAGE (conditional — applies ONLY when `project_surface = visual` AND the specific task touches UI files)**:
   - Identify which tasks touch UI files (components, screens, native UI views, style sheets). Backend-only tasks within a visual project (controllers, services, migrations, jobs, queue workers) MUST NOT receive visual blocks — leave the "Identidade Visual" template section OUT of those task files.
   - For UI-touching tasks:
     - Fill the "Identidade Visual" section of the task template with: style files to be modified, classes/selectors to create, design tokens to use, breakpoints / adaptive rules to respect, themes to apply.
     - Reference the visual identity requirements from the PRD and the "Implementação Visual" section of the Tech Spec.
     - Include a `<critical>` tag on style coverage:
       `<critical>Toda classe/seletor de estilo usado na UI DEVE ter regra correspondente no arquivo de estilo. Quando o review (Step 6.5 do do-execute-task) detectar lacunas, o executor TENTARÁ fechá-las ativamente com até 2 ciclos de retry visual (gerar regras faltantes, trocar literais por design tokens, adicionar adaptações). Após visual_retry = 2/2: gaps MAIOR/MENOR remanescentes são documentados e o pipeline continua; gaps CRÍTICO remanescentes (4+ classes faltando, tela visualmente quebrada, theme exigido não aplicado) HALTAM o pipeline — paralelo a teste falhando.</critical>`
   - When `project_surface = backend`, OMIT the visual block from every task file.
10. **POST-SAVE VERIFICATION**: After writing all files, list the contents of `./prds/prd-[feature-slug]/tasks/` to confirm all expected files exist. If any file is missing, halt and report the error.
11. **DUPLICATE & FILENAME GUARD (applies to all AI tools)**:
    - Spurious decimal-prefixed duplicates (e.g., `1.0_task.md` alongside the canonical `1_task.md`) corrupt the task pipeline and must be detected and removed regardless of which AI tool is running this skill.
    - Before creating any file, plan and write down the exact list of filenames you will produce. Every filename MUST match the regex `^[0-9]+_task\.md$` or be exactly `tasks.md`. If your plan includes ANY filename with a decimal point (e.g., `1.0_task.md`, `2.0_task.md`), STOP and rewrite the plan — those filenames are PROHIBITED.
    - Create files strictly one per task. Do NOT create both `<X>_task.md` and `<X>.0_task.md` — only the integer-prefixed form is allowed. Decimal numbering belongs INSIDE the file content (`# Tarefa X.0:` heading, subtasks `X.1`, `X.2`), never in the filename.
    - After Step 4.10 (POST-SAVE VERIFICATION), scan `./prds/prd-[feature-slug]/tasks/` and identify any file whose name does NOT match `^[0-9]+_task\.md$`, is not exactly `tasks.md`, and is not an `<integer>_task_review.md` (review files are produced by `do-execute-task`, not by this skill — but never delete them if present from prior runs). For every spurious file (e.g., `1.0_task.md`, `2.0_task.md`, `1_task_login.md`), delete it with `rm` via Bash. Re-list the directory afterwards and confirm only `tasks.md`, `<integer>_task.md`, and any pre-existing `<integer>_task_review.md` remain.
    - If the directory listing still shows any decimal-prefixed task file after deletion, HALT and report the error to the user — do not proceed to Step 5.

**Step 5: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the tasks are generated, if `TaskUpdate` is available (Claude Code only; skip in Copilot and Cursor), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Present all generated files to the user.
3. Await confirmation before any implementation begins.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Are all task files (`tasks/tasks.md` and `tasks/[num]_task.md`) saved correctly?
    - Is the internal task tracking synchronized?
    - Does the task list follow the template structure?

## Output Language
Todos os artefatos gerados (tasks.md, arquivos de task individuais) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Guidelines
- Assume the primary reader is a junior developer — be as clear as possible.
- Group tasks by logical deliverable.
- Make each main task independently completable.
- Define clear scope and deliverables for each task.
- Include tests as subtasks within each main task.
- Do NOT implement anything — focus solely on task listing and detailing.

## Quality Checklist
- [ ] PRD and Tech Spec analyzed.
- [ ] High-level task list approved by user.
- [ ] Task files generated using templates.
- [ ] Each task has unit and integration test subtasks.
- [ ] UI-touching tasks include "Identidade Visual" subsection (only if `project_surface = visual`; otherwise N/A).
- [ ] Files saved to `./prds/prd-[feature-slug]/tasks/`.
- [ ] Results presented to user.

## Error Handling
- If the PRD or Tech Spec is missing, halt and direct the user to the `do-create-prd` or `do-create-techspec` skill.
- If the user rejects the high-level task list, revise based on feedback and re-present for approval.
- If the output directory (`./prds/prd-[feature-slug]/tasks/`) already contains task files, confirm with the user before overwriting.
- If a template file cannot be loaded by your AI tool's skill discovery, report the error and halt — do not generate tasks without the templates.

## References
- Templates: `do-create-tasks/assets/tasks-template.md`, `do-create-tasks/assets/task-template.md`
- PRD: `prds/prd-[feature-slug]/prd.md`
- TechSpec: `prds/prd-[feature-slug]/techspec.md`
- Output: `./prds/prd-[feature-slug]/tasks/tasks.md`, `./prds/prd-[feature-slug]/tasks/[num]_task.md`

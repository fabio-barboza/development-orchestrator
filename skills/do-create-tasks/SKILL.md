---
name: do-create-tasks
description: Converts PBI and Tech Spec into a detailed, sequenced list of implementation tasks. Each task is a functional, incremental deliverable with its own test suite. Outputs tasks.md and individual task files. Use when the user asks to create tasks, break down work, or plan implementation from an existing PBI and Tech Spec. Do not use for PBI creation, tech spec creation, or actual code implementation.
---

# Task Creation

## Role
You are a senior project manager specialized in breaking down features into incremental, independently deliverable tasks.

## Interactive Execution Policy
**This skill is interactive by design.** It requires user approval at Step 3 (high-level task list) before generating files. Do NOT proceed past Step 3 without explicit user confirmation.

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. **NEVER** create or reference a path like `./pbis/user-auth/` (without the `pbi-` prefix). The `tasks/` subdirectory is always inside this prefixed folder: `./pbis/pbi-[feature-slug]/tasks/`.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable (use file paths relative to this skill's location)
3. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Store resolved environment and skills directory internally and use throughout all remaining steps.

**Step 1: Validate Prerequisites**
1. Confirm the feature slug has been provided.
2. Verify the PBI exists at `./pbis/pbi-[feature-slug]/pbi.md`. The directory MUST be `pbi-[feature-slug]` — never `[feature-slug]` alone. If missing, halt.
3. Verify the Tech Spec exists at `./pbis/pbi-[feature-slug]/techspec.md`. If missing, halt.
4. **Path check**: Before creating any file, confirm you are writing to `./pbis/pbi-[feature-slug]/tasks/` — not `./pbis/[feature-slug]/tasks/`.

**Step 2: Analyze PBI and Tech Spec (Mandatory)**
1. Read the PBI completely to extract requirements.
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
1. Read the tasks summary template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-create-tasks/assets/tasks-template.md` for Claude Code).
2. Read the individual task template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-create-tasks/assets/task-template.md` for Claude Code).
3. Create the directory `./pbis/pbi-[feature-slug]/tasks/` if it does not exist.
4. Create the summary file: `./pbis/pbi-[feature-slug]/tasks/tasks.md`.
5. Create individual task files: `./pbis/pbi-[feature-slug]/tasks/[num]_task.md`.
5. Use format X.0 for main tasks, X.Y for subtasks.
6. Do NOT repeat implementation details already in the Tech Spec — reference it instead.

**Step 5: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the tasks are generated, if `TaskUpdate` is available (Claude Code), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Present all generated files to the user.
3. Await confirmation before any implementation begins.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Are all task files (`tasks/tasks.md` and `tasks/[num]_task.md`) saved correctly?
    - Is the internal task tracking synchronized?
    - Does the task list follow the template structure?

## Output Language
All generated artifacts (tasks.md, individual task files) must be written in Brazilian Portuguese (PT-BR). Code examples, variable names, and technical terms remain in English.

## Guidelines
- Assume the primary reader is a junior developer — be as clear as possible.
- Group tasks by logical deliverable.
- Make each main task independently completable.
- Define clear scope and deliverables for each task.
- Include tests as subtasks within each main task.
- Do NOT implement anything — focus solely on task listing and detailing.

## Quality Checklist
- [ ] PBI and Tech Spec analyzed.
- [ ] High-level task list approved by user.
- [ ] Task files generated using templates.
- [ ] Each task has unit and integration test subtasks.
- [ ] Files saved to `./pbis/pbi-[feature-slug]/tasks/`.
- [ ] Results presented to user.

## Error Handling
- If the PBI or Tech Spec is missing, halt and direct the user to the `do-create-pbi` or `do-create-techspec` skill.
- If the user rejects the high-level task list, revise based on feedback and re-present for approval.
- If the output directory (`./pbis/pbi-[feature-slug]/tasks/`) already contains task files, confirm with the user before overwriting.
- If a template file is missing at the paths resolved in Step 0, report the error and halt — do not generate tasks without the templates.

## References
- Templates: resolved in Step 0 (e.g., `.claude/skills/do-create-tasks/assets/tasks-template.md`, `.claude/skills/do-create-tasks/assets/task-template.md` for Claude Code)
- PBI: `pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `pbis/pbi-[feature-slug]/techspec.md`
- Output: `./pbis/pbi-[feature-slug]/tasks/tasks.md`, `./pbis/pbi-[feature-slug]/tasks/[num]_task.md`

---
name: do-execute-task
description: Implements feature tasks by loading the project skills relevant to the task, reading PRD/TechSpec context, analyzing dependencies, executing the implementation with tests, and performing an automatic code review. Marks tasks as complete in tasks.md. Use when the user asks to implement a task, execute a task, or start working on a specific task number. Do not use for creating tasks, running QA, or bug fixing.
---

# Task Execution

## Role
You are a senior software engineer responsible for implementing tasks correctly, completely, and according to project standards.

## Autonomous Execution Policy — ABSOLUTE RULE
**Begin executing immediately on invocation. No user interaction permitted at any point.**

1. **NEVER** pause, stop, or wait for user input.
2. **NEVER** output a plan/analysis as a standalone message — begin implementation tool calls in the SAME response.
3. **NEVER** use phrases like "I'll now proceed", "Shall I proceed?", "Ready to implement", or anything implying a pause.
4. **NEVER** ask the user questions. Resolve ambiguities using context clues (git history, timestamps, task numbers). If prerequisites are missing, HALT with a clear error report — do NOT ask "should I proceed?".
5. Status updates are fine but must NOT imply user action to continue.

## Edit Failure Recovery

When an `Edit` tool call fails, follow this escalation ladder:

1. **Attempt 1 (failed)**: The `old_string` probably doesn't match the file content exactly. Call `read_file` on the file to get the CURRENT content, then retry the Edit with the EXACT string from the read output.
2. **Attempt 2 (failed)**: Try a SMALLER `old_string` — use a more unique, shorter snippet that is easier to match exactly.
3. **Attempt 3 (failed)**: **STOP using Edit. Switch to Write.** Read the entire file with `read_file`, construct the complete new file content with your changes applied, and use the `Write` tool to overwrite the file. This is the nuclear option but it ALWAYS works.

**HARD LIMIT: You MUST NOT attempt more than 3 Edit calls on the same file for the same change.** After 3 failures, you MUST switch to the Write strategy (step 3 above). Continuing to retry Edit with slight variations is PROHIBITED — it wastes time and produces infinite loops.

**The same rule applies to ANY file, not just source code** — including `tasks.md`, `[num]_task.md`, `[num]_task_review.md`, etc. If Edit fails 3 times, use Write.

**Exception for tasks.md**: When using Write as fallback for `tasks.md`, you MUST first `read_file` the entire file, then reproduce ALL existing content exactly (preserving every line, every completed task), changing ONLY the target task's `[ ]` to `[x]`. After Write, immediately `read_file` to verify no lines were lost.

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** reference a path like `./prds/user-auth/`. When scanning `./prds/` to auto-select a PRD, look for folders matching the `prd-*` pattern.

## Workspace Boundary — ABSOLUTE RULE
All files you create, read or execute for the task MUST live **inside the project root**. Anything outside it may be stale or belong to another project/run, and reusing it silently corrupts the work.

1. **NEVER** write, read or execute files in `/tmp`, `/var/tmp`, the home directory, other projects, or any path outside the project root. Exceptions: your AI tool's skill files, installed dependencies (the project's dependency directory, global CLIs), and paths that the task file, the PRD/TechSpec or the user explicitly name (e.g. another project to analyze, a shared library).
2. **Scratch files** (exploratory scripts, generators, ad-hoc checks, intermediate outputs) go ONLY in `./.do-tmp/task-[num]/` (e.g. `./.do-tmp/task-3/`). Create it with `mkdir -p` when first needed. If the project is a git repository and `.gitignore` does not already ignore `.do-tmp` (with or without the trailing slash), append `.do-tmp/` once; never add a second line for it.
3. **Do not reuse scratch files from other tasks or previous runs.** Never list, read or execute anything in `./.do-tmp/` outside the current `task-[num]` folder. If you need a helper again, recreate it.
4. **Tests that validate the task are NOT scratch** — write them in the project's test location defined by the TechSpec / project conventions, so they persist and run in the full suite (Step 4B).
5. Before finishing (Step 8), delete `./.do-tmp/task-[num]/`.
6. **Do not redirect command output to files.** Filter it in the pipe instead (`| tail`, `| grep`); if you really need a file, it goes in the scratch folder. A redirect to `/tmp` or any path outside the project root is a violation even if the file is deleted afterwards.
7. **Evidence that must persist** (screenshots or logs cited in the review) goes in `./prds/prd-[feature-slug]/task-screenshots/`: run `mkdir -p` on it first and ALWAYS pass the full relative path in the screenshot tool's `filename` parameter (e.g. `prds/prd-[feature-slug]/task-screenshots/task-[num]-<description>.png`). Never leave screenshots or other evidence files in the project root; throwaway captures go in the scratch folder.

## Context Compaction
If your context is compacted or summarized in the middle of the task (you can no longer see the text of this SKILL.md in your context), **re-read this SKILL.md and the task file before continuing**. Rules that were only in the discarded context still apply — in particular the Workspace Boundary rule and the Step 8 final gate.

## Procedures

**Step 0: Detect AI Tool Environment (execute silently)**
1. Identify the AI tool you are running in — you know whether you are Claude Code, GitHub Copilot, Cursor AI or Opencode. Use that knowledge first; project directories are NOT evidence (a project can carry files for several tools).
2. Only if you cannot tell, fall back to project markers: `.claude/` → **Claude Code**; `.github/copilot-instructions.md`, `.github/agents/` or `.github/prompts/` (a bare `.github/` with only workflows does NOT indicate Copilot) → **GitHub Copilot**; `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI**; `opencode.json`, `.opencode/` or `AGENTS.md` → **Opencode**. If several match, prefer the tool whose agent/command files are installed and whose tools you actually have.
5. Resolve available tools:
   - **TaskUpdate**: available in Claude Code; in Copilot, Cursor, and Opencode, skip gracefully
   - **Documentation MCP** (capability `documentation`): optional — use it only if discovered per `do-shared/references/do-mcp-discovery-instructions.md`; otherwise use Web Search or the codebase

Skill assets/references are loaded via your AI tool's native skill resolver — do not hard-code paths. Paths like `do-shared/...` are relative to the skills directory (the parent folder of this skill, e.g. `.agents/skills/`), not the project root; glob tools skip hidden directories such as `.agents/`, so read these files by their full path (see `do-shared/references/do-project-context.md`). Store the detected tool and capability flags internally.

**Step 1: Pre-Task Configuration (Mandatory — execute silently, do NOT present results to user)**
1. If the user did not provide the `[feature-slug]`, scan the `./prds/` directory to identify the target PRD folder. **AUTOMATIC SELECTION RULE**: If only one PRD folder exists, use it automatically. If multiple exist, select based on this priority: (a) match with task number mentioned by user, (b) most recently modified folder (check timestamps), (c) alphabetically first. **NEVER ask the user to choose.**
2. If the user said "task 3" or "task 3.0", the canonical filename is `3_task.md`. The naming convention is **strictly** `<integer>_task.md` (e.g., `1_task.md`, `11_task.md`) — never `3.0_task.md`, never with descriptive slug. If the exact `<integer>_task.md` file does not exist, list `./prds/prd-[feature-slug]/tasks/` and HALT with a clear error reporting the actual filenames found — do NOT guess or fall back to a similar name.
3. Read the task definition file at `./prds/prd-[feature-slug]/tasks/[num]_task.md` (where `[num]` is an integer such as `1`, `2`, `11`).
4. Read the PRD at `./prds/prd-[feature-slug]/prd.md` for context. If the PRD file does not exist, **HALT IMMEDIATELY** with a clear error: "PRD file missing. Run `do-create-prd` first." — do NOT ask permission or wait for user response.
5. Read the Tech Spec at `./prds/prd-[feature-slug]/techspec.md` for technical requirements. If the TechSpec file does not exist, **HALT IMMEDIATELY** with a clear error: "TechSpec file missing. Run `do-create-techspec` first." — do NOT ask permission or wait for user response.
6. Read `./prds/prd-[feature-slug]/tasks/tasks.md` to understand the full task list and verify dependencies. If `tasks.md` does not exist, **HALT IMMEDIATELY** with a clear error: "Tasks file missing. Run `do-create-tasks` first." — do NOT ask permission or wait for user response.
7. Identify dependencies from previous tasks and verify they are complete.
8. If the task file contains an optional `<skills>` section listing skills, read those that exist (let your AI tool resolve them) and incorporate their guidance during implementation. Skills that are not installed are skipped without error.

**Step 2: Load Project Skills and MCPs**
1. Identify the technologies and the kind of work involved in the task (UI, API, database, tests, etc.).
2. Follow `do-shared/references/do-project-context.md` (Skills procedure): read the `Available Skills` section of the project configuration file, complete it with the skills your AI tool exposes natively (ignoring `do-*`), pick the ones relevant to this task and read their `SKILL.md`. No specific skill is assumed to exist — if none is installed or relevant, continue without them.
3. If a documentation MCP (capability `documentation`) was discovered, use it to check documentation of the involved frameworks/libraries; otherwise rely on the codebase, the TechSpec or Web Search.
4. Validation and end-to-end tasks follow the same rule: the skills for the technologies being validated are read too. "Consulted indirectly through the TechSpec" does NOT count as reading a skill.
5. Keep the result (skills read / none applicable / none installed) for the review file (Step 7).

**Step 3: Task Analysis (INTERNAL ONLY — do NOT output as standalone message)**
1. Briefly analyze: main objectives, how the task fits the project, dependencies.
2. Parse all `<critical>` tags from the task file and keep them as mandatory constraints for the implementation in Step 4.
3. Define a numbered step-by-step approach plan internally.
4. **TRANSITION RULE: Step 3 and Step 4 are a SINGLE ATOMIC OPERATION. You MUST begin implementation tool calls in the SAME response where you do the analysis. There is NO message boundary between Step 3 and Step 4.**

**Step 4: Implementation (SAME response as Step 3 — zero latency, zero user interaction)**
1. Implementation begins in the same response as analysis. There is no separate "planning" message.
2. Follow all project standards established in the project configuration file (CLAUDE.md, AGENTS.md, .github/copilot-instructions.md, or .cursor/rules/project.mdc), the project rules and the guidance of the skills read in Step 2.
3. Implement solutions without workarounds.
4. Respect ALL `<critical>` tags identified in Step 3 — they are mandatory constraints, not suggestions.
5. As you complete each subtask listed in the task file (X.1, X.2, etc.), mark it as `[x]` in the `[num]_task.md` file.
6. Resolve the project's commands (test suite, static checks, local service start) following `do-shared/references/do-project-commands.md`; never assume a language, package manager or tool.
7. **MCP Discovery & E2E tests — WHEN TO RUN**: Execute the MCP discovery procedure at `do-shared/references/do-mcp-discovery-instructions.md`:
   a. List the MCP servers (names and type only — never print or read the whole configuration file, see the security rule in `do-shared/references/do-mcp-discovery-instructions.md`) from the MCP configuration file for the current AI tool (`.mcp.json` for Claude Code, `.vscode/mcp.json` for GitHub Copilot, `.cursor/mcp.json` for Cursor, `opencode.json` for Opencode) 
   b. Read the capability catalog at `do-shared/references/do-mcp-capabilities.md` and classify each server by what its tools do (never by the server's name).
   c. Build capability map and apply the **capability guard**:
      - Task with a user interface + `ui-testing` MCP available → run UI E2E.
      - Task exposing a service + service-capable MCP available (`api-testing`, `messaging`, `data-store`) → run service E2E via that MCP.
      - UI + service, with both kinds available → run both.
      - Task type + no relevant MCP → skip E2E, continue with unit/integration tests, document gap in review.
   Any code you hand to an MCP tool (a script run in the page, an evaluated expression, a hand-built request) must terminate: bound every wait, never repeat an interrupted call unchanged. See "Chamadas que terminam sempre" in `do-shared/references/do-mcp-capabilities.md`.

   **Scope: smoke check only.** Validate the main happy-path flow of what THIS task delivered (one flow per capability) and nothing else. Do not audit accessibility, test several viewports, sweep error states or hunt bugs in the rest of the feature: that is `do-execute-qa`. If the smoke check finds a defect in what this task delivered, fix it as part of the task; a defect in another task's work goes in the review as an observation, without stopping the queue.
8. **HOW TO RUN E2E tests**: MUST be executed via the appropriate MCP tools, as described for the capability in the catalog — **NEVER via CLI**.
9. **Service readiness — MANDATORY before invoking any MCP/E2E that depends on a running service**: Follow the procedure at `do-shared/references/do-service-readiness.md`. The rule is **check first, reuse if running, start only when needed, never kill running services**. For capabilities that require a running target, probe the service before invoking tools; if it is not running, start it only with the project's own local start command (see `do-shared/references/do-project-commands.md`); for brokers, data stores or other external services, document the gap instead of attempting to start them automatically.
10. **If an MCP is unavailable** (connection error, tools not responding): Follow the "Se indisponível" handling of the capability in the catalog. Continue with unit/integration tests and document the E2E gap in the review. A connection-refused (or similar) error from the *target service* is NOT MCP unavailability — the service went down: bring it back with the service-readiness procedure (item 9) and retry. Never switch the validation method — do not replace the MCP with scripts that drive an automation library directly or with a CLI.

**Step 4B: ALL TESTS MUST PASS — NON-NEGOTIABLE GATE**

Run the tests with the commands from `Project Commands`. Re-run only after a code change or a diagnosed cause; use the single-test command to iterate on one failure and the full-suite command once to confirm. Do not repeat the same run hoping for a different result. Variants of the declared command that only add flags (coverage, filtering, reporters) are allowed.


**A task with failing tests is NOT a completed task. This is an ABSOLUTE, NON-NEGOTIABLE rule.**

1. After implementation, run the FULL test suite (unit, integration, and the project's static checks if it defines any), using the commands resolved in Step 4 item 6.
2. If ALL tests pass → proceed to Step 5.
3. If ANY test fails → **YOU ARE PROHIBITED FROM PROCEEDING TO STEP 5.** You MUST:
   a. Read the failing test output carefully. If the failure is that the project's dependencies are not installed, run the project's dependency install command once (see `do-shared/references/do-project-commands.md`, item 5) and re-run the suite instead of treating it as a code failure.
   b. Diagnose the root cause.
   c. Fix the code (NOT the test, unless the test itself is wrong due to your changes).
   d. Re-run the FULL test suite.
   e. Repeat until ALL tests pass.
4. **Loop protection**: Track fix attempts. After **5 failed fix-and-retest cycles**, STOP and report to the user:
   - Which tests are still failing and why.
   - What you tried in each cycle.
   - Your best assessment of the root cause.
   - **DO NOT mark the task as complete. DO NOT update tasks.md. DO NOT create the review file.**
   - The task status remains **incomplete** until the user provides guidance and tests pass.
5. **UNDER NO CIRCUMSTANCES may you:**
   - Skip failing tests and mark the task as complete.
   - Delete or disable failing tests to make the suite pass.
   - Mark the task as "complete with known test failures".
   - Proceed to Step 5 with any test in a failing state.
   - Consider a task "done" if tests were never run.

**Step 5: Mark Task Complete in tasks.md (MANDATORY — ONLY AFTER ALL TESTS PASS)**
1. **PRE-CONDITION**: ALL tests (unit, integration, typecheck) MUST have passed in Step 4B. If they haven't, you are PROHIBITED from executing this step.
2. After successful implementation and ALL tests passing, you MUST update `tasks.md` to mark the task as complete.
3. Change the task status from `[ ]` (or equivalent) to `[x]` (or equivalent "Concluída"/"Done") in `tasks.md`.
4. **IMMEDIATE VERIFICATION (MANDATORY)**: Right after the edit tool call, you MUST call `read_file` on `tasks.md` in the SAME response to verify the `[x]` is actually present in the file content. If the `[x]` is NOT visible in the read output, the edit FAILED — redo it.
5. Mark all subtasks (X.1, X.2, etc.) as `[x]` in the `[num]_task.md` file, and also the checkboxes of the task's test checklist (e.g. the "Testes da Tarefa" section) for every test that was actually executed and passed. Then call `read_file` on `[num]_task.md` to verify.
6. **SYNC INTERNAL PROGRESS**: If `TaskUpdate` is available (Claude Code only; skip in Copilot and Cursor), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.

**TASKS.MD PROTECTION RULE — ABSOLUTE, NON-NEGOTIABLE:**
The ONLY permitted modification to `tasks.md` is changing `[ ]` to `[x]` for the CURRENT task being executed. Everything else in the file is READ-ONLY. Specifically, you are **PROHIBITED** from:
- Deleting ANY line from `tasks.md` (including previously completed tasks)
- Removing, rewriting, or reformatting existing entries
- Changing the status of ANY task other than the current one
- Rewriting the file with `Write` tool — you MUST use the `Edit` tool with a **minimal, surgical replacement** (`old_string` → `new_string`) that ONLY changes `[ ]` to `[x]` for the target task line
- Using `Write` to overwrite `tasks.md` with a "cleaned up" or "reorganized" version

**HOW TO EDIT tasks.md SAFELY:**
- ALWAYS start with the `Edit` tool (prefer over `Write`)
- The `old_string` must be the EXACT task line with `[ ]`
- The `new_string` must be the SAME line with `[x]`
- Nothing else changes. No other lines are touched.
- After the edit, `read_file` the entire file and confirm ALL previously completed tasks are still present and marked as `[x]`.
- If Edit fails after 3 attempts, the **Edit Failure Recovery** rule applies: switch to `Write` using the tasks.md exception (read full file, reproduce ALL content, change ONLY `[ ]` to `[x]` for target task, then verify with `read_file`).

**ANTI-HALLUCINATION RULE FOR STEP 5**: You MUST NOT claim you updated a file unless the tool call (Edit/Write) was actually executed AND the subsequent `read_file` confirms the change. "I updated tasks.md" without a preceding successful Edit tool call and a confirming Read tool call is a LIE. If you are unsure whether the edit happened, re-read the file to check.

**Step 6: Code Review**
1. Re-read the task file to ensure you have the latest content (context compression may have discarded earlier reads).
2. Check if the project is a git repository by running `git rev-parse --is-inside-work-tree`. If git is available, use `git diff` and `git log` to identify files changed as part of this task and read the full context of modified files, not just the diffs. If git is NOT available, manually list all files you created or modified during implementation and read their full content for review.
3. Read the `code-standards.md` file at `do-execute-task/references/code-standards.md`. Review the code against those criteria and verify compliance with the project configuration file if it exists.
4. For each issue found, classify as:
   - **CRÍTICO**: Bugs, problemas de segurança, funcionalidade quebrada, tratamento de erros ausente.
   - **MAIOR**: Violações de padrão de código, testes ausentes, nomenclatura incorreta.
   - **MENOR**: Sugestões de estilo, melhorias menores.
   - **POSITIVO**: Coisas bem feitas que merecem reconhecimento.
5. Run the test suite and, if the project defines them, its static checks (commands from Step 4 item 6). Include any failures as critical issues.
6. Address any issues identified. **Iteration limit**: You may perform a maximum of 3 fix-and-review cycles. If critical issues persist after 3 cycles, mark as **MUDANÇAS SOLICITADAS** in Step 7.

**Step 7: Create Review File (Mandatory)**

1. Read the template at `do-execute-task/assets/review-artifact-template.md`. Fill its `Skills e MCPs do Projeto` section with the record defined in `do-shared/references/do-project-context.md` (skills read in Step 2, MCPs used or gaps — "nenhuma instalada" is a valid value).
2. Determine the review status based on Step 6 findings:
   - **APROVADO**: Sem problemas críticos/maiores.
   - **APROVADO COM OBSERVAÇÕES**: Sem críticos, problemas menores ou poucos maiores não bloqueantes.
   - **MUDANÇAS SOLICITADAS**: Problemas críticos ou múltiplos problemas maiores.
3. **CREATE THE FILE**: Use the `Write` tool to create `[num]_task_review.md` in `./prds/prd-[feature-slug]/tasks/`. This is the MOST IMPORTANT action in this step.
   **MANDATORY FILENAME RULE — ABSOLUTE, NON-NEGOTIABLE:**
   - The filename MUST be exactly `<integer>_task_review.md` matching the integer of the task being reviewed (e.g., task `11_task.md` → review `11_task_review.md`).
   - **PROHIBITED**: descriptive slugs (`11_descriptive_name_review.md` ❌), decimal (`11.0_task_review.md` ❌), variants (`review_11.md`, `11_review.md` ❌).
4. **VERIFY THE FILE EXISTS**: Immediately after the Write tool call, call `read_file` on `./prds/prd-[feature-slug]/tasks/[num]_task_review.md`. You MUST see the file content returned. If the read fails or returns empty → **the Write FAILED. Redo it NOW.**
5. If the `read_file` succeeds, the file is confirmed created. Proceed to Step 8.

**FAILURE RECOVERY**: If the Write tool call is denied by the user or fails for any reason:
- Try again with the Write tool.
- If denied again, inform the user that the review file could NOT be created and the task CANNOT be considered complete without it.
- **DO NOT proceed to Step 8. DO NOT report the task as complete.**

**ANTI-HALLUCINATION RULE**: "Review file created" without a successful `Write` tool call followed by a successful `read_file` showing content is a LIE. You MUST have BOTH tool calls executed successfully.

**Step 8: Final Gate — Mandatory Artifact Verification**

**Every check requires an actual tool call. Do NOT rely on memory — re-read files from disk. Fix any failure before sending the final response.**

Perform ALL checks below. If ANY fails, fix it first.

1. **CHECK 1 — All tests pass**: Confirm that the last test run had ALL tests passing (zero failures). If tests were not run or any test failed → **STOP. Go back to Step 4B. You CANNOT mark the task as complete with failing tests.**

2. **CHECK 2 — Review file exists**: Call `read_file` on `./prds/prd-[feature-slug]/tasks/[num]_task_review.md`. You MUST see actual file content returned by the tool. If the tool returns an error or the file does not exist → **STOP. Go back to Step 7 and create it NOW using the Write tool. Then call `read_file` again to confirm it exists. Do NOT proceed until this check passes.**

3. **CHECK 3 — tasks.md is updated AND intact**: Call `read_file` on `./prds/prd-[feature-slug]/tasks/tasks.md`. Verify TWO things:
   - (a) The current task is marked as `[x]`. If not → **STOP. Edit it NOW.**
   - (b) **ALL other tasks that were previously in the file are STILL PRESENT.** No lines were deleted, no entries were removed. If any previously existing task entry is missing → **STOP. This is a critical error — you destroyed data. Restore the missing entries using git or by re-adding them manually, then re-read to confirm.**

4. **CHECK 4 — Subtasks are marked**: Call `read_file` on `./prds/prd-[feature-slug]/tasks/[num]_task.md`. Scan ALL checkboxes in the file — the subtasks (X.1, X.2, etc.) AND the test checklist (e.g. the "Testes da Tarefa" section). Every single one MUST show `[x]`; a test item may only be marked after that test was actually executed and passed. If any shows `[ ]` → **STOP. Edit the file NOW. Then call `read_file` again to confirm.**

5. **CHECK 5 — Requirements cross-check**: List all requirements from the task file and confirm each is implemented and tested.

6. **CHECK 6 — Critical tags**: Verify all `<critical>` tags from the task file were satisfied.

7. **CHECK 7 — Project skills and MCPs registered**: the review file read in CHECK 2 contains the `Skills e MCPs do Projeto` section filled in (skills read, or "nenhuma aplicável"/"nenhuma instalada"). This is a record of the Step 2 consultation, not an obligation to use a skill. If the section is missing → add it NOW and re-read the file.

8. **CHECK 8 — Workspace boundary**: Confirm no file was created outside the project root during this task (Workspace Boundary rule). Remove `./.do-tmp/task-[num]/` (`rm -rf ./.do-tmp/task-[num]`) and confirm it no longer exists. Review every command you ran in this task: any redirect, `tee`, `-o`/`--output` option or script that wrote outside the project root (for example `/tmp`) is a violation. Delete those files now (they are yours) and note the violation in the review. If any task artifact ended up outside the project, move what must persist (e.g. tests) into the project and delete the rest. Also confirm that no screenshot or evidence file was left in the project root — move the ones that must persist to `./prds/prd-[feature-slug]/task-screenshots/` and delete the rest.

**BLOCKING RULE: If Check 1, 2, 3, 4, 7, or 8 fails, you are PROHIBITED from sending a final response. Fix the issue and re-run the checks. A task with failing tests is NEVER complete.**

**ANTI-HALLUCINATION ENFORCEMENT**: If your final response includes ✅ for an artifact but the corresponding `read_file` tool call in Step 7 was never made or returned an error, you are LYING to the user. This is the worst possible outcome. When in doubt, re-read the file.

9. **FINAL OUTPUT MANIFEST** (include in your final response to the user):
   ```
   📋 Artefatos:
   - Testes: ✅ Todos passando
   - [num]_task_review.md: ✅ Criado ([STATUS])
   - tasks.md: ✅ Atualizado (task [num] marcada como concluída)
   - [num]_task.md: ✅ Subtasks marcadas como concluídas
   - .do-tmp/task-[num]: ✅ Removido (nada criado fora do projeto)
   ```
   If any artifact shows ❌ instead of ✅, you have violated this skill's rules. Do NOT send the response — fix the artifact first.
   **Subagent exception:** when you were invoked by an orchestrator as a subagent (e.g. `agent-execute-task`), do NOT include this manifest — your final response is ONLY the short `TASK <ID>: ...` block defined in `do-execute-task/references/subagent-mode.md`.

## Error Handling
- If the task file does not exist, halt and report to the user.
- If the PRD file does not exist, halt and direct the user to run `do-create-prd`.
- If the TechSpec file does not exist, halt and direct the user to run `do-create-techspec`.
- If `tasks.md` does not exist, halt and direct the user to run `do-create-tasks`.
- If dependencies are not complete, warn the user in a status message and proceed anyway — do NOT wait for confirmation.
- If tests fail, fix the issues and re-run (up to 5 cycles). NEVER mark the task as complete with failing tests. If stuck after 5 cycles, report to the user and leave the task as incomplete.
- If the review identifies critical issues, address them (up to 3 fix cycles) before finalizing.
- If a service required by an MCP is not running (app, broker, etc.), apply `do-shared/references/do-service-readiness.md` — probe first, reuse if running, start only safe dev servers, document gaps for external services. Never kill a running service.
- If an MCP is unavailable, follow its "Se indisponivel" handling from the registry. Continue with unit/integration tests and document the E2E gap in the review.
- If git is not initialized, skip git-based diff analysis and manually track changed files.
- If implementation fails mid-way (compilation errors, incompatible dependencies, etc.), document what was completed and what remains, ensure the codebase compiles (revert broken partial changes if needed), and report the blocker to the user.
- If an Edit tool call fails, follow the **Edit Failure Recovery** escalation ladder (see above): retry with exact content from read_file → try smaller old_string → switch to Write after 3 failures. NEVER retry Edit more than 3 times for the same change.

## Output Language
Todos os artefatos gerados (incluindo o arquivo de review) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## References
- Task: `./prds/prd-[feature-slug]/tasks/[num]_task.md`
- PRD: `./prds/prd-[feature-slug]/prd.md`
- TechSpec: `./prds/prd-[feature-slug]/techspec.md`
- Tasks: `./prds/prd-[feature-slug]/tasks/tasks.md`
- Review template: `do-execute-task/assets/review-artifact-template.md`
- Code standards: `do-execute-task/references/code-standards.md`
- Project Skills and MCPs: `do-shared/references/do-project-context.md`
- MCP Discovery: `do-shared/references/do-mcp-discovery-instructions.md`
- MCP Registry: `do-shared/references/do-mcp-capabilities.md`
- Service Readiness: `do-shared/references/do-service-readiness.md`
- Review output: `[num]_task_review.md` (same directory as the task file)

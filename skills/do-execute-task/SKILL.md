---
name: do-execute-task
description: Implements feature tasks by loading required skills, reading PBI/TechSpec context, analyzing dependencies, executing the implementation with tests, and performing an automatic code review. Marks tasks as complete in tasks.md. Use when the user asks to implement a task, execute a task, or start working on a specific task number. Do not use for creating tasks, running QA, or bug fixing.
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

## Procedures

**Step 0: Detect AI Tool Environment (execute silently)**
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable
3. Resolve available tools:
   - **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Store resolved environment and skills directory internally.

**Step 1: Pre-Task Configuration (Mandatory — execute silently, do NOT present results to user)**
1. If the user did not provide the `[feature-slug]`, scan the `./pbis/` directory to identify the target PBI folder. **AUTOMATIC SELECTION RULE**: If only one PBI folder exists, use it automatically. If multiple exist, select based on this priority: (a) match with task number mentioned by user, (b) most recently modified folder (check timestamps), (c) alphabetically first. **NEVER ask the user to choose.**
2. If the user said "task 3", interpret it as `3.0_task.md`. If the file doesn't exist, try `3_task.md` as a fallback.
3. Read the task definition file at `./pbis/pbi-[feature-slug]/tasks/[num]_task.md`.
4. Read the PBI at `./pbis/pbi-[feature-slug]/pbi.md` for context. If the PBI file does not exist, **HALT IMMEDIATELY** with a clear error: "PBI file missing. Run `do-create-pbi` first." — do NOT ask permission or wait for user response.
5. Read the Tech Spec at `./pbis/pbi-[feature-slug]/techspec.md` for technical requirements. If the TechSpec file does not exist, **HALT IMMEDIATELY** with a clear error: "TechSpec file missing. Run `do-create-techspec` first." — do NOT ask permission or wait for user response.
6. Read `./pbis/pbi-[feature-slug]/tasks/tasks.md` to understand the full task list and verify dependencies. If `tasks.md` does not exist, **HALT IMMEDIATELY** with a clear error: "Tasks file missing. Run `do-create-tasks` first." — do NOT ask permission or wait for user response.
7. Identify dependencies from previous tasks and verify they are complete.
8. If the task file contains a `<skills>` section listing relevant skills, read those skill files from the skills directory resolved in Step 0 and incorporate their guidance during implementation.

**Step 2: Load Required Skills**
1. Identify the technologies involved in the task.
2. Use Context7 MCP to check documentation of involved frameworks/libraries if needed.

**Step 3: Task Analysis (INTERNAL ONLY — do NOT output as standalone message)**
1. Briefly analyze: main objectives, how the task fits the project, dependencies.
2. Parse all `<critical>` tags from the task file and keep them as mandatory constraints for the implementation in Step 4.
3. Define a numbered step-by-step approach plan internally.
4. **TRANSITION RULE: Step 3 and Step 4 are a SINGLE ATOMIC OPERATION. You MUST begin implementation tool calls in the SAME response where you do the analysis. There is NO message boundary between Step 3 and Step 4.**

**Step 4: Implementation (SAME response as Step 3 — zero latency, zero user interaction)**
1. Implementation begins in the same response as analysis. There is no separate "planning" message.
2. Follow all project standards established in the project configuration file (CLAUDE.md or equivalent) and project rules.
3. Implement solutions without workarounds.
4. Respect ALL `<critical>` tags identified in Step 3 — they are mandatory constraints, not suggestions.
5. As you complete each subtask listed in the task file (X.1, X.2, etc.), mark it as `[x]` in the `[num]_task.md` file.
6. Detect the project's package manager from lock files (`bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm, default: `npm`).
7. **MCP Discovery & E2E tests — WHEN TO RUN**: Execute the MCP discovery procedure from the shared skills directory resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code):
   a. Read the MCP configuration file for the current AI tool (`.mcp.json` for Claude Code, `.vscode/mcp.json` for GitHub Copilot, `.cursor/mcp.json` for Cursor) to list configured MCP servers.
   b. Read the MCP capabilities file from the shared skills directory resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-capabilities.md` for Claude Code) to map each server to capabilities and tools.
   c. Build capability map and apply the **capability guard**:
      - Frontend task + `browser-testing` MCP available → run browser E2E.
      - Backend task + backend-capable MCP available (`message-queue`, `database`, `cache`, `api-testing`) → run backend E2E via that MCP.
      - Frontend + Backend + both available → run both.
      - Task type + no relevant MCP → skip E2E, continue with unit/integration tests, document gap in review.
8. **HOW TO RUN E2E tests**: MUST be executed via the appropriate MCP tools as listed in the capability registry — **NEVER via CLI**. Use the tools described in each MCP's registry entry. For MCPs that require a running service (check "Requer app rodando" in registry), verify the service is accessible before invoking tools. If not running, attempt to start only the dev server using known-safe commands (`npm run dev`, `npm start`, `bun dev`, `pnpm dev`) — for brokers or external services, document the gap instead of attempting to start them automatically.
9. **If an MCP is unavailable** (connection error, tools not responding): Follow the "Se indisponivel" handling from the MCP's registry entry. Continue with unit/integration tests and document the E2E gap in the review.

**Step 4B: ALL TESTS MUST PASS — NON-NEGOTIABLE GATE**

**A task with failing tests is NOT a completed task. This is an ABSOLUTE, NON-NEGOTIABLE rule.**

1. After implementation, run the FULL test suite (unit, integration, and typecheck if available).
2. If ALL tests pass → proceed to Step 5.
3. If ANY test fails → **YOU ARE PROHIBITED FROM PROCEEDING TO STEP 5.** You MUST:
   a. Read the failing test output carefully.
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
5. Mark all subtasks (X.1, X.2, etc.) as `[x]` in the `[num]_task.md` file. Then call `read_file` on `[num]_task.md` to verify.
6. **SYNC INTERNAL PROGRESS**: If `TaskUpdate` is available (Claude Code), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.

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
3. Read the `code-standards.md` file from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-task/references/code-standards.md` for Claude Code). Review the code against those criteria and verify compliance with the project configuration file (CLAUDE.md or equivalent) if it exists.
4. For each issue found, classify as:
   - **CRITICAL**: Bugs, security issues, broken functionality, missing error handling.
   - **MAJOR**: Code standard violations, missing tests, bad naming.
   - **MINOR**: Style suggestions, minor improvements.
   - **POSITIVE**: Well-done things worth recognizing.
5. Run the test suite using the detected package manager. If a `typecheck` script exists in `package.json`, run it. Include any failures as critical issues.
6. Address any issues identified. **Iteration limit**: You may perform a maximum of 3 fix-and-review cycles. If critical issues persist after 3 cycles, mark as **CHANGES REQUESTED** in Step 7.

**Step 7: Create Review File (Mandatory)**

1. Read the template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-task/assets/review-artifact-template.md` for Claude Code).
2. Determine the review status based on Step 6 findings:
   - **APPROVED**: No critical/major issues.
   - **APPROVED WITH OBSERVATIONS**: No critical, minor or few non-blocking major issues.
   - **CHANGES REQUESTED**: Critical issues or multiple major issues.
3. **CREATE THE FILE**: Use the `Write` tool to create `[num]_task_review.md` in `./pbis/pbi-[feature-slug]/`. This is the MOST IMPORTANT action in this step.
4. **VERIFY THE FILE EXISTS**: Immediately after the Write tool call, call `read_file` on `./pbis/pbi-[feature-slug]/tasks/[num]_task_review.md`. You MUST see the file content returned. If the read fails or returns empty → **the Write FAILED. Redo it NOW.**
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

2. **CHECK 2 — Review file exists**: Call `read_file` on `./pbis/pbi-[feature-slug]/tasks/[num]_task_review.md`. You MUST see actual file content returned by the tool. If the tool returns an error or the file does not exist → **STOP. Go back to Step 7 and create it NOW using the Write tool. Then call `read_file` again to confirm it exists. Do NOT proceed until this check passes.**

3. **CHECK 3 — tasks.md is updated AND intact**: Call `read_file` on `./pbis/pbi-[feature-slug]/tasks/tasks.md`. Verify TWO things:
   - (a) The current task is marked as `[x]`. If not → **STOP. Edit it NOW.**
   - (b) **ALL other tasks that were previously in the file are STILL PRESENT.** No lines were deleted, no entries were removed. If any previously existing task entry is missing → **STOP. This is a critical error — you destroyed data. Restore the missing entries using git or by re-adding them manually, then re-read to confirm.**

4. **CHECK 4 — Subtasks are marked**: Call `read_file` on `./pbis/pbi-[feature-slug]/tasks/[num]_task.md`. Scan ALL subtask checkboxes (X.1, X.2, etc.). Every single one MUST show `[x]`. If any shows `[ ]` → **STOP. Edit the file NOW. Then call `read_file` again to confirm.**

5. **CHECK 5 — Requirements cross-check**: List all requirements from the task file and confirm each is implemented and tested.

6. **CHECK 6 — Critical tags**: Verify all `<critical>` tags from the task file were satisfied.

**BLOCKING RULE: If Check 1, 2, 3, or 4 fails, you are PROHIBITED from sending a final response. Fix the issue and re-run the checks. A task with failing tests is NEVER complete.**

**ANTI-HALLUCINATION ENFORCEMENT**: If your final response includes ✅ for an artifact but the corresponding `read_file` tool call in Step 7 was never made or returned an error, you are LYING to the user. This is the worst possible outcome. When in doubt, re-read the file.

7. **FINAL OUTPUT MANIFEST** (include in your final response to the user):
   ```
   📋 Artefatos:
   - Testes: ✅ Todos passando
   - [num]_task_review.md: ✅ Criado ([STATUS])
   - tasks.md: ✅ Atualizado (task [num] marcada como concluída)
   - [num]_task.md: ✅ Subtasks marcadas como concluídas
   ```
   If any artifact shows ❌ instead of ✅, you have violated this skill's rules. Do NOT send the response — fix the artifact first.

## Error Handling
- If the task file does not exist, halt and report to the user.
- If the PBI file does not exist, halt and direct the user to run `do-create-pbi`.
- If the TechSpec file does not exist, halt and direct the user to run `do-create-techspec`.
- If `tasks.md` does not exist, halt and direct the user to run `do-create-tasks`.
- If dependencies are not complete, warn the user in a status message and proceed anyway — do NOT wait for confirmation.
- If tests fail, fix the issues and re-run (up to 5 cycles). NEVER mark the task as complete with failing tests. If stuck after 5 cycles, report to the user and leave the task as incomplete.
- If the review identifies critical issues, address them (up to 3 fix cycles) before finalizing.
- If a service required by an MCP is not running (app, broker, etc.), start it or document the gap.
- If an MCP is unavailable, follow its "Se indisponivel" handling from the registry. Continue with unit/integration tests and document the E2E gap in the review.
- If git is not initialized, skip git-based diff analysis and manually track changed files.
- If implementation fails mid-way (compilation errors, incompatible dependencies, etc.), document what was completed and what remains, ensure the codebase compiles (revert broken partial changes if needed), and report the blocker to the user.
- If an Edit tool call fails, follow the **Edit Failure Recovery** escalation ladder (see above): retry with exact content from read_file → try smaller old_string → switch to Write after 3 failures. NEVER retry Edit more than 3 times for the same change.

## Output Language
All generated artifacts (including the review file) must be written in Brazilian Portuguese (PT-BR). Code examples, variable names, and technical terms remain in English.

## References
- Task: `./pbis/pbi-[feature-slug]/tasks/[num]_task.md`
- PBI: `./pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `./pbis/pbi-[feature-slug]/techspec.md`
- Tasks: `./pbis/pbi-[feature-slug]/tasks/tasks.md`
- Review template: resolved in Step 0 (e.g., `.claude/skills/do-execute-task/assets/review-artifact-template.md` for Claude Code)
- Code standards: resolved in Step 0 (e.g., `.claude/skills/do-execute-task/references/code-standards.md` for Claude Code)
- MCP Discovery: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code)
- MCP Registry: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-capabilities.md` for Claude Code)
- Review output: `[num]_task_review.md` (same directory as the task file)

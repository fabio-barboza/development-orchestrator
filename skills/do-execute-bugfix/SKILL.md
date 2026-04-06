---
name: do-execute-bugfix
description: Reads documented bugs from bugs.md, analyzes root causes, implements fixes with regression tests, and validates the full test suite. Prioritizes fixes by severity (high to low). Updates bugs.md with correction status and generates a final bugfix report. Use when the user asks to fix bugs, resolve issues, or run the bugfix workflow for a feature. Do not use for new feature implementation, code review, or QA testing.
---

# Bug Fix Execution

## Role
You are a senior software engineer specialized in root-cause analysis and implementing robust, regression-tested bug fixes.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reason to stop and ask the user is when there is a genuine doubt or ambiguity that cannot be resolved by reading the project files. Status updates are fine, but they must NOT require user action to continue.

## Procedures

**Step 1: Context Analysis (Mandatory)**
1. Read the bugs file at `./pbis/pbi-[feature-slug]/bugs.md` and extract ALL documented bugs. If `bugs.md` does not exist, halt and report to the user — there are no bugs to fix.
2. Read the PBI at `./pbis/pbi-[feature-slug]/pbi.md` to understand affected requirements. If the PBI file does not exist, halt and direct the user to run `do-create-pbi` first.
3. Read the Tech Spec at `./pbis/pbi-[feature-slug]/techspec.md` to understand relevant technical decisions. If the TechSpec file does not exist, halt and direct the user to run `do-create-techspec` first.
4. Review project rules for compliance in fixes.
5. Do NOT skip this step — full context understanding is fundamental for quality fixes.

**Step 2: Plan Fixes (Mandatory)**
1. For each bug, generate a planning summary:
   - Bug ID, Severity (High/Medium/Low), Affected Component.
   - Root Cause analysis.
   - Files to modify.
   - Fix strategy description.
   - Planned regression tests (unit, integration, E2E).
2. Use Context7 MCP to analyze documentation of involved languages, frameworks, and libraries.
3. **DO NOT stop here. DO NOT present the plan and wait for approval. Proceed IMMEDIATELY to Step 3.**

**Step 3: Implement Fixes (starts immediately after Step 2 — no pause, no confirmation)**
1. Detect the project's package manager from lock files (`bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm). Default to `npm` if none found.
2. Fix bugs in severity order: High first, then Medium, then Low.
3. **Iteration limit**: You may perform a maximum of 3 fix-and-test cycles per bug. If a bug persists after 3 cycles, document the remaining issue in `bugs.md` with status "Unresolved" and report the blocker to the user.
4. For each bug follow this sequence:
   a. Locate and read the affected code.
   b. Reason about the flow causing the bug.
   c. Implement the root-cause fix — no superficial workarounds.
   d. If a `typecheck` script exists in `package.json`, run it after each fix.
   e. Run existing tests to ensure no regressions.

**Edit Failure Recovery**: When an `Edit` tool call fails, follow this escalation: (1) `read_file` to get current content, retry Edit with exact string. (2) Try a smaller, more unique `old_string`. (3) After 3 failed Edit attempts on the same file, switch to `Write` (read full file, apply changes, overwrite). **HARD LIMIT: max 3 Edit retries per file per change.**

**Step 4: Create Regression Tests (Mandatory)**
1. For each fixed bug, create tests that:
   - Simulate the original bug scenario (test must fail if the fix is reverted).
   - Validate the correct behavior with the fix applied.
   - Cover related edge cases.
2. Choose test type based on bug nature:
   - **Unit test**: Bug in isolated function/method logic.
   - **Integration test**: Bug in module communication (e.g., controller + service).
   - **E2E test**: Bug visible in the UI or full flow.

**Step 5: MCP Discovery & Validation (Mandatory for bugs that can be validated via MCP)**
1. Execute the MCP discovery procedure from `.claude/skills/do-shared/do-mcp-discovery-instructions.md`:
   a. Read the MCP configuration file for the current AI tool (`.mcp.json` for Claude Code, `.vscode/mcp.json` for GitHub Copilot, `.cursor/mcp.json` for Cursor) to list configured MCP servers.
   b. Read `.claude/skills/do-shared/do-mcp-capabilities.md` to map each server to capabilities.
   c. Build capability map and apply the capability guard.
2. For bugs affecting the **UI** (and `browser-testing` MCP available):
   a. Verify the app is running (check "Requer app rodando" in registry). Start if needed.
   b. If MCP unavailable: follow "Se indisponivel" handling from registry. Document gap in bugfix report.
   c. Navigate to the application, snapshot page state, reproduce the bug flow, capture screenshot evidence of the fix.
3. For bugs affecting **backend behavior** (and backend-capable MCP available — `message-queue`, `database`, `cache`, `api-testing`):
   a. Verify the required service is accessible. Document gap if unavailable.
   b. Use the MCP tools to validate the fix end-to-end (e.g., verify messages are published/consumed correctly, verify data integrity).
4. If no MCP with relevant capability exists for the bug type: document the validation gap in the bugfix report, rely on unit/integration tests only.

**Step 6: Final Test Execution (Mandatory)**
1. Run ALL project tests using the package manager detected in Step 3 (e.g., `npm test`).
2. **E2E tests via MCP**: Only run E2E tests when a relevant MCP is available for the bug type (per the capability guard from Step 5). Use the MCP tools listed in the registry — **NEVER via CLI**.
4. If a `typecheck` script exists in `package.json`, run it.
5. Verify ALL pass with 100% success.
6. The task is NOT complete if any test fails.

**Step 7: Update bugs.md (Mandatory)**
1. For each fixed bug, append to its entry:
   - **Status:** Fixed.
   - **Applied fix:** Brief description.
   - **Regression tests:** List of created tests.
2. For any unresolved bugs, update status to "Unresolved" with a description of the blocker.

**Step 8: Generate Final Report (Mandatory)**
1. Read the report template at `.claude/skills/do-execute-bugfix/assets/bugfix-report-template.md`.
2. Fill in all sections with actual results.
3. Save the report to `./pbis/pbi-[feature-slug]/bugfix-report.md`.

**Step 9: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once all bugs are fixed and the report is generated, use the `TaskUpdate` tool to mark all corresponding items in your internal task tracking as `completed`.
2. Provide the final bugfix report to the user.
3. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is `bugs.md` updated with the status "Fixed" (or "Unresolved")?
    - Is the final report generated?
    - Did all regression tests pass?

## Output Language
All generated artifacts (bugfix report, status updates in bugs.md) must be written in Brazilian Portuguese (PT-BR). Code examples, variable names, and technical terms remain in English.

## Error Handling
- If `bugs.md` does not exist, halt and report to the user.
- If the PBI file does not exist, halt and direct the user to run `do-create-pbi`.
- If the TechSpec file does not exist, halt and direct the user to run `do-create-techspec`.
- If a bug requires significant architectural changes, document the justification before proceeding.
- If new bugs are discovered during fixes, document them in `bugs.md` with status "New" but do NOT fix them in this cycle — they will be addressed in the next bugfix run.
- If a service required by an MCP is not running (app, broker, etc.), start it or document the gap.
- If an MCP is unavailable, follow its "Se indisponivel" handling from the registry and document the gap in the bugfix report.
- If implementation fails mid-way (compilation errors, incompatible dependencies), document what was completed and what remains, ensure the codebase compiles (revert broken partial changes if needed), and report the blocker to the user.

## References
- Bugs: `./pbis/pbi-[feature-slug]/bugs.md`
- Template: `.claude/skills/do-execute-bugfix/assets/bugfix-report-template.md`
- MCP Discovery: `.claude/skills/do-shared/do-mcp-discovery-instructions.md`
- MCP Registry: `.claude/skills/do-shared/do-mcp-capabilities.md`
- PBI: `./pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `./pbis/pbi-[feature-slug]/techspec.md`
- Bugfix Report output: `./pbis/pbi-[feature-slug]/bugfix-report.md`

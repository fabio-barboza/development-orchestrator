---
name: do-execute-review
description: Performs comprehensive code review by analyzing git diff, verifying conformance with project rules, validating test suites, and checking adherence to Tech Spec and Tasks. Generates a structured code review report with severity-classified findings. Use when the user asks for a code review, wants to validate code quality, or needs pre-merge verification. Do not use for QA testing, bug fixing, or task implementation.
---

# Code Review Execution

## Role
You are a senior code reviewer focused on quality, standards compliance, and providing constructive, actionable feedback. You review and report — you do NOT implement fixes.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reason to stop and ask the user is when there is a genuine doubt or ambiguity that cannot be resolved by reading the project files. Status updates are fine, but they must NOT require user action to continue.

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. **NEVER** reference a path like `./pbis/user-auth/`.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable (use file paths relative to this skill's location)
3. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Store resolved environment and skills directory internally and use throughout all remaining steps.

**Step 1: Documentation Analysis (Mandatory)**
1. Read the PBI at `./pbis/pbi-[feature-slug]/pbi.md` to understand the feature requirements and expected outcomes. If the PBI file does not exist, warn in the review report and continue.
2. Read the Tech Spec at `./pbis/pbi-[feature-slug]/techspec.md` to understand expected architectural decisions. If the TechSpec file does not exist, warn in the review report that the review was performed without a TechSpec reference and continue.
3. Read the Tasks at `./pbis/pbi-[feature-slug]/tasks/tasks.md` to verify the scope implemented. If `tasks.md` does not exist, warn in the review report and continue.
4. Read the project rules to know the required standards.

**Step 2: Code Change Analysis (Mandatory)**
1. Check if the project is a git repository by running `git rev-parse --is-inside-work-tree`.
2. **If git is available**, run git commands to understand what changed:
   - `git status` to see modified files.
   - `git diff` and `git diff --staged` to see all changes.
   - `git log main..HEAD --oneline` to see branch commits.
   - `git diff main...HEAD` for the full branch diff.
3. **If git is NOT available**, ask the user which files to review, or scan the project for recently modified files using `ls -lt`. Document in the review report that git was unavailable.
4. For each modified file:
   a. Analyze changes line by line.
   b. Verify adherence to project standards.
   c. Identify potential issues.
5. Read the full context of modified files, not just the diff.

**Step 3: Rules Conformance Verification (Mandatory)**
1. For each code change, verify:
   - Naming conventions per project rules.
   - Project folder structure adherence.
   - Code standards (formatting, linting).
   - No unauthorized dependencies introduced.
   - Error handling patterns.
   - Language conventions (Portuguese/English as defined).

**Step 4: Tech Spec Adherence Verification (Mandatory)**
1. Compare implementation against the Tech Spec (if available):
   - Architecture implemented as specified.
   - Components created as defined.
   - Interfaces and contracts follow specification.
   - Data models as documented.
   - Endpoints/APIs as specified.
   - Integrations implemented correctly.
2. For each deviation found, classify as:
   - **NÃO-ADERENTE**: Difere sem justificativa técnica → registrar como problema MAIOR.
   - **DESVIO JUSTIFICADO**: Difere por razão técnica válida (abordagem melhor encontrada, restrição da spec era impraticável, nova informação surgiu durante a implementação) → documentar no relatório mas NÃO reprovar o review somente por isso. Recomendar atualizar a TechSpec para refletir a abordagem real.

**Step 5: Task Completeness Verification (Mandatory)**
1. For each task marked as complete (if `tasks.md` is available):
   - Corresponding code was implemented.
   - Acceptance criteria were met.
   - Subtasks were all completed.
   - Task tests were implemented.

**Step 6: Test Execution (Mandatory)**
1. Detect the project's package manager from lock files (`bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm). Default to `npm` if none found.
2. Run the test suite using the detected package manager (e.g., `npm test`).
3. **E2E tests via MCP**: Execute the MCP discovery procedure from the shared skills directory resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code) — read the MCP configuration file for the current AI tool (`.mcp.json` for Claude Code, `.vscode/mcp.json` for GitHub Copilot, `.cursor/mcp.json` for Cursor) and the MCP capabilities file from the shared skills directory to build the capability map. Apply the capability guard:
   - Frontend changes + `browser-testing` MCP → run browser E2E via MCP tools. If MCP is unavailable, document the E2E gap in the review report — do NOT use CLI fallback.
   - Backend changes + backend-capable MCP (`message-queue`, `database`, `cache`, `api-testing`) → run backend E2E via MCP tools.
   - Changes type + no relevant MCP → skip E2E, document gap in review report.
4. If a `typecheck` script exists in `package.json`, run it. Otherwise, skip type checking.
5. Verify:
   - All tests pass.
   - New tests added for new code.
   - Coverage did not decrease.
   - Tests are meaningful (not just for coverage).
6. The review CANNOT be approved if any test fails.

**Step 7: Code Quality Analysis (Mandatory)**
1. Read the code quality checklist from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-review/references/code-quality-checklist.md` for Claude Code).
2. Use Context7 MCP (`resolve-library-id` → `query-docs`) to verify correct API usage, best practices, and recommended patterns for the frameworks/libraries used in the reviewed code. If Context7 MCP is unavailable, proceed without it.
3. Assess: complexity, DRY, SOLID, naming, comments, error handling, security, performance.

**Step 8: Generate Review Report (Mandatory)**
1. Read the report template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-review/assets/review-report-template.md` for Claude Code).
2. Fill in all sections with actual findings.
3. Save the report to `./pbis/pbi-[feature-slug]/review-report.md`.
4. **Important**: This skill only reports findings — it does NOT implement fixes. All issues are documented for the developer to address.
5. Apply approval criteria:
   - **APROVADO**: Todos os critérios atendidos, testes passando, código em conformidade com as regras e TechSpec.
   - **APROVADO COM RESSALVAS**: Critérios principais atendidos, problemas menores ou poucos problemas maiores não bloqueantes.
   - **REPROVADO**: Testes falhando, violações graves de regras, não-aderência à TechSpec ou problemas de segurança.

**Step 9: Create Fix Tasks (Mandatory when findings exist)**
1. Read the fix task template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-review/assets/fix-task-template.md` for Claude Code).
2. For each finding listed in the "Problemas Encontrados" section of the review report:
   a. Assign a sequential ID (R-01, R-02, …) if not already assigned.
   b. Generate a `brief-slug` from the finding description (lowercase, hyphen-separated, max 5 words).
   c. Fill the template with the finding's ID, severity, status (`pendente`), affected file/line, description, and suggested correction.
   d. Save to `./pbis/pbi-[feature-slug]/review-fixes/fix-[R-XX]-[severidade-completa]-[brief-slug].md`, where `[severidade-completa]` is the full severity word in lowercase: `critico`, `maior` or `menor`. Do NOT abbreviate (e.g. `fix-R01-critico-campo-senha-exposto.md`, NOT `fix-R01-crit-campo-senha.md`).
3. Create the `review-fixes/` directory automatically if it does not exist — no need to run `mkdir`, simply write the first file to the path.
4. If there are no findings (status APROVADO), skip this step entirely — do NOT create an empty directory.

**Step 10: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the review report is generated, if `TaskUpdate` is available (Claude Code), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Provide the final review report to the user.
3. **COMPLIANCE CHECK**: Before responding to the user, verify with actual tool calls:
    - Call `read_file` on `./pbis/pbi-[feature-slug]/review-report.md` to confirm the report was saved. If missing, go back to Step 8 and create it.
    - If findings exist, confirm that `./pbis/pbi-[feature-slug]/review-fixes/` contains one file per finding.
    - Do all findings match the `git diff` and code analysis?

## Output Language
Todos os artefatos gerados (relatório de review, arquivos de tarefa de correção) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If no git changes are found and git is available, report that there is nothing to review.
- If git is not initialized, fall back to manual file listing and document this in the report.
- If tests fail, the review status MUST be REPROVADO regardless of other findings.
- If PBI/TechSpec/tasks.md are missing, proceed with the review but document the missing context in the report.
- If a configured MCP is unavailable at runtime, follow its "Se indisponivel" handling from the registry. Do NOT use CLI fallback (e.g., `npx playwright test`) — document the E2E gap in the review report instead.
- Check if there are files that SHOULD have been modified but were not.
- Be constructive in criticism — always suggest alternatives.

## References
- Template: resolved in Step 0 (e.g., `.claude/skills/do-execute-review/assets/review-report-template.md` for Claude Code)
- Fix Task template: resolved in Step 0 (e.g., `.claude/skills/do-execute-review/assets/fix-task-template.md` for Claude Code)
- Code quality checklist: resolved in Step 0 (e.g., `.claude/skills/do-execute-review/references/code-quality-checklist.md` for Claude Code)
- MCP Discovery: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code)
- MCP Registry: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-capabilities.md` for Claude Code)
- PBI: `./pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `./pbis/pbi-[feature-slug]/techspec.md`
- Tasks: `./pbis/pbi-[feature-slug]/tasks/tasks.md`
- Review Report output: `./pbis/pbi-[feature-slug]/review-report.md`
- Fix Tasks output dir: `./pbis/pbi-[feature-slug]/review-fixes/fix-[R-XX]-[severidade-completa]-[slug].md`

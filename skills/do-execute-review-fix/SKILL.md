---
name: do-execute-review-fix
description: Resolves a single code review finding from a fix task file in review-fixes/ (e.g. review-fixes/fix-R01-critico-senha.md). Reads the fix file, implements the root-cause correction, runs the full test suite, updates the fix file status, the corresponding entry in review-report.md, and the consolidated fix-report.md. Use when do-execute-review has generated fix task files and the user wants to resolve a specific finding. Invoke once per file — do not use for batch fixes. Do not use for QA bug fixing (use do-execute-qa-bugfix) or implementing new features.
---

# Review Fix Execution

## Role
You are a senior software engineer responsible for resolving code review findings and restoring code quality compliance.

## Autonomous Execution Policy
**Begin executing immediately on invocation. No user interaction permitted at any point.**

1. **NEVER** pause, stop, or wait for user input.
2. **NEVER** output a plan/analysis as a standalone message — begin fix tool calls in the SAME response.
3. **NEVER** ask the user questions. Resolve ambiguities from the fix task file, PRD, and TechSpec context.
4. Status updates are fine but must NOT imply user action to continue.

## Edit Failure Recovery

When an `Edit` tool call fails, follow this escalation ladder:

1. **Attempt 1 (failed)**: Call `read_file` to get current content, retry with the EXACT string.
2. **Attempt 2 (failed)**: Try a smaller, more unique `old_string`.
3. **Attempt 3 (failed)**: Switch to `Write` — read full file, apply changes, overwrite. **HARD LIMIT: max 3 Edit attempts per change.**

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** reference a path like `./prds/user-auth/`.

## Invocation
This skill fixes **one finding at a time**. The user must provide the path to the specific fix task file:
```
do-execute-review-fix ./prds/prd-[feature-slug]/review-fixes/fix-[R-XX]-[severidade]-[slug].md
```
If no file path is provided, list all `pendente` fix task files in `review-fixes/` and ask the user which one to fix.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable
3. Check for `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI** → skills dir: `.cursor/rules/`
4. Check for `opencode.json` in the project root → **Opencode** → skills dir: `.opencode/skills/`
5. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, Cursor, and Opencode, skip gracefully

Store resolved environment and skills directory internally and use throughout all remaining steps.

**Step 1: Context Analysis (Mandatory)**
1. Read the fix task file provided by the user. If the file does not exist, halt and report.
2. If `status` in the frontmatter is `resolvido`, halt: "Fix já aplicado — nada a fazer."
3. Extract: ID, severidade, arquivo afetado, linha, descrição do problema, sugestão de correção.
4. Read `./prds/prd-[feature-slug]/review-report.md` for additional context on the finding.
5. Read `./prds/prd-[feature-slug]/prd.md` and `./prds/prd-[feature-slug]/techspec.md` for context.
6. Read the project configuration file (CLAUDE.md, .github/copilot-instructions.md, .cursor/rules/project.mdc, or opencode.json) for project conventions.

**Step 2: Plan Fix (INTERNAL — do NOT output as standalone message)**
1. Identify affected files and determine root cause from the fix task description.
2. Define fix strategy.
3. **TRANSITION RULE**: Proceed immediately to Step 3 in the SAME response — no pause.

**Step 3: Implement Fix (starts immediately after Step 2 — no pause, no confirmation)**
1. Detect package manager from lock files (`bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm, default: `npm`).
2. Read the affected file(s) completely.
3. Implement the root-cause fix — no superficial workarounds.
4. Run `typecheck` if it exists in `package.json`.
5. **Iteration limit**: Maximum 3 fix-and-test cycles per finding. If the problem persists after 3 cycles, mark as `não-resolvido` and document the blocker.

**Step 4: Run Full Test Suite (Mandatory Gate)**
1. Run all tests using the detected package manager (e.g., `npm test`).
2. If all tests pass → proceed to Step 5.
3. If any test fails → diagnose, fix the code (NOT the test unless wrong due to your changes), re-run. Maximum 5 fix-and-retest cycles.
4. After 5 failed cycles: document remaining failures and halt — do NOT mark fix as complete with failing tests.

**Step 5: Update Fix Task File (Mandatory)**
1. Update the fix task file's frontmatter `status`:
   - Fixed: `resolvido`
   - Blocked: `não-resolvido`
2. Append a `## Resolução` section describing the fix applied.
3. Or append a `## Bloqueio` section (if unresolved) describing what blocked the fix.

**Step 6: Update review-report.md (Mandatory)**
1. Locate the corresponding finding entry in `review-report.md` by matching the ID.
2. Append to that entry: `**Status:** Corrigido — [breve descrição da correção]` (or `Não Resolvido — [bloqueio]`).
3. Do NOT modify original review metadata (date, branch, summary).

**Step 7: Update Consolidated Fix Report (Mandatory)**
1. List all files in `./prds/prd-[feature-slug]/review-fixes/` matching `fix-*.md`.
2. For each file, read its frontmatter (`id`, `severidade`, `status`) and extract the short description from the heading and the `## Resolução` / `## Bloqueio` section if present.
3. Read the report template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-review-fix/assets/fix-report-template.md` for Claude Code, `.cursor/rules/do-execute-review-fix/assets/fix-report-template.md` for Cursor AI, `.opencode/skills/do-execute-review-fix/assets/fix-report-template.md` for Opencode).
4. Read `review-report.md` to extract the original review status (APROVADO COM RESSALVAS or REPROVADO) for the report header.
5. Generate (or overwrite) `./prds/prd-[feature-slug]/fix-report.md` filling the template with:
   - Resumo: total findings (sum), counts per severidade, totals corrigidos / não-resolvidos.
   - Tabela "Resultado por Finding": one row per file (ID, severidade, descrição, status, correção/bloqueio).
   - Testes: status of last full test run.
   - Findings Não Resolvidos: only entries with `status: não-resolvido`.
   - Próximo Passo: choose the appropriate suggestion based on whether all findings are resolved.
6. **POST-SAVE VERIFICATION**: Call `read_file` on `./prds/prd-[feature-slug]/fix-report.md` to confirm it was written.

**Step 8: Report Results (Mandatory)**
1. If `TaskUpdate` is available, mark internal tasks as `completed`.
2. **Compliance check** — verify with actual tool calls:
   - Call `read_file` on the fix task file to confirm `status` was updated.
   - Call `read_file` on `review-report.md` to confirm the finding entry was updated.
   - Call `read_file` on `fix-report.md` to confirm the consolidated report was written/updated.
3. Inform the user: finding fixed (or blocked) and tests passing.
4. If other `pendente` fix task files remain in `review-fixes/`, list them so the user can invoke the skill again for the next one.
5. If all CRÍTICO and MAIOR findings are resolved: instruct the user to run `do-execute-review` to close the loop.

## Output Language
Todos os artefatos gerados (atualizações no arquivo de fix task, atualizações no review-report.md, fix-report.md consolidado) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If no file path is provided, list all `pendente` fix task files in `review-fixes/` and ask the user which one to fix.
- If the fix task file does not exist, halt and report.
- If status is already `resolvido`, halt: nothing to do.
- If `review-report.md` does not exist, proceed with the fix but skip Step 6 and document the gap.
- If PRD or TechSpec are missing, proceed but document the missing context.
- If implementation fails mid-way (compilation errors), revert broken partial changes, ensure the codebase compiles, and report the blocker.
- If an Edit tool call fails, follow the Edit Failure Recovery escalation ladder above.

## References
- Fix task file (input): `./prds/prd-[feature-slug]/review-fixes/fix-[R-XX]-[severidade-completa]-[slug].md`
- Review Report: `./prds/prd-[feature-slug]/review-report.md`
- Consolidated Fix Report template: resolved in Step 0 (e.g., `.claude/skills/do-execute-review-fix/assets/fix-report-template.md` for Claude Code, `.cursor/rules/do-execute-review-fix/assets/fix-report-template.md` for Cursor AI, `.opencode/skills/do-execute-review-fix/assets/fix-report-template.md` for Opencode)
- Consolidated Fix Report output: `./prds/prd-[feature-slug]/fix-report.md`
- PRD: `./prds/prd-[feature-slug]/prd.md`
- TechSpec: `./prds/prd-[feature-slug]/techspec.md`

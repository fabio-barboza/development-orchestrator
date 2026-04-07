---
name: do-execute-review-fix
description: Recebe o caminho de um arquivo de fix task em review-fixes/ (ex: review-fixes/fix-R01-critico-senha.md), implementa a correção, roda a suite de testes e atualiza o status do arquivo. Use quando do-execute-review gerou arquivos de fix task e o usuário quiser resolver um problema específico. Não use para correções em lote — invoque uma vez por arquivo. Não use para QA, correção de bugs de QA (use do-execute-qa-bugfix) ou implementação de novas features.
---

# Review Fix Execution

## Role
You are a senior software engineer responsible for resolving code review findings and restoring code quality compliance.

## Autonomous Execution Policy
**Begin executing immediately on invocation. No user interaction permitted at any point.**

1. **NEVER** pause, stop, or wait for user input.
2. **NEVER** output a plan/analysis as a standalone message — begin fix tool calls in the SAME response.
3. **NEVER** ask the user questions. Resolve ambiguities from the fix task file, PBI, and TechSpec context.
4. Status updates are fine but must NOT imply user action to continue.

## Edit Failure Recovery

When an `Edit` tool call fails, follow this escalation ladder:

1. **Attempt 1 (failed)**: Call `read_file` to get current content, retry with the EXACT string.
2. **Attempt 2 (failed)**: Try a smaller, more unique `old_string`.
3. **Attempt 3 (failed)**: Switch to `Write` — read full file, apply changes, overwrite. **HARD LIMIT: max 3 Edit attempts per change.**

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. **NEVER** reference a path like `./pbis/user-auth/`.

## Invocation
This skill fixes **one finding at a time**. The user must provide the path to the specific fix task file:
```
do-execute-review-fix ./pbis/pbi-[feature-slug]/review-fixes/fix-[R-XX]-[severidade]-[slug].md
```
If no file path is provided, list all `pendente` fix task files in `review-fixes/` and ask the user which one to fix.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable (use file paths relative to this skill's location)
3. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully

Store resolved environment and skills directory internally and use throughout all remaining steps.

**Step 1: Context Analysis (Mandatory)**
1. Read the fix task file provided by the user. If the file does not exist, halt and report.
2. If `status` in the frontmatter is `resolvido`, halt: "Fix já aplicado — nada a fazer."
3. Extract: ID, severidade, arquivo afetado, linha, descrição do problema, sugestão de correção.
4. Read `./pbis/pbi-[feature-slug]/review-report.md` for additional context on the finding.
5. Read `./pbis/pbi-[feature-slug]/pbi.md` and `./pbis/pbi-[feature-slug]/techspec.md` for context.
6. Read the project configuration file (CLAUDE.md or equivalent) for project conventions.

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

**Step 7: Report Results (Mandatory)**
1. If `TaskUpdate` is available, mark internal tasks as `completed`.
2. **Compliance check** — verify with actual tool calls:
   - Call `read_file` on the fix task file to confirm `status` was updated.
   - Call `read_file` on `review-report.md` to confirm the finding entry was updated.
3. Inform the user: finding fixed (or blocked) and tests passing.
4. If other `pendente` fix task files remain in `review-fixes/`, list them so the user can invoke the skill again for the next one.
5. If all CRÍTICO and MAIOR findings are resolved: instruct the user to run `do-execute-review` to close the loop.

## Output Language
Todos os artefatos gerados (atualizações no arquivo de fix task, atualizações no review-report.md) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If no file path is provided, list all `pendente` fix task files in `review-fixes/` and ask the user which one to fix.
- If the fix task file does not exist, halt and report.
- If status is already `resolvido`, halt: nothing to do.
- If `review-report.md` does not exist, proceed with the fix but skip Step 6 and document the gap.
- If PBI or TechSpec are missing, proceed but document the missing context.
- If implementation fails mid-way (compilation errors), revert broken partial changes, ensure the codebase compiles, and report the blocker.
- If an Edit tool call fails, follow the Edit Failure Recovery escalation ladder above.

## References
- Fix task file (input): `./pbis/pbi-[feature-slug]/review-fixes/fix-[R-XX]-[severidade-completa]-[slug].md`
- Review Report: `./pbis/pbi-[feature-slug]/review-report.md`
- PBI: `./pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `./pbis/pbi-[feature-slug]/techspec.md`

---
name: do-execute-qa-bugfix
description: "Recebe o caminho de um arquivo de bug em qa-bugs/ (ex: qa-bugs/bug-01-alta-formulario.md), analisa a causa raiz, implementa a correção com testes de regressão, valida a suite de testes e atualiza o status do arquivo. Use quando o usuário pedir para corrigir um bug específico encontrado no QA. Não use para correções em lote — invoque uma vez por bug. Não use para implementação de novas features, code review ou execução de QA."
---

# Bug Fix Execution

## Role
You are a senior software engineer specialized in root-cause analysis and implementing robust, regression-tested bug fixes.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reason to stop and ask the user is when there is a genuine doubt or ambiguity that cannot be resolved by reading the project files. Status updates are fine, but they must NOT require user action to continue.

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. **NEVER** reference a path like `./pbis/user-auth/`.

## Invocation
This skill fixes **one bug at a time**. The user must provide the path to the specific bug file to fix:
```
do-execute-qa-bugfix ./pbis/pbi-[feature-slug]/qa-bugs/bug-[XX]-[severidade]-[slug].md
```
If no file path is provided, list all `aberto` bug files in `qa-bugs/` and ask the user which one to fix.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable (use file paths relative to this skill's location)
3. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Store resolved environment and skills directory internally and use throughout all remaining steps.

**Step 1: Context Analysis (Mandatory)**
1. Read the bug file provided by the user. If the file does not exist, halt and report.
2. If `status` in the frontmatter is `corrigido`, halt: "Bug já corrigido — nada a fazer."
3. Extract: ID, severidade, descrição, passos para reproduzir, resultado esperado/atual, componente afetado.
4. Read the PBI at `./pbis/pbi-[feature-slug]/pbi.md` to understand affected requirements.
5. Read the Tech Spec at `./pbis/pbi-[feature-slug]/techspec.md` for technical context.
6. Read the project configuration file (CLAUDE.md or equivalent) for project conventions.

**Step 2: Plan Fix (INTERNAL — do NOT output as standalone message)**
1. Identify affected files and determine root cause from the bug description.
2. Define fix strategy. Use Context7 MCP to verify documentation of involved libraries if needed.
3. **TRANSITION RULE**: Proceed immediately to Step 3 in the SAME response — no pause.

**Step 3: Implement Fix (starts immediately after Step 2 — no pause, no confirmation)**
1. Detect package manager from lock files (`bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm, default: `npm`).
2. Locate and read the affected code completely.
3. Implement the root-cause fix — no superficial workarounds.
4. If a `typecheck` script exists in `package.json`, run it after the fix.
5. **Iteration limit**: Maximum 3 fix-and-test cycles. If the bug persists after 3 cycles, mark as `não-resolvido` and document the blocker.

**Edit Failure Recovery**: When an `Edit` tool call fails: (1) `read_file` to get current content, retry with exact string. (2) Try a smaller, more unique `old_string`. (3) After 3 failed attempts, switch to `Write`. **HARD LIMIT: max 3 Edit retries per change.**

**Step 4: Create Regression Test (Mandatory)**
1. Read `references/regression-test-patterns.md` for patterns and naming conventions.
2. Create a test that:
   - Simulates the original bug scenario (must fail when the fix is reverted).
   - Validates the correct behavior with the fix applied.
   - Covers at least one related edge case.
3. Choose test type based on bug nature (unit / integration / E2E) as described in the reference.

**Step 5: MCP Validation (Mandatory when applicable)**
1. Execute the MCP discovery procedure from the shared skills directory resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code).
2. For bugs affecting the **UI** (and `browser-testing` MCP available): run `mkdir -p ./pbis/pbi-[feature-slug]/qa-screenshots` via Bash, then navigate, reproduce the fix flow, and capture screenshot evidence using `filename: pbis/pbi-[feature-slug]/qa-screenshots/fix-[BUG-XX]-[slug].png`.
3. For bugs affecting **backend** (and backend-capable MCP available): validate end-to-end via MCP tools.
4. If no relevant MCP available: document the validation gap in the fix report, rely on unit/integration tests only.

**Step 6: Final Test Execution (Mandatory Gate)**
1. Run ALL project tests using the detected package manager (e.g., `npm test`).
2. If a `typecheck` script exists, run it.
3. Verify ALL pass. The fix is NOT complete if any test fails.

**Step 7: Update Bug File (Mandatory)**
1. Update the bug file's frontmatter `status`:
   - Fixed: `corrigido`
   - Blocked: `não-resolvido`
2. Append a `## Resolução` section (if fixed) with: correção aplicada and testes de regressão criados.
3. Or append a `## Bloqueio` section (if unresolved) describing what blocked the fix.

**Step 8: Report Results (Mandatory)**
1. If `TaskUpdate` is available, mark internal tasks as `completed`.
2. **Compliance check**: call `read_file` on the bug file to confirm `status` was updated.
3. Inform the user: bug fixed (or blocked), tests passing, and which regression test was created.
4. If other `aberto` bugs remain in `qa-bugs/`, list them so the user can invoke the skill again for the next one.

## Output Language
Todos os artefatos gerados (atualizações no arquivo de bug, seções de resolução/bloqueio) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If no file path is provided, list all `aberto` bug files in `qa-bugs/` and ask the user which one to fix.
- If the bug file does not exist, halt and report.
- If status is already `corrigido`, halt: nothing to do.
- If a bug requires significant architectural changes, document the justification before proceeding.
- If new bugs are discovered during the fix, create a new bug file in `qa-bugs/` with status `novo` — do NOT fix in this cycle.
- If an MCP is unavailable, follow its "Se indisponivel" handling from the registry and document the gap.
- If implementation fails mid-way, revert broken partial changes, ensure the codebase compiles, and report the blocker.

## References
- Bug file (input): `./pbis/pbi-[feature-slug]/qa-bugs/bug-[XX]-[severidade-completa]-[slug].md`
- Regression test patterns: resolved in Step 0 (e.g., `.claude/skills/do-execute-qa-bugfix/references/regression-test-patterns.md` for Claude Code)
- MCP Discovery: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code)
- MCP Registry: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-capabilities.md` for Claude Code)
- PBI: `./pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `./pbis/pbi-[feature-slug]/techspec.md`
- Screenshots: `./pbis/pbi-[feature-slug]/qa-screenshots/`

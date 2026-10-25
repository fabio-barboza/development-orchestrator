---
name: do-execute-qa-bugfix
description: "Recebe o caminho de um arquivo de bug em qa/qa-[N]/bugs/ (ex: qa/qa-1/bugs/bug-01-alta-formulario.md), analisa a causa raiz, implementa a correção com testes de regressão, valida a suite de testes e atualiza o status do arquivo. Use quando o usuário pedir para corrigir um bug específico encontrado no QA. Não use para correções em lote — invoque uma vez por bug. Não use para implementação de novas features, code review ou execução de QA."
---

# Bug Fix Execution

## Role
You are a senior software engineer specialized in root-cause analysis and implementing robust, regression-tested bug fixes.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reason to stop and ask the user is when there is a genuine doubt or ambiguity that cannot be resolved by reading the project files. Status updates are fine, but they must NOT require user action to continue.

## Fix Scope — ABSOLUTE RULE
A bug fix changes **only code and tests** (plus the bug file and the round's `qa-report.md`, as the steps below say). **NEVER** edit the PRD, the TechSpec, `tasks.md`, task files or any other requirement or planning document, even if the bug file suggests it.

If resolving the bug would require changing a requirement document, or depends on a product or scope decision that only the user can make, do NOT decide it and do NOT edit anything: keep the bug open, record the decision the user must make in the bug file, and report it. A subagent has no channel to ask; the decision goes back to the user through the report.

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** reference a path like `./prds/user-auth/`.

## Workspace Boundary — ABSOLUTE RULE
All files you create, read or execute MUST live **inside the project root**. Files outside it may be stale or belong to another project/run.

1. **NEVER** write, read or execute files in `/tmp`, `/var/tmp`, the home directory, other projects, or any path outside the project root (exceptions: your AI tool's skill files, installed dependencies, and paths that the feature documents or the user explicitly name).
2. Scratch files (ad-hoc scripts, intermediate outputs) go ONLY in `./.do-tmp/qa-bugfix/`. Create it with `mkdir -p` when needed; if the project is a git repository and `.gitignore` does not already ignore `.do-tmp` (with or without the trailing slash), append `.do-tmp/` once.
3. Never reuse scratch files from other runs or from other `./.do-tmp/` folders.
4. Tests that must persist go in the project's test location, not in `./.do-tmp/`.
5. Delete `./.do-tmp/qa-bugfix/` before your final response.
6. Do not redirect command output to files: filter it in the pipe (`| tail`, `| grep`). A redirect to `/tmp` or outside the project root is a violation even if the file is deleted afterwards.

## Invocation
This skill fixes bugs from a QA round. The user provides:
```
do-execute-qa-bugfix ./prds/prd-[feature-slug]/qa/qa-[N]/bugs/bug-[XX]-[severidade]-[slug].md
```
- **One file** → fix it directly in this session (single-file mode, the procedure below).
- **A folder** (`.../bugs/`), **two or more files**, or a folder with `--min=<alta|media|baixa>` → **queue mode**: do NOT fix anything inline. Follow `do-shared/references/do-fix-queue.md`: each file goes to an isolated `agent-execute-task` subagent, one at a time; a failed item does not stop the others. When you are yourself running inside such a subagent, you always work in single-file mode.

If no file path is provided, list all `aberto` bug files of the most recent QA round (and of the legacy `qa-bugs/`, if any) as described in `do-shared/references/do-run-folders.md` and ask the user which one to fix.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Identify the AI tool you are running in — you know whether you are Claude Code, GitHub Copilot, Cursor AI or Opencode. Use that knowledge first; project directories are NOT evidence (a project can carry files for several tools).
2. Only if you cannot tell, fall back to project markers: `.claude/` → **Claude Code**; `.github/copilot-instructions.md`, `.github/agents/` or `.github/prompts/` (a bare `.github/` with only workflows does NOT indicate Copilot) → **GitHub Copilot**; `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI**; `opencode.json`, `.opencode/` or `AGENTS.md` → **Opencode**. If several match, prefer the tool whose agent/command files are installed and whose tools you actually have.
5. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, Cursor, and Opencode, skip gracefully
   - **Documentation MCP** (capability `documentation`): optional — use it only if discovered per `do-shared/references/do-mcp-discovery-instructions.md`; otherwise use Web Search or the codebase

Skill assets/references are loaded via your AI tool's native skill resolver — do not hard-code paths. Paths like `do-shared/...` are relative to the skills directory (the parent folder of this skill, e.g. `.agents/skills/`), not the project root; glob tools skip hidden directories such as `.agents/`, so read these files by their full path (see `do-shared/references/do-project-context.md`). Store the detected tool and capability flags internally.

**Step 1: Context Analysis (Mandatory)**
1. Read the bug file provided by the user. If the file does not exist, halt and report.
2. If `status` in the frontmatter is `corrigido`, halt: "Bug já corrigido — nada a fazer."
3. Extract: ID, severidade, descrição, passos para reproduzir, resultado esperado/atual, componente afetado.
4. Read the PRD at `./prds/prd-[feature-slug]/prd.md` to understand affected requirements.
5. Read the Tech Spec at `./prds/prd-[feature-slug]/techspec.md` for technical context.
6. Read the project configuration file (CLAUDE.md, AGENTS.md, .github/copilot-instructions.md, or .cursor/rules/project.mdc) for project conventions.
7. Load the project skills relevant to the affected component by following `do-shared/references/do-project-context.md` (Skills procedure); no specific skill is assumed to exist.
8. **Resolve the QA round from the bug file path** following `do-shared/references/do-run-folders.md`: `<qa-dir>` is the folder that contains `bugs/` (`qa-[N]`), or the PRD root for a legacy `qa-bugs/` file. Everything produced by this fix (screenshots, newly found bugs, updates) stays in that same round.

**Step 2: Plan Fix (INTERNAL — do NOT output as standalone message)**
1. Identify affected files and determine root cause from the bug description.
2. Define fix strategy. Verify documentation of involved libraries if needed (documentation MCP when discovered, otherwise Web Search or the codebase).
3. **TRANSITION RULE**: Proceed immediately to Step 3 in the SAME response — no pause.

**Step 3: Implement Fix (starts immediately after Step 2 — no pause, no confirmation)**
1. Resolve the project's commands (test suite, static checks) following `do-shared/references/do-project-commands.md`; never assume a language, package manager or tool.
2. Locate and read the affected code completely.
3. Implement the root-cause fix — no superficial workarounds.
4. If the project defines static checks (type checking, lint, formatting), run them after the fix.
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
1. Execute the MCP discovery procedure at `do-shared/references/do-mcp-discovery-instructions.md`.
2. For bugs affecting the **UI** (and `ui-testing` MCP available): run `mkdir -p ./<qa-dir>/screenshots` via Bash, then navigate, reproduce the fix flow, and capture screenshot evidence using `filename: <qa-dir>/screenshots/fix-[BUG-XX]-[slug].png`.
3. For bugs affecting a **service** (and a service-capable MCP — `api-testing`, `messaging` or `data-store` — available): validate end-to-end via MCP tools.
4. If no relevant MCP available: document the validation gap in the bug file's `## Resolução` section (Step 7), rely on unit/integration tests only.

**Step 6: Final Test Execution (Mandatory Gate)**
1. Run ALL project tests with the resolved test command.
2. If the project defines static checks, run them.
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
4. If other `aberto` bugs remain in the same round's `bugs/` folder, list them so the user can invoke the skill again for the next one.

## Output Language
Todos os artefatos gerados (atualizações no arquivo de bug, seções de resolução/bloqueio) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If no file path is provided, list all `aberto` bug files of the most recent QA round (and of the legacy `qa-bugs/`, if any) as described in `do-shared/references/do-run-folders.md` and ask the user which one to fix.
- If the bug file does not exist, halt and report.
- If status is already `corrigido`, halt: nothing to do.
- If a bug requires significant architectural changes, document the justification before proceeding.
- If new bugs are discovered during the fix, create a new bug file in the same round's `bugs/` folder (continuing that folder's numbering) with status `novo` — do NOT fix in this cycle.
- If an MCP is unavailable, follow its "Se indisponivel" handling from the registry and document the gap.
- If implementation fails mid-way, revert broken partial changes, ensure the codebase compiles, and report the blocker.

## References
- Round folders: `do-shared/references/do-run-folders.md`
- Bug file (input): `./prds/prd-[feature-slug]/qa/qa-[N]/bugs/bug-[XX]-[severidade-completa]-[slug].md`
- Regression test patterns: `do-execute-qa-bugfix/references/regression-test-patterns.md`
- MCP Discovery: `do-shared/references/do-mcp-discovery-instructions.md`
- MCP Registry: `do-shared/references/do-mcp-capabilities.md`
- PRD: `./prds/prd-[feature-slug]/prd.md`
- TechSpec: `./prds/prd-[feature-slug]/techspec.md`
- Screenshots: `./prds/prd-[feature-slug]/qa/qa-[N]/screenshots/`

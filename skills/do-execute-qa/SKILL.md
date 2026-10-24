---
name: do-execute-qa
description: Validates feature implementation against PRD, Tech Spec, and Tasks through E2E testing via available MCP tools, accessibility verification, and visual analysis. Documents all bugs found with screenshot evidence and generates a comprehensive QA report. Use when the user asks to run QA, validate a feature, or test implementation completeness. Do not use for code review, bug fixing, or task implementation.
---

# QA Execution

## Role
You are a senior QA engineer specialized in E2E testing, accessibility validation, and thorough feature verification. **Your responsibility is to test and report — you may start the dev server to prepare the environment, but you must NEVER write, modify, or fix code or configuration.** Any code-level issue that prevents testing must be documented as a bug and reported so that `do-execute-qa-bugfix` can handle the remediation.

## Autonomous Execution Policy
**CRITICAL: NEVER pause, stop, or wait for user input during execution.** Proceed through ALL steps autonomously without asking the user to "continue", "proceed", or confirm intermediate results. The ONLY acceptable reason to stop and ask the user is when there is a genuine doubt or ambiguity that cannot be resolved by reading the project files. Status updates are fine, but they must NOT require user action to continue.

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** reference a path like `./prds/user-auth/`.

## Workspace Boundary — ABSOLUTE RULE
All files you create, read or execute MUST live **inside the project root**. Files outside it may be stale or belong to another project/run.

1. **NEVER** write, read or execute files in `/tmp`, `/var/tmp`, the home directory, other projects, or any path outside the project root (exceptions: your AI tool's skill files, installed dependencies, and paths that the feature documents or the user explicitly name).
2. Scratch files (ad-hoc scripts, intermediate outputs) go ONLY in `./.do-tmp/qa/`. Create it with `mkdir -p` when needed; if the project is a git repository and `.gitignore` does not list `.do-tmp/`, append it.
3. Never reuse scratch files from other runs or from other `./.do-tmp/` folders.
4. Tests that must persist go in the project's test location, not in `./.do-tmp/`.
5. Delete `./.do-tmp/qa/` before your final response.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Identify the AI tool you are running in — you know whether you are Claude Code, GitHub Copilot, Cursor AI or Opencode. Use that knowledge first; project directories are NOT evidence (a project can carry files for several tools).
2. Only if you cannot tell, fall back to project markers: `.claude/` → **Claude Code**; `.github/copilot-instructions.md`, `.github/agents/` or `.github/prompts/` (a bare `.github/` with only workflows does NOT indicate Copilot) → **GitHub Copilot**; `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI**; `opencode.json`, `.opencode/` or `AGENTS.md` → **Opencode**. If several match, prefer the tool whose agent/command files are installed and whose tools you actually have.
5. Resolve available tools based on environment:
   - **TaskUpdate**: available in Claude Code; in Copilot, Cursor, and Opencode, skip gracefully
   - **Documentation MCP** (capability `documentation`): optional — use it only if discovered per `do-shared/references/do-mcp-discovery-instructions.md`; otherwise use Web Search or the codebase

Skill assets/references are loaded via your AI tool's native skill resolver — do not hard-code paths. Paths like `do-shared/...` are relative to the skills directory (the parent folder of this skill, e.g. `.agents/skills/`), not the project root; glob tools skip hidden directories such as `.agents/`, so read these files by their full path (see `do-shared/references/do-project-context.md`). Store the detected tool and capability flags internally.

**Step 1: Documentation Analysis (Mandatory)**
1. Read the PRD at `./prds/prd-[feature-slug]/prd.md` and extract ALL numbered functional requirements. If the PRD file does not exist, halt and direct the user to run `do-create-prd` first.
2. Read the Tech Spec at `./prds/prd-[feature-slug]/techspec.md` and verify implemented technical decisions. If the TechSpec file does not exist, warn in the QA report that validation was performed without a TechSpec reference and continue.
3. Read Tasks at `./prds/prd-[feature-slug]/tasks/tasks.md` and verify completion status of each task. If `tasks.md` does not exist, warn in the QA report and continue.
4. Load the project skills relevant to the feature under test by following `do-shared/references/do-project-context.md` (Skills procedure); no specific skill is assumed to exist. Then create a verification checklist based on the requirements.
5. **Resolve the QA round** following `do-shared/references/do-run-folders.md`: `<qa-dir>` = `prds/prd-[feature-slug]/qa-[N]`, where `[N]` is the highest existing `qa-<number>` folder + 1 (or 1). Decide `[N]` now and use the same value for the whole execution (bugs, screenshots and report all live inside it); create the folder with the first file you write. Never write into an existing round or overwrite the legacy `qa-report.md`/`qa-bugs/`/`qa-screenshots/` at the PRD root. Then collect the open bugs of the previous round (see "Pendências da rodada anterior" in that reference).
6. Do NOT skip this step — understanding requirements is fundamental for QA.
7. **DO NOT stop here. DO NOT present the checklist and wait for approval. Proceed IMMEDIATELY to Step 2.**

**Step 2: MCP Discovery & Capability Guard (starts immediately after Step 1 — no pause, no confirmation)**
1. **MCP Discovery**: Execute the discovery procedure at `do-shared/references/do-mcp-discovery-instructions.md`:
   a. List the MCP servers (names and type only — never print or read the whole configuration file, see the security rule in `do-shared/references/do-mcp-discovery-instructions.md`) from the MCP configuration file for the current AI tool (`.mcp.json` for Claude Code, `.vscode/mcp.json` for GitHub Copilot, `.cursor/mcp.json` for Cursor, `opencode.json` for Opencode) 
   b. Read the capability catalog at `do-shared/references/do-mcp-capabilities.md` and classify each server by what its tools do (never by the server's name).
   c. Build an internal capability map (e.g., `{ "ui-testing": ["<server-a>"], "messaging": ["<server-b>"] }`).
2. **Capability Guard**: Analyze the PRD, Tech Spec, and Tasks to determine if the feature involves a **user interface**, a **service** (API, messaging, data store), or **both**. Apply the capability guard from the discovery instructions:
   - Feature with a user interface + `ui-testing` MCP available → proceed to Steps 3-5 using UI-testing MCP tools.
   - Feature exposing a service + service-capable MCP available (`api-testing`, `messaging`, `data-store`) → proceed to Step 3 using those MCP tools (skip Steps 4-5, which are UI-specific).
   - Interface + service, with both kinds of MCP available → proceed to Steps 3-5 using both.
   - Feature type + no MCP with relevant capability → **skip Steps 3-5 entirely**, proceed to Step 6 (Bug Documentation), and document in the QA report that E2E testing was not possible ("MCP com capacidade [X] não configurado").
3. **Environment Preparation — Service Readiness** (only if MCP tools will be used):
   - Resolve the project's commands (local service start, health address) following `do-shared/references/do-project-commands.md`.
   - **Apply `do-shared/references/do-service-readiness.md`**: probe each required service (the application under test, an API, a broker, a data store) first; reuse it if already running; start only the project's own services with its local start command if missing; document gaps for external services (broker, data store, third-party APIs) instead of attempting to start them. **Never kill a running service** — services already up belong to the user and must be reused as-is.
   - **If a dev server fails to start or errors out during startup**, do NOT attempt to fix the code or configuration. Immediately create a bug file at `./<qa-dir>/bugs/bug-[XX]-alta-aplicacao-nao-inicializa.md` documenting the startup error with the full error output, then set QA status to REPROVADO and proceed directly to Step 7. Code fixes are the responsibility of `do-execute-qa-bugfix`.
   - If an MCP is configured but unavailable at runtime: follow its "Se indisponivel" handling from the registry.

**Step 3: E2E Tests via MCP (Mandatory — skipped only if capability guard determined no relevant MCP)**
1. Check documentation of frameworks/libraries involved in the feature under test — this helps validate expected component behavior, API responses, and correct usage patterns. If a documentation MCP (capability `documentation`) was discovered, use it; otherwise proceed without it.
2. **Screenshot directory — MANDATORY SETUP**: Before taking any screenshot, run `mkdir -p ./<qa-dir>/screenshots` via Bash to ensure the directory exists. The UI-testing MCP may save files relative to its own output directory — if the subdirectory does not exist, the file will land in the project root.
3. **Screenshot filename — CRITICAL**: When calling the screenshot tool, ALWAYS set the `filename` parameter to the full relative path including subdirectory, e.g. `<qa-dir>/screenshots/req-01-login-success.png`. Never pass just a filename without the path prefix.
4. For each functional requirement from the PRD, use the appropriate MCP tools (as identified in Step 2):
   - **UI-testing MCP** (requirements involving a user interface): Navigate to the feature, execute the expected flow, verify results, capture screenshot with `filename: <qa-dir>/screenshots/[name].png`. Always read the current screen state (snapshot/structure tool) before interacting. Check the interface's console/log output for errors. Verify the calls the interface makes to services, when the MCP exposes them.
   - **Service-capable MCP** (requirements involving a service): Verify the flow end-to-end using the MCP tools of the capability (see the catalog). For `messaging`: verify messages are published/consumed correctly, inspect queue state, validate side effects. For `data-store`: verify data integrity and state changes. For `api-testing`: verify requests and responses.
5. Mark each requirement as APROVADO or REPROVADO.

**Step 4: Accessibility Verification (Mandatory — only if `ui-testing` MCP is available)**
1. Use the accessibility standard defined by the PRD/TechSpec or the project conventions, if any. Read `references/accessibility-checklist.md` as the baseline checklist and for the UI-testing MCP instructions; adapt it to the interface type.
2. Verify all checklist items applicable to the feature under test.
3. Use the UI-testing MCP tools to test keyboard navigation, labels, focus order, and contrast.

**Step 5: Visual Verification (Mandatory — only if `ui-testing` MCP is available)**
1. Capture screenshots of main screens using the UI-testing MCP screenshot tool. Always set `filename` to the full relative path: `<qa-dir>/screenshots/visual-home-empty.png`, `<qa-dir>/screenshots/visual-home-with-data.png`, etc. The directory was already created in Step 3.
2. Verify layouts in different states (empty, with data, error).
3. Document visual inconsistencies found.
4. Verify responsiveness if applicable.

**Step 6: Bug Documentation**
1. Read the bug template at `do-execute-qa/assets/bug-template.md`.
2. For each bug found, create an individual file at `./<qa-dir>/bugs/bug-[XX]-[severidade-completa]-[brief-slug].md`, where:
   - `[XX]` is a sequential number (01, 02, …).
   - `[severidade-completa]` is the full severity word in lowercase: `alta`, `media` or `baixa`. Do NOT abbreviate.
   - `[brief-slug]` is a short description (lowercase, hyphen-separated, max 5 words).
   - Example: `bug-01-alta-formulario-nao-valida-email.md`
3. Fill the template with: ID, severidade, status (`aberto`), descrição, passos para reproduzir, resultado esperado, resultado atual, evidência (screenshot path) e componente afetado.
4. For bug screenshots, run `mkdir -p ./<qa-dir>/screenshots` if not already done, then call the screenshot tool with `filename: <qa-dir>/screenshots/bug-[XX]-[slug].png`.
5. If a blocking bug is found, document and report immediately.

**Step 7: Generate QA Report (Mandatory)**
1. Read the report template at `do-execute-qa/assets/qa-report-template.md`.
2. Fill in all sections with actual results, including `Pendências da Rodada Anterior` (open bugs of the previous round that still persist; do NOT create new bug files for them).
3. Include a "Ferramentas MCP Utilizadas" section listing which MCPs were used and which capabilities were missing, and the project skills record (read / none applicable / none installed) defined in `do-shared/references/do-project-context.md`.
4. Save the report to `./<qa-dir>/qa-report.md`.
5. Set status to APROVADO only when ALL PRD requirements are verified and functioning.

**Step 8: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the QA report is generated and bugs are documented, if `TaskUpdate` is available (Claude Code only; skip in Copilot and Cursor), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Provide the final QA report to the user.
3. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the QA report generated and saved?
    - Does `./<qa-dir>/bugs/` contain one file per bug found?
    - Did all E2E/Accessibility tests pass?

## Output Language
Todos os artefatos gerados (relatório de QA, arquivos de bug individuais) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If the PRD does not exist, halt and direct the user to run `do-create-prd`.
- If there is nothing to validate (no task marked complete in `tasks.md` and no implemented code for the feature), report that **in the response only**: create no file and no `qa-[N]` folder.
- If the TechSpec or tasks.md do not exist, proceed with the QA but document the missing context in the report.
- If a required service is not running, apply `do-shared/references/do-service-readiness.md` — probe first, reuse if running, start only safe dev servers when missing, document gaps for external services. Never kill a running service. If the dev server fails to start or throws errors, **immediately open a bug file** documenting the startup error (including full error output), finalize the report with status REPROVADO, and stop. Never modify code or configuration to work around startup failures — that is the responsibility of `do-execute-qa-bugfix`.
- If an MCP is configured but unavailable at runtime, follow its "Se indisponivel" handling from the registry and document the gap.
- If a blocking bug prevents testing subsequent features, document it and continue with testable areas.
- Bug numbering restarts at 01 in each QA round (the round folder identifies it); never write into the folder of a previous round or overwrite its files.

## References
- Template: `do-execute-qa/assets/qa-report-template.md`
- Accessibility checklist: `do-execute-qa/references/accessibility-checklist.md`
- MCP Discovery: `do-shared/references/do-mcp-discovery-instructions.md`
- MCP Registry: `do-shared/references/do-mcp-capabilities.md`
- Service Readiness: `do-shared/references/do-service-readiness.md`
- PRD: `./prds/prd-[feature-slug]/prd.md`
- TechSpec: `./prds/prd-[feature-slug]/techspec.md`
- Tasks: `./prds/prd-[feature-slug]/tasks/tasks.md`
- Bug template: `do-execute-qa/assets/bug-template.md`
- Round folders: `do-shared/references/do-run-folders.md`
- Bugs output dir: `./prds/prd-[feature-slug]/qa-[N]/bugs/bug-[XX]-[severidade-completa]-[slug].md`
- QA Report output: `./prds/prd-[feature-slug]/qa-[N]/qa-report.md`
- Screenshots output dir: `./prds/prd-[feature-slug]/qa-[N]/screenshots/`

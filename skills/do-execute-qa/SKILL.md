---
name: do-execute-qa
description: Validates feature implementation against PBI, Tech Spec, and Tasks through E2E testing via available MCP tools, accessibility verification (WCAG 2.2), and visual analysis. Documents all bugs found with screenshot evidence and generates a comprehensive QA report. Use when the user asks to run QA, validate a feature, or test implementation completeness. Do not use for code review, bug fixing, or task implementation.
---

# QA Execution

## Role
You are a senior QA engineer specialized in E2E testing, accessibility validation, and thorough feature verification.

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
1. Read the PBI at `./pbis/pbi-[feature-slug]/pbi.md` and extract ALL numbered functional requirements. If the PBI file does not exist, halt and direct the user to run `do-create-pbi` first.
2. Read the Tech Spec at `./pbis/pbi-[feature-slug]/techspec.md` and verify implemented technical decisions. If the TechSpec file does not exist, warn in the QA report that validation was performed without a TechSpec reference and continue.
3. Read Tasks at `./pbis/pbi-[feature-slug]/tasks/tasks.md` and verify completion status of each task. If `tasks.md` does not exist, warn in the QA report and continue.
4. Create a verification checklist based on the requirements.
5. Do NOT skip this step — understanding requirements is fundamental for QA.
6. **DO NOT stop here. DO NOT present the checklist and wait for approval. Proceed IMMEDIATELY to Step 2.**

**Step 2: MCP Discovery & Capability Guard (starts immediately after Step 1 — no pause, no confirmation)**
1. **MCP Discovery**: Execute the discovery procedure from the shared skills directory resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code):
   a. Read the MCP configuration file for the current AI tool (`.mcp.json` for Claude Code, `.vscode/mcp.json` for GitHub Copilot, `.cursor/mcp.json` for Cursor) to list configured MCP servers.
   b. Read the MCP capabilities file from the shared skills directory resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-capabilities.md` for Claude Code) to map each server to its capabilities and tools.
   c. Build an internal capability map (e.g., `{ "browser-testing": ["playwright"], "message-queue": ["rabbitmq"] }`).
2. **Capability Guard**: Analyze the PBI, Tech Spec, and Tasks to determine if the feature involves **frontend/UI**, **backend**, or **both**. Apply the capability guard from the discovery instructions:
   - Frontend feature + `browser-testing` MCP available → proceed to Steps 3-5 using browser MCP tools.
   - Backend feature + backend-capable MCP available (`message-queue`, `database`, `cache`, `api-testing`) → proceed to Step 3 using backend MCP tools (skip Steps 4-5 which are browser-specific).
   - Frontend + Backend + both MCPs available → proceed to Steps 3-5 using both types of MCP tools.
   - Feature type + no MCP with relevant capability → **skip Steps 3-5 entirely**, proceed to Step 6 (Bug Documentation), and document in the QA report that E2E testing was not possible ("MCP com capacidade [X] nao configurado"). Only unit/integration test results from `do-execute-task` or `do-execute-review` should be referenced.
3. **Environment Preparation** (only if MCP tools will be used):
   - For MCPs that require a running app (check "Requer app rodando" in registry): verify the service is accessible. If not, start only the dev server using known-safe commands (`npm run dev`, `npm start`, `bun dev`, `pnpm dev`) — do NOT automatically start brokers or external services; document the gap instead.
   - Detect the package manager from lock files (`bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm, default: `npm`).
   - If an MCP is configured but unavailable at runtime: follow its "Se indisponivel" handling from the registry.

**Step 3: E2E Tests via MCP (Mandatory — skipped only if capability guard determined no relevant MCP)**
1. Use Context7 MCP (`resolve-library-id` → `query-docs`) to check documentation of frameworks/libraries involved in the feature under test — this helps validate expected component behavior, API responses, and correct usage patterns. If Context7 MCP is unavailable, proceed without it.
2. **Screenshot directory — MANDATORY SETUP**: Before taking any screenshot, run `mkdir -p ./pbis/pbi-[feature-slug]/qa-screenshots` via Bash to ensure the directory exists. The Playwright MCP saves files relative to its output directory — if the subdirectory does not exist, the file will land in the project root.
3. **Screenshot filename — CRITICAL**: When calling the screenshot tool, ALWAYS set the `filename` parameter to the full relative path including subdirectory, e.g. `pbis/pbi-[feature-slug]/qa-screenshots/req-01-login-success.png`. Never pass just a filename without the path prefix.
4. For each functional requirement from the PBI, use the appropriate MCP tools (as identified in Step 2):
   - **Browser-testing MCP** (frontend requirements): Navigate to the feature, execute the expected flow, verify results, capture screenshot with `filename: pbis/pbi-[feature-slug]/qa-screenshots/[name].png`. Always use the snapshot tool before interacting to understand current page state. Check browser console for errors. Verify API calls via network requests tool.
   - **Backend MCP** (backend requirements — e.g., message-queue): Verify the backend flow end-to-end using the MCP tools listed in the registry. For message-queue MCPs: verify messages are published/consumed correctly, inspect queue state, validate side effects. For database/cache MCPs: verify data integrity and state changes.
5. Mark each requirement as APROVADO or REPROVADO.

**Step 4: Accessibility Verification (Mandatory — only if browser-testing MCP is available)**
1. Read `references/wcag-checklist.md` for the full WCAG 2.2 verification items and browser MCP testing instructions.
2. Verify all checklist items applicable to the feature under test.
3. Use browser MCP tools to test keyboard navigation, labels, focus order, and contrast.

**Step 5: Visual Verification (Mandatory — only if browser-testing MCP is available)**
1. Capture screenshots of main screens using the browser MCP screenshot tool. Always set `filename` to the full relative path: `pbis/pbi-[feature-slug]/qa-screenshots/visual-home-empty.png`, `pbis/pbi-[feature-slug]/qa-screenshots/visual-home-with-data.png`, etc. The directory was already created in Step 3.
2. Verify layouts in different states (empty, with data, error).
3. Document visual inconsistencies found.
4. Verify responsiveness if applicable.

**Step 6: Bug Documentation**
1. Read the bug template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-qa/assets/bug-template.md` for Claude Code).
2. For each bug found, create an individual file at `./pbis/pbi-[feature-slug]/qa-bugs/bug-[XX]-[severidade-completa]-[brief-slug].md`, where:
   - `[XX]` is a sequential number (01, 02, …).
   - `[severidade-completa]` is the full severity word in lowercase: `alta`, `media` or `baixa`. Do NOT abbreviate.
   - `[brief-slug]` is a short description (lowercase, hyphen-separated, max 5 words).
   - Example: `bug-01-alta-formulario-nao-valida-email.md`
3. Fill the template with: ID, severidade, status (`aberto`), descrição, passos para reproduzir, resultado esperado, resultado atual, evidência (screenshot path) e componente afetado.
4. For bug screenshots, run `mkdir -p ./pbis/pbi-[feature-slug]/qa-screenshots` if not already done, then call the screenshot tool with `filename: pbis/pbi-[feature-slug]/qa-screenshots/bug-[XX]-[slug].png`.
5. If a blocking bug is found, document and report immediately.

**Step 7: Generate QA Report (Mandatory)**
1. Read the report template from the skills directory resolved in Step 0 (e.g., `.claude/skills/do-execute-qa/assets/qa-report-template.md` for Claude Code).
2. Fill in all sections with actual results.
3. Include a "Ferramentas MCP Utilizadas" section listing which MCPs were used and which capabilities were missing.
4. Save the report to `./pbis/pbi-[feature-slug]/qa-report.md`.
5. Set status to APROVADO only when ALL PBI requirements are verified and functioning.

**Step 8: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the QA report is generated and bugs are documented, if `TaskUpdate` is available (Claude Code), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Provide the final QA report to the user.
3. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the QA report generated and saved?
    - Does `./pbis/pbi-[feature-slug]/qa-bugs/` contain one file per bug found?
    - Did all E2E/Accessibility tests pass?

## Output Language
Todos os artefatos gerados (relatório de QA, arquivos de bug individuais) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Error Handling
- If the PBI does not exist, halt and direct the user to run `do-create-pbi`.
- If the TechSpec or tasks.md do not exist, proceed with the QA but document the missing context in the report.
- If a required service is not running (app, broker, etc.), detect the package manager and start it, or document the gap if unable.
- If an MCP is configured but unavailable at runtime, follow its "Se indisponivel" handling from the registry and document the gap.
- If a blocking bug prevents testing subsequent features, document it and continue with testable areas.
- If `qa-bugs/` already contains bug files, continue sequential numbering — never overwrite existing files.

## References
- Template: resolved in Step 0 (e.g., `.claude/skills/do-execute-qa/assets/qa-report-template.md` for Claude Code)
- Accessibility checklist: resolved in Step 0 (e.g., `.claude/skills/do-execute-qa/references/wcag-checklist.md` for Claude Code)
- MCP Discovery: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-discovery-instructions.md` for Claude Code)
- MCP Registry: resolved in Step 0 (e.g., `.claude/skills/do-shared/do-mcp-capabilities.md` for Claude Code)
- PBI: `./pbis/pbi-[feature-slug]/pbi.md`
- TechSpec: `./pbis/pbi-[feature-slug]/techspec.md`
- Tasks: `./pbis/pbi-[feature-slug]/tasks/tasks.md`
- Bug template: resolved in Step 0 (e.g., `.claude/skills/do-execute-qa/assets/bug-template.md` for Claude Code)
- Bugs output dir: `./pbis/pbi-[feature-slug]/qa-bugs/bug-[XX]-[severidade-completa]-[slug].md`
- QA Report output: `./pbis/pbi-[feature-slug]/qa-report.md`
- Screenshots output dir: `./pbis/pbi-[feature-slug]/qa-screenshots/`

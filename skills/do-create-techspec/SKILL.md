---
name: do-create-techspec
description: Creates Technical Specifications from existing PRDs, translating product requirements into architectural decisions and implementation guidance. Performs deep project analysis, uses Context7 MCP for technical research and Web Search for business rules. Use when the user asks to create a tech spec, define architecture, or plan implementation for a feature with an existing PRD. Do not use for PRD creation, task breakdowns, or direct code implementation.
---

# Tech Spec Creation

## Role
You are a senior software architect specialized in translating product requirements into clear, implementation-ready technical specifications.

## Interactive Execution Policy
**This skill is interactive by design.** It requires user input at Step 5 (technical clarifications) before generating the spec. Do NOT proceed past Step 5 without explicit user answers.

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to produce the Tech Spec document. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** reference a path like `./prds/user-auth/` (without the `prd-` prefix). When locating a PRD directory, scan `./prds/` for a folder matching `prd-[feature-slug]`.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable
3. Check for `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI** → skills dir: `.cursor/rules/`
4. Resolve available tools based on environment:
   - **AskUserQuestion**: use in all environments.
   - **TaskUpdate**: available in Claude Code; in Copilot and Cursor, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Store resolved environment and tool availability internally and use throughout all remaining steps.

**Step 1: Validate Prerequisites**
1. Confirm the feature slug has been provided.
2. Verify the PRD exists at `prds/prd-[feature-slug]/prd.md`. If missing, halt and report.

**Step 2: Analyze PRD (Mandatory)**
1. Read the PRD completely — do NOT skip this step.
2. Identify technical content, constraints, and success metrics.
3. Extract core requirements for architectural consideration.

**Step 3: Deep Project Analysis (Mandatory)**
1. Explore the codebase to discover files, modules, interfaces, and integration points.
2. Map symbols, dependencies, and critical paths.
3. Analyze: callers/callees, configs, middleware, persistence, concurrency, error handling, tests, infra.
4. Explore solution strategies, patterns, risks, and alternatives.

**Step 4: Research (Mandatory)**
1. Use Context7 MCP to resolve technical questions about frameworks and libraries.
2. Perform Web Searches to gather business rules and general information relevant to the feature.
3. Complete all research BEFORE asking clarification questions.

**Step 5: Technical Clarifications (Mandatory)**
1. Explore the project BEFORE asking questions.
2. Ask focused clarification questions covering:
   - Domain positioning.
   - Data flow.
   - External dependencies.
   - Key interfaces.
   - Test scenarios.
   - Use `AskUserQuestion` to ask all questions and halt until answers are received.
3. Do NOT proceed until answers are received.

**Step 6: Standards Compliance Mapping (Mandatory)**
1. Identify project skills using the skills directory resolved in Step 0. Skip this sub-step if the skills directory is not applicable (e.g., GitHub Copilot without a configured skills path). **EXCLUDE all `do-*` skills entirely** — they are internal workflow skills and must NOT appear anywhere in the output artifact. Only evaluate technology/library skills (e.g., `claude-api`, `find-skills`). For Cursor AI, scan `.cursor/rules/` for `.mdc` files.
2. Highlight deviations with justification and compliant alternatives.

**Step 7: Generate Tech Spec (Mandatory)**
1. Read the template at the path resolved in Step 0 (e.g., `.claude/skills/do-create-techspec/assets/techspec-template.md` for Claude Code, `.cursor/rules/do-create-techspec/assets/techspec-template.md` for Cursor AI).
2. Provide: architecture overview, component design, interfaces, data models, endpoints, integration points, impact analysis, test strategy, observability.
3. Focus on HOW, not WHAT (the PRD owns what/why).
4. Avoid repeating functional requirements from the PRD.
5. The spec is about specification, NOT detailed implementation code.
6. Keep under ~2,000 words.
7. Do NOT deviate from the template structure.
8. Prefer existing libraries over custom development.

**Step 8: Save Tech Spec (Mandatory)**
1. **PATH VERIFICATION**: Before writing, confirm the target path is exactly `./prds/prd-[feature-slug]/techspec.md`. Verify the directory name starts with `prd-`. Verify the PRD directory exists (it must, since the PRD was read in Step 2).
2. Save to: `./prds/prd-[feature-slug]/techspec.md`.
3. **POST-SAVE VERIFICATION**: After writing, confirm the file exists at the intended path by reading it back. If the file is not found, halt and report the error.

**Step 9: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the Tech Spec is saved, if `TaskUpdate` is available, use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Provide the final file path.
3. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the Tech Spec file saved correctly?
    - Is the internal task tracking synchronized?
    - Does the spec follow the architecture guidelines?

## Output Language
Todos os artefatos gerados (documento Tech Spec) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis, nomes de API e caminhos de arquivos permanecem em inglês.

## Core Principles
- Tech Spec focuses on HOW, not WHAT (PRD owns the what/why).
- Prefer simple, evolutionary architecture with clear interfaces.
- Provide testability and observability considerations upfront.
- Prefer existing libraries over custom solutions.

## Quality Checklist
- [ ] PRD reviewed.
- [ ] Deep repository analysis completed.
- [ ] Key technical clarifications answered.
- [ ] Tech Spec generated using the template.
- [ ] Project skills verified for compliance.
- [ ] File written to `./prds/prd-[feature-slug]/techspec.md`.
- [ ] Final output path provided and confirmed.

## Error Handling
- If the PRD does not exist at the expected path, halt and ask the user to create it first via the `do-create-prd` skill.
- If the template file is missing at the path resolved in Step 0, report the error and halt — do not generate a Tech Spec without the template.
- If Context7 MCP is unavailable, fall back to Web Search for technical documentation.
- If the output file already exists, confirm with the user before overwriting.

## References
- Template: resolved in Step 0 (e.g., `.claude/skills/do-create-techspec/assets/techspec-template.md` for Claude Code, `.cursor/rules/do-create-techspec/assets/techspec-template.md` for Cursor AI)
- PRD: `prds/prd-[feature-slug]/prd.md`
- Output: `prds/prd-[feature-slug]/techspec.md`

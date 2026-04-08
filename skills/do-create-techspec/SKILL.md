---
name: do-create-techspec
description: Creates Technical Specifications from existing PBIs, translating product requirements into architectural decisions and implementation guidance. Performs deep project analysis, uses Context7 MCP for technical research and Web Search for business rules. Use when the user asks to create a tech spec, define architecture, or plan implementation for a feature with an existing PBI. Do not use for PBI creation, task breakdowns, or direct code implementation.
---

# Tech Spec Creation

## Role
You are a senior software architect specialized in translating product requirements into clear, implementation-ready technical specifications.

## Interactive Execution Policy
**This skill is interactive by design.** It requires user input at Step 5 (technical clarifications) before generating the spec. Do NOT proceed past Step 5 without explicit user answers.

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to produce the Tech Spec document. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. **NEVER** reference a path like `./pbis/user-auth/` (without the `pbi-` prefix). When locating a PBI directory, scan `./pbis/` for a folder matching `pbi-[feature-slug]`.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot** → skills dir: not applicable (skip skills discovery in Step 6)
3. Resolve available tools based on environment:
   - **AskUserQuestion**: use in all environments (Claude Code and GitHub Copilot).
   - **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully
   - **Context7 MCP**: available if configured; fallback to Web Search otherwise

Store resolved environment and tool availability internally and use throughout all remaining steps.

**Step 1: Validate Prerequisites**
1. Confirm the feature slug has been provided.
2. Verify the PBI exists at `pbis/pbi-[feature-slug]/pbi.md`. If missing, halt and report.

**Step 2: Analyze PBI (Mandatory)**
1. Read the PBI completely — do NOT skip this step.
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
1. Identify project skills using the skills directory resolved in Step 0. Skip this sub-step if the skills directory is not applicable (e.g., GitHub Copilot without a configured skills path).
2. Highlight deviations with justification and compliant alternatives.

**Step 7: Generate Tech Spec (Mandatory)**
1. Read the template at the path resolved in Step 0 (e.g., `.claude/skills/do-create-techspec/assets/techspec-template.md` for Claude Code, or the equivalent path for the current tool).
2. Provide: architecture overview, component design, interfaces, data models, endpoints, integration points, impact analysis, test strategy, observability.
3. Focus on HOW, not WHAT (the PBI owns what/why).
4. Avoid repeating functional requirements from the PBI.
5. The spec is about specification, NOT detailed implementation code.
6. Keep under ~2,000 words.
7. Do NOT deviate from the template structure.
8. Prefer existing libraries over custom development.

**Step 8: Save Tech Spec (Mandatory)**
1. **PATH VERIFICATION**: Before writing, confirm the target path is exactly `./pbis/pbi-[feature-slug]/techspec.md`. Verify the directory name starts with `pbi-`. Verify the PBI directory exists (it must, since the PBI was read in Step 2).
2. Save to: `./pbis/pbi-[feature-slug]/techspec.md`.
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
- Tech Spec focuses on HOW, not WHAT (PBI owns the what/why).
- Prefer simple, evolutionary architecture with clear interfaces.
- Provide testability and observability considerations upfront.
- Prefer existing libraries over custom solutions.

## Quality Checklist
- [ ] PBI reviewed.
- [ ] Deep repository analysis completed.
- [ ] Key technical clarifications answered.
- [ ] Tech Spec generated using the template.
- [ ] Project skills verified for compliance.
- [ ] File written to `./pbis/pbi-[feature-slug]/techspec.md`.
- [ ] Final output path provided and confirmed.

## Error Handling
- If the PBI does not exist at the expected path, halt and ask the user to create it first via the `do-create-pbi` skill.
- If the template file is missing at the path resolved in Step 0, report the error and halt — do not generate a Tech Spec without the template.
- If Context7 MCP is unavailable, fall back to Web Search for technical documentation.
- If the output file already exists, confirm with the user before overwriting.

## References
- Template: resolved in Step 0 (e.g., `.claude/skills/do-create-techspec/assets/techspec-template.md` for Claude Code)
- PBI: `pbis/pbi-[feature-slug]/pbi.md`
- Output: `pbis/pbi-[feature-slug]/techspec.md`

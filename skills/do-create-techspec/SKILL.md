---
name: do-create-techspec
description: Creates Technical Specifications from existing PBIs, translating product requirements into architectural decisions and implementation guidance. Performs deep project analysis, uses Context7 MCP for technical research and Web Search for business rules. Use when the user asks to create a tech spec, define architecture, or plan implementation for a feature with an existing PBI. Do not use for PBI creation, task breakdowns, or direct code implementation.
---

# Tech Spec Creation

## Role
You are a senior software architect specialized in translating product requirements into clear, implementation-ready technical specifications.

## Interactive Execution Policy
**This skill is interactive by design.** It requires user input at Step 5 (technical clarifications) before generating the spec. Do NOT proceed past Step 5 without explicit user answers.

## Procedures

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
2. Ask focused clarification questions using the AskUserQuestion tool covering:
   - Domain positioning.
   - Data flow.
   - External dependencies.
   - Key interfaces.
   - Test scenarios.
3. Do NOT proceed until answers are received.

**Step 6: Standards Compliance Mapping (Mandatory)**
1. Identify project skills in the AI tool's skills directory (e.g., `.claude/skills/` for Claude Code) that apply to this spec.
2. Highlight deviations with justification and compliant alternatives.

**Step 7: Generate Tech Spec (Mandatory)**
1. Read the template at `.claude/skills/do-create-techspec/assets/techspec-template.md`.
2. Provide: architecture overview, component design, interfaces, data models, endpoints, integration points, impact analysis, test strategy, observability.
3. Focus on HOW, not WHAT (the PBI owns what/why).
4. Avoid repeating functional requirements from the PBI.
5. The spec is about specification, NOT detailed implementation code.
6. Keep under ~2,000 words.
7. Do NOT deviate from the template structure.
8. Prefer existing libraries over custom development.

**Step 8: Save Tech Spec (Mandatory)**
1. Save to: `pbis/pbi-[feature-slug]/techspec.md`.
2. Confirm the write operation and path.

**Step 9: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the Tech Spec is saved, use the `TaskUpdate` tool to mark all corresponding items in your internal task tracking as `completed`.
2. Provide the final file path.
3. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the Tech Spec file saved correctly?
    - Is the internal task tracking synchronized?
    - Does the spec follow the architecture guidelines?

## Output Language
All generated artifacts (Tech Spec document) must be written in Brazilian Portuguese (PT-BR). Code examples, variable names, API names, and technical terms remain in English.

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
- If the template file (`.claude/skills/do-create-techspec/assets/techspec-template.md`) is missing, report the error and halt — do not generate a Tech Spec without the template.
- If Context7 MCP is unavailable, fall back to Web Search for technical documentation.
- If the output file already exists, confirm with the user before overwriting.

## References
- Template: `.claude/skills/do-create-techspec/assets/techspec-template.md`
- PBI: `pbis/pbi-[feature-slug]/pbi.md`
- Output: `pbis/pbi-[feature-slug]/techspec.md`

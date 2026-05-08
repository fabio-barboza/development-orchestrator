---
name: do-create-prd
description: Creates Product Requirements Documents (PRDs) from feature requests following a structured workflow of clarification, planning, and drafting. Outputs a standardized PRD to the project prds directory. Use when the user asks to create a PRD, define requirements, or document a new feature. Do not use for technical specifications, task breakdowns, or implementation planning.
---

# PRD Creation

## Role
You are a senior product manager specialized in writing clear, actionable PRDs for development and product teams.

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. **NEVER** create or reference a path like `./prds/user-auth/` (without the `prd-` prefix).

## Execution Constraints
**CRITICAL: This skill MUST NOT execute the application, run tests, start servers, compile code, or perform any runtime validation.** Its sole purpose is to produce the PRD document. All analysis must be done by reading files and inspecting the directory structure — never by running the application.

## Resumption Detection (Stateless Environments: GitHub Copilot, Cursor AI)
In GitHub Copilot and Cursor AI, each user message starts a fresh invocation — the agent has no memory of previous turns. To handle resumption:

1. **On every invocation**, before running Step 1, check if a `prd-answers.md` file exists at `./prds/prd-[feature-slug]/prd-answers.md` (use the slug derived from the user's input).
2. **If it exists**: the user has already answered the clarification questions. Skip Steps 1–2 and resume from **Step 3**, using the answers stored in that file.
3. **After presenting questions (Step 2, stateless environment path)**: immediately save the questions to `./prds/prd-[feature-slug]/prd-answers.md` as a placeholder (with empty answer fields). Instruct the user: *"Edite o arquivo `prd-answers.md` com suas respostas e invoque `/do-create-prd` novamente para continuar."*
4. **On resumption**: read `prd-answers.md`, use the answers, then delete the file after the PRD is successfully saved.

## Procedures

**Step 0: Detect AI Tool Environment**
Before anything else, determine the execution environment:
1. Check for `.claude/` directory in the project root → **Claude Code**
2. Check for `.github/copilot-instructions.md` or `.github/` directory → **GitHub Copilot**
3. Check for `.cursor/rules/` or `.cursor/mcp.json` → **Cursor AI**
4. Check for `opencode.json`, `.opencode/` directory, or `AGENTS.md` → **Opencode**
5. Resolve available tools based on environment:
   - **AskUserQuestion**: use in all environments.
   - **TaskUpdate**: available in Claude Code; in Copilot, Cursor, and Opencode, skip gracefully.

Skill assets/references are loaded via your AI tool's native skill resolver — do not hard-code paths. Store the detected tool and capability flags internally.

**Step 1: Validate Prerequisites**
1. Confirm the feature name or description has been provided by the user.
2. Derive the slug in kebab-case and apply the mandatory `prd-` prefix for the output directory: `./prds/prd-[feature-slug]/`. Example: `user-auth` → `./prds/prd-user-auth/`.

**Step 2: Clarify Requirements (Mandatory)**
1. Use `AskUserQuestion` to ask the user clarification questions before generating any content. Halt until answers are received.
2. Cover all areas from the clarification checklist:
   - **Problem and Objectives**: What problem to solve, measurable goals.
   - **Users and Stories**: Primary users, user stories, main flows.
   - **Core Functionality**: Data inputs/outputs, actions.
   - **Scope and Planning**: What is NOT included, dependencies.
   - **Design and Experience**: UI/UX guidelines and accessibility.
3. Do NOT proceed to Step 3 until clarification answers are received.

**Step 3: Plan the PRD (Mandatory)**
1. Create a development plan including:
   - Section-by-section approach.
   - Areas requiring research (use Web Search for business rules).
   - Assumptions and dependencies.
2. Present the plan to the user for alignment.

**Step 4: Draft the PRD (Mandatory)**
1. Read the template at `do-create-prd/assets/prd-template.md`.
2. Focus on WHAT and WHY, never on HOW (implementation belongs in Tech Spec).
3. Include numbered functional requirements.
4. Keep the document under 2,000 words.
5. Do NOT deviate from the template structure.

**Step 5: Save the PRD (Mandatory)**
1. **PATH VERIFICATION**: Before writing, confirm the target path is exactly `./prds/prd-[feature-slug]/prd.md`. Verify the directory name starts with `prd-`. If the `./prds/` directory does not exist, create it first.
2. Create the directory: `./prds/prd-[feature-slug]/`.
3. Save the PRD to: `./prds/prd-[feature-slug]/prd.md`.
4. **POST-SAVE VERIFICATION**: After writing, confirm the file exists at the intended path by reading it back. If the file is not found, halt and report the error.

**Step 6: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the PRD is saved, if `TaskUpdate` is available (Claude Code only; skip in Copilot and Cursor), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Provide the final file path.
3. Provide a brief summary of the PRD outcome.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the PRD file saved correctly?
    - Is the internal task tracking synchronized?
    - Did you follow the template structure?

## Output Language
Todos os artefatos gerados (documento PRD, resumos) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Core Principles
- Clarify before planning; plan before drafting.
- Minimize ambiguity; prefer measurable statements.
- PRD defines outcomes and constraints, NOT implementation.
- Always consider usability and accessibility.

## Quality Checklist
- [ ] Clarification questions completed and answered.
- [ ] Detailed plan created.
- [ ] PRD generated using the template.
- [ ] Numbered functional requirements included.
- [ ] File saved to `./prds/prd-[feature-slug]/prd.md`.
- [ ] Final path provided.

## Error Handling
- If the user provides insufficient context, ask follow-up clarification questions before proceeding.
- If the template file is missing at the skills directory path (e.g., `do-create-prd/assets/prd-template.md`), report the error and halt — do not generate a PRD without the template.
- If the output directory already exists, confirm with the user before overwriting.
- If the output file cannot be written (permission error, invalid path), report the error to the user.

## References
- Template: `do-create-prd/assets/prd-template.md`
- Output: `./prds/prd-[feature-slug]/prd.md`

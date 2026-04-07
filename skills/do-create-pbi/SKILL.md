---
name: do-create-pbi
description: Creates Product Backlog Items (PBIs) from feature requests following a structured workflow of clarification, planning, and drafting. Outputs a standardized PBI to the project pbis directory. Use when the user asks to create a PBI, define requirements, or document a new feature. Do not use for technical specifications, task breakdowns, or implementation planning.
---

# PBI Creation

## Role
You are a senior product manager specialized in writing clear, actionable PBIs for development and product teams.

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. **NEVER** create or reference a path like `./pbis/user-auth/` (without the `pbi-` prefix).

## Resumption Detection (GitHub Copilot only)
In GitHub Copilot, each user message starts a fresh invocation — the agent has no memory of previous turns. To handle resumption:

1. **On every invocation**, before running Step 1, check if a `pbi-answers.md` file exists at `./pbis/pbi-[feature-slug]/pbi-answers.md` (use the slug derived from the user's input).
2. **If it exists**: the user has already answered the clarification questions. Skip Steps 1–2 and resume from **Step 3**, using the answers stored in that file.
3. **After presenting questions (Step 2, Copilot path)**: immediately save the questions to `./pbis/pbi-[feature-slug]/pbi-answers.md` as a placeholder (with empty answer fields). Instruct the user: *"Edite o arquivo `pbi-answers.md` com suas respostas e invoque `/do-create-pbi` novamente para continuar."*
4. **On resumption**: read `pbi-answers.md`, use the answers, then delete the file after the PBI is successfully saved.

## Procedures

**Step 1: Validate Prerequisites**
1. Confirm the feature name or description has been provided by the user.
2. Derive the slug in kebab-case and apply the mandatory `pbi-` prefix for the output directory: `./pbis/pbi-[feature-slug]/`. Example: `user-auth` → `./pbis/pbi-user-auth/`.

**Step 2: Clarify Requirements (Mandatory)**
1. Use `AskUserQuestion` to ask the user clarification questions before generating any content. Halt until answers are received.
2. Cover all areas from the clarification checklist:
   - **Problem and Objectives**: What problem to solve, measurable goals.
   - **Users and Stories**: Primary users, user stories, main flows.
   - **Core Functionality**: Data inputs/outputs, actions.
   - **Scope and Planning**: What is NOT included, dependencies.
   - **Design and Experience**: UI/UX guidelines and accessibility.
3. Do NOT proceed to Step 3 until clarification answers are received.

**Step 3: Plan the PBI (Mandatory)**
1. Create a development plan including:
   - Section-by-section approach.
   - Areas requiring research (use Web Search for business rules).
   - Assumptions and dependencies.
2. Present the plan to the user for alignment.

**Step 4: Draft the PBI (Mandatory)**
1. Read the template at `.claude/skills/do-create-pbi/assets/pbi-template.md`.
2. Focus on WHAT and WHY, never on HOW (implementation belongs in Tech Spec).
3. Include numbered functional requirements.
4. Keep the document under 2,000 words.
5. Do NOT deviate from the template structure.

**Step 5: Save the PBI (Mandatory)**
1. Create the directory: `./pbis/pbi-[feature-slug]/`.
2. Save the PBI to: `./pbis/pbi-[feature-slug]/pbi.md`.

**Step 6: Report Results & Sync Progress (Mandatory)**
1. **SYNC INTERNAL PROGRESS**: Once the PBI is saved, if `TaskUpdate` is available (Claude Code), use it to mark all corresponding items in your internal task tracking as `completed`. Otherwise, skip this step.
2. Provide the final file path.
3. Provide a brief summary of the PBI outcome.
4. **COMPLIANCE CHECK**: Before responding to the user, verify:
    - Is the PBI file saved correctly?
    - Is the internal task tracking synchronized?
    - Did you follow the template structure?

## Environment Detection
Before anything else, determine the execution environment:
1. Check for `.claude/` directory → **Claude Code** → skills dir: `.claude/skills/`
2. Check for `.github/` directory → **GitHub Copilot** → skills dir: not applicable
3. **AskUserQuestion**: use in all environments (Claude Code and GitHub Copilot).
4. **TaskUpdate**: available in Claude Code; in Copilot, skip gracefully

## Output Language
Todos os artefatos gerados (documento PBI, resumos) devem ser escritos em Português do Brasil (PT-BR). Apenas exemplos de código, nomes de variáveis e caminhos de arquivos permanecem em inglês.

## Core Principles
- Clarify before planning; plan before drafting.
- Minimize ambiguity; prefer measurable statements.
- PBI defines outcomes and constraints, NOT implementation.
- Always consider usability and accessibility.

## Quality Checklist
- [ ] Clarification questions completed and answered.
- [ ] Detailed plan created.
- [ ] PBI generated using the template.
- [ ] Numbered functional requirements included.
- [ ] File saved to `./pbis/pbi-[feature-slug]/pbi.md`.
- [ ] Final path provided.

## Error Handling
- If the user provides insufficient context, ask follow-up clarification questions before proceeding.
- If the template file is missing at the skills directory path (e.g., `.claude/skills/do-create-pbi/assets/pbi-template.md` for Claude Code), report the error and halt — do not generate a PBI without the template.
- If the output directory already exists, confirm with the user before overwriting.
- If the output file cannot be written (permission error, invalid path), report the error to the user.

## References
- Template: resolved by environment (e.g., `.claude/skills/do-create-pbi/assets/pbi-template.md` for Claude Code)
- Output: `./pbis/pbi-[feature-slug]/pbi.md`

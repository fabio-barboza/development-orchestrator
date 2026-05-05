---
name: do-status
description: Shows current progress of a PRD — completed tasks, next pending task, completion percentage, and missing artifacts. Use when the user asks for status, progress, what's next, or wants to resume work on a feature. Read-only skill — does not modify any files. Do not use for task implementation, review, or QA.
---

# Status Report

## Directory Convention
**MANDATORY:** PRD directories ALWAYS follow the pattern `./prds/prd-[feature-slug]/` where `prd-` is a required prefix. Example: feature `user-auth` → directory `./prds/prd-user-auth/`. When scanning `./prds/`, only consider folders matching the `prd-*` pattern.

## Procedures

**Step 1: Identify Target PRD**
1. If the user did not provide `[feature-slug]`, scan `./prds/` for folders matching `prd-*`.
2. If only one `prd-*` folder exists, use it automatically. If multiple exist, select the most recently modified.
3. If `./prds/` does not exist or contains no `prd-*` folders, halt: "No PRDs found — run `do-create-prd` first."

**Step 2: Read Tasks**
1. Read `./prds/prd-[feature-slug]/tasks/tasks.md`. If it does not exist, halt: "tasks.md not found — run `do-create-tasks` first."
2. Parse all tasks: `[x]` = completed, `[ ]` = pending.
3. Identify the next pending task (first `[ ]` in order).
4. Check if the next pending task has unmet dependencies (any prior task still `[ ]`).

**Step 3: Check Artifact Integrity (Optional)**
1. For each task marked `[x]`, verify its review file exists at `./prds/prd-[feature-slug]/tasks/[num]_task_review.md`.
2. Flag any `[x]` task missing its review file as incomplete.
3. If the next pending task is currently in progress, read `./prds/prd-[feature-slug]/tasks/[num]_task.md` to show subtask-level progress.

**Step 4: Output Status Report**

Produce a concise report in this format:

```
📦 [Feature Name] (prd-[slug])

Progresso: X/Y tasks concluídas (Z%)

✅ Concluídas: 1, 2, 3
⏳ Próxima:    4 — [título da task]
⏸️  Pendentes: 5, 6, 7 ... Y

⚠️  Artefatos ausentes: [lista de tasks [x] sem review file, se houver]
🚫 Bloqueios: [tasks com dependências não atendidas, se houver]
```

## Output Language
Report in Brazilian Portuguese (PT-BR).

## References
- Tasks index: `./prds/prd-[feature-slug]/tasks/tasks.md`
- Task files: `./prds/prd-[feature-slug]/tasks/[num]_task.md`

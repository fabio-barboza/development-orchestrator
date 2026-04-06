---
name: do-status
description: Shows current progress of a PBI — completed tasks, next pending task, completion percentage, and missing artifacts. Use when the user asks for status, progress, what's next, or wants to resume work on a feature. Read-only skill — does not modify any files. Do not use for task implementation, review, or QA.
---

# Status Report

## Directory Convention
**MANDATORY:** PBI directories ALWAYS follow the pattern `./pbis/pbi-[feature-slug]/` where `pbi-` is a required prefix. Example: feature `user-auth` → directory `./pbis/pbi-user-auth/`. When scanning `./pbis/`, only consider folders matching the `pbi-*` pattern.

## Procedures

**Step 1: Identify Target PBI**
1. If the user did not provide `[feature-slug]`, scan `./pbis/` for folders matching `pbi-*`.
2. If only one `pbi-*` folder exists, use it automatically. If multiple exist, select the most recently modified.
3. If `./pbis/` does not exist or contains no `pbi-*` folders, halt: "No PBIs found — run `do-create-pbi` first."

**Step 2: Read Tasks**
1. Read `./pbis/pbi-[feature-slug]/tasks/tasks.md`. If it does not exist, halt: "tasks.md not found — run `do-create-tasks` first."
2. Parse all tasks: `[x]` = completed, `[ ]` = pending.
3. Identify the next pending task (first `[ ]` in order).
4. Check if the next pending task has unmet dependencies (any prior task still `[ ]`).

**Step 3: Check Artifact Integrity (Optional)**
1. For each task marked `[x]`, verify its review file exists at `./pbis/pbi-[feature-slug]/tasks/[num]_task_review.md`.
2. Flag any `[x]` task missing its review file as incomplete.
3. If the next pending task is currently in progress, read `./pbis/pbi-[feature-slug]/tasks/[num]_task.md` to show subtask-level progress.

**Step 4: Output Status Report**

Produce a concise report in this format:

```
📦 [Feature Name] (pbi-[slug])

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
- Tasks index: `./pbis/pbi-[feature-slug]/tasks/tasks.md`
- Task files: `./pbis/pbi-[feature-slug]/tasks/[num]_task.md`

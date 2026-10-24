---
name: do-shared
description: Shared resources and dependencies used by other do-* skills. Not a user-invocable skill. Contains the MCP capability registry, MCP discovery instructions, project skills/MCPs procedure and service readiness rules referenced by the do-* execution, review, QA and specification skills.
---

# Shared Resources

## Purpose
Centralizes references consumed by multiple `do-*` execution skills. Avoids duplication of the MCP capability registry and the discovery procedure across skills.

## Contents
- `references/do-mcp-capabilities.md`: Registry mapping each configured MCP server to its capabilities, tool prefix, runtime requirements, and unavailability handling.
- `references/do-mcp-discovery-instructions.md`: Standard procedure for discovering MCP servers in the active AI tool's configuration file and applying the capability guard.
- `references/do-project-commands.md`: Standard procedure to obtain the project's own commands (run tests, static checks, start the local service) from the `Project Commands` section or from what the project declares. The framework never assumes a language, package manager or tool.
- `references/do-project-context.md`: Standard procedure for using the skills and MCPs installed in the project (read `Available Skills`/`Available MCPs`, confirm against what the session exposes, pick the relevant ones, record the result). Assumes nothing about which non-`do-*` skills or MCPs exist. Also explains how to locate `do-shared` files (skills-directory-relative paths, hidden directories skipped by glob).
- `references/do-fix-queue.md`: Queue mode of `do-execute-review-fix` and `do-execute-qa-bugfix` (folder or several files): one isolated subagent per file, sequential, a failed item does not stop the others unless the suite is left red.
- `references/do-run-folders.md`: Standard procedure for the sequential round folders of review and QA (`review/review-1/`, `qa/qa-2/`...): how to create a new round, how fix/bugfix skills find their round from the file path, legacy layout handling and the "previous round pending items" section.
- `references/do-service-readiness.md`: Standard procedure for checking, reusing, and starting services (the application under test, an API, a broker, a data store) before invoking MCP tools or E2E tests. Enforces "check first, reuse if running, start only when needed, never kill running services".

## Consumers
The following skills read these references at runtime:
- `do-create-techspec`
- `do-execute-task`
- `do-execute-qa`
- `do-execute-qa-bugfix`
- `do-execute-review`
- `do-execute-review-fix`

## Usage
This skill is **not user-invocable**. Other `do-*` skills cite the absolute path (e.g., `do-shared/references/do-mcp-capabilities.md`) when their procedure requires the registry or the discovery flow.

## Error Handling
- If a consumer skill cannot locate `do-shared/references/do-mcp-capabilities.md`, halt and report the missing dependency — do not fall back to hardcoded MCP assumptions.
- If a new MCP is configured by the user but absent from the registry, add an entry to `do-shared/references/do-mcp-capabilities.md` following the existing schema; no skill code needs to change.

## References
- Project Commands: `do-shared/references/do-project-commands.md`
- Project Skills and MCPs: `do-shared/references/do-project-context.md`
- MCP Registry: `do-shared/references/do-mcp-capabilities.md`
- MCP Discovery: `do-shared/references/do-mcp-discovery-instructions.md`
- Run Folders: `do-shared/references/do-run-folders.md`
- Service Readiness: `do-shared/references/do-service-readiness.md`

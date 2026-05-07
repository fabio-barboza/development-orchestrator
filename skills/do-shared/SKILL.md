---
name: do-shared
description: Shared resources and dependencies used by other do-* skills. Not a user-invocable skill. Contains MCP capability registry and discovery instructions referenced by do-execute-task, do-execute-qa, do-execute-review, and do-execute-qa-bugfix.
---

# Shared Resources

## Purpose
Centralizes references consumed by multiple `do-*` execution skills. Avoids duplication of the MCP capability registry and the discovery procedure across skills.

## Contents
- `references/do-mcp-capabilities.md`: Registry mapping each configured MCP server to its capabilities, tool prefix, runtime requirements, and unavailability handling.
- `references/do-mcp-discovery-instructions.md`: Standard procedure for discovering MCP servers in the active AI tool's configuration file and applying the capability guard.
- `references/do-service-readiness.md`: Standard procedure for checking, reusing, and starting services (frontend dev server, backend API, broker, database) before invoking MCP tools or E2E tests. Enforces "check first, reuse if running, start only when needed, never kill running services".

## Consumers
The following skills read these references at runtime:
- `do-execute-task`
- `do-execute-qa`
- `do-execute-qa-bugfix`
- `do-execute-review`

## Usage
This skill is **not user-invocable**. Other `do-*` skills cite the absolute path (e.g., `do-shared/references/do-mcp-capabilities.md`) when their procedure requires the registry or the discovery flow.

## Error Handling
- If a consumer skill cannot locate `do-shared/references/do-mcp-capabilities.md`, halt and report the missing dependency — do not fall back to hardcoded MCP assumptions.
- If a new MCP is configured by the user but absent from the registry, add an entry to `do-shared/references/do-mcp-capabilities.md` following the existing schema; no skill code needs to change.

## References
- MCP Registry: `do-shared/references/do-mcp-capabilities.md`
- MCP Discovery: `do-shared/references/do-mcp-discovery-instructions.md`
- Service Readiness: `do-shared/references/do-service-readiness.md`

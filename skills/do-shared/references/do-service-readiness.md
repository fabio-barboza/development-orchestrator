# Service Readiness Procedure

Standard procedure referenced by execution skills (`do-execute-task`, `do-execute-review`, `do-execute-qa`) whenever an MCP, automated test, or validation step requires a running service (the application under test, an API, a broker, a data store, a cache, etc.).

**Rule of thumb:** check first, reuse if running, start only when needed, never kill services that the user might be using.

## When to Apply

Apply this procedure before invoking any tool/test that depends on a running service:

- An MCP whose capability (see `do-shared/references/do-mcp-capabilities.md`) requires a running target (`ui-testing`, `api-testing`, `messaging`, `data-store`).
- End-to-end tests (user-interface or service) that hit an HTTP endpoint, queue, data store, or cache.
- Integration tests that require external infrastructure.

If the current step uses only unit tests, static checks, or static analysis, **skip this procedure**.

## Step A: Identify Required Services

1. List the services that the upcoming step depends on. Sources of truth:
   - The capability table at `do-shared/references/do-mcp-capabilities.md` (`Requer alvo em execução`).
   - The project's own configuration for ports and start commands: the `Project Commands` section of `.do/context.md` (see `do-shared/references/do-project-commands.md`) and the project's declared run/infrastructure files.
   - Task/PRD/TechSpec context (a feature with a user interface → the application serving it; a feature exposing a service → that service and any broker, data store or cache it mentions).
2. For each required service, determine:
   - **Type**: `dev-server` (the project's own application, safe to auto-start) or `external` (broker, data store, cache, third-party API — never auto-start).
   - **Expected port or address** (when known): read from the project's configuration or the `Project Commands` section of `.do/context.md`; never assume a port by technology.
   - **Health check** (when available): an HTTP probe, a health tool exposed by the MCP, or a process probe.

## Step B: Check If Already Running

For each required service, in this order:

1. **Port probe** (preferred when port is known): run `lsof -ti :<port>` (Linux/macOS) or `ss -ltn 'sport = :<port>'`. A non-empty result means the port is bound.
2. **HTTP health probe** (when applicable): `curl -fsS -o /dev/null -w "%{http_code}" http://localhost:<port>/<health-path>` and accept any 2xx/3xx response.
3. **MCP health probe** (when the MCP exposes a health/status tool): use that tool.
4. **Process probe** (last resort): `pgrep -f "<process-pattern>"`, using a fragment of the project's own start command.

Record each service as `running` or `not-running`. **Never kill a process that is already running** — the user may be using it.

## Step C: Start Only What Is Missing

For each service marked `not-running`:

1. **If type is `dev-server`** — start it with the project's own local start command, obtained per `do-shared/references/do-project-commands.md` (the `Iniciar o serviço local` row or what the project declares). Use only a command the project itself defines for local development; never invent one. For multi-package projects, use the command scoped to the package under test.

   Start the process in the background so the tool call returns immediately — a dev server never exits, so a foreground call blocks the session. Use your AI tool's native background mechanism when available:
   - **Opencode**: `shell` tool with `background: true`
   - **Claude Code**: `Bash` tool with `run_in_background: true`
   - **GitHub Copilot**: `run_in_terminal` with `isBackground: true`
   - **Cursor**: `run_terminal_cmd` with `is_background: true`
   - **Otherwise**: `nohup <command> > <log> 2>&1 & echo $! > <scratch folder>/<service>.pid`, with the working directory set before (the tool's workdir option, or a `cd <dir>` in its own statement). **Never** wrap the start in a subshell such as `(cd <dir> && <command> &)`: it hides the PID and leaves you unable to stop the tree.

   Whatever the mechanism, keep the handle or the PID of every service you start; Step E needs it.

   If you redirect output to a log file, write it inside your skill's scratch folder (e.g. `./.do/tmp/task-[num]/<service>.log`; if your skill defines none, `./.do/tmp/<skill-name>/`) — never in `/tmp` or any path outside the project root.

   Then re-run Step B until the service responds or a 60-second timeout elapses. If the service fails to start (compile error, port already taken by an unrelated process, missing env var), do NOT modify code or configuration to work around it — report the failure (each consumer skill defines how: bug file, review finding, or HALT).

2. **If type is `external`** (broker, database, cache, third-party API) — do NOT attempt to start. Document the gap in the consumer skill's report (review report, QA report, or task review) using the `Se indisponível` wording of the capability in `do-shared/references/do-mcp-capabilities.md`. Continue with whichever tests do not require that service.

## Step D: Verify Before Use

1. After Step C, re-run the health probe from Step B for each service that was started.
2. Only proceed to invoke MCP tools / run E2E tests once every required service is `running`.
3. If any service is still `not-running`, fall back to the consumer skill's gap-handling rules — never invoke an MCP that requires an unavailable service.

## Step E: Cleanup Policy

1. Services that were **already running** when Step B executed must remain running after the consumer skill finishes — they belong to the user.
2. Services that were **started by Step C** belong to this run: stop them before the final response, never processes you did not start. Leaving dev servers running keeps ports busy and corrupts the next run.
   - A start command usually creates a **tree** of processes (a package-manager wrapper, a file-watcher/supervisor, the server itself). Stopping only the process that holds the port leaves the supervisor alive, and it can respawn the server.
   - Stop the whole tree you started: use the stop/kill facility of the background handle your tool gave you when starting, or the PID you recorded and its children (`pkill -P <pid>` before `kill <pid>`, or the process group). Record the PID when you start the service.
   - If you have no handle or PID, find the process holding the port, then walk up its parents only while the parent's command line matches the start command **you** ran; stop that top-most matching process and its children. Never go past it and never touch a process that predates your start.
   - **Verify**: re-run the port probe (Step B.1) and `pgrep -f "<fragment of the start command>"`. Both must be empty (or contain only processes that existed before Step C) before you answer.
3. Outside this cleanup of your own started services, **never** issue `kill`, `pkill`, or `killall` against dev-server or external-service processes — that is destructive and out of scope for this procedure.

## Error Handling

- **Port bound by an unrelated process:** report the conflict to the user via the consumer skill's report; do NOT terminate the conflicting process.
- **Dev server starts but health check fails after timeout:** treat as startup failure; the consumer skill defines the next step (HALT, bug file, or review finding).
- **MCP health tool itself fails:** treat the MCP as unavailable and follow the `Se indisponível` handling of its capability in `do-shared/references/do-mcp-capabilities.md`.
- **Multiple candidate start commands:** pick the one the project documents as primary; never run more than one starter for the same service.

# Service Readiness Procedure

Standard procedure referenced by execution skills (`do-execute-task`, `do-execute-review`, `do-execute-qa`) whenever an MCP, automated test, or validation step requires a running service (frontend dev server, backend API, broker, database, cache, etc.).

**Rule of thumb:** check first, reuse if running, start only when needed, never kill services that the user might be using.

## When to Apply

Apply this procedure before invoking any tool/test that depends on a running service:

- An MCP whose registry entry has `Requer app rodando: Sim` (e.g., `playwright`, `rabbitmq`).
- E2E tests (browser-based or backend) that hit an HTTP endpoint, queue, database, or cache.
- Integration tests that require external infrastructure.

If the current step uses only unit tests, typecheck, lint, or static analysis, **skip this procedure**.

## Step A: Identify Required Services

1. List the services that the upcoming step depends on. Sources of truth:
   - MCP registry entry at `do-shared/references/do-mcp-capabilities.md` (`Requer app rodando` field).
   - Project configuration (`package.json` scripts, `docker-compose.yml`, `.env`) for known ports.
   - Task/PRD/TechSpec context (frontend feature → frontend dev server; backend feature → API server and any broker/database mentioned).
2. For each required service, determine:
   - **Type**: `dev-server` (safe to auto-start) or `external` (broker, database, cache — never auto-start).
   - **Expected port** (when known): e.g., `3000` for Next.js, `5173` for Vite, `5672` for RabbitMQ AMQP, `15672` for RabbitMQ HTTP, `5432` for Postgres, `6379` for Redis.
   - **Health check command** (when available): a curl/HTTP probe, MCP `connection.health`, or process probe.

## Step B: Check If Already Running

For each required service, in this order:

1. **Port probe** (preferred when port is known): run `lsof -ti :<port>` (Linux/macOS) or `ss -ltn 'sport = :<port>'`. A non-empty result means the port is bound.
2. **HTTP health probe** (when applicable): `curl -fsS -o /dev/null -w "%{http_code}" http://localhost:<port>/<health-path>` and accept any 2xx/3xx response.
3. **MCP health probe** (when the MCP exposes one): use the registry-listed health tool (e.g., `mcp__rabbitmq__connection.health`).
4. **Process probe** (last resort): `pgrep -f "<process-pattern>"` (e.g., `"npm run dev"`, `"vite"`, `"bun dev"`).

Record each service as `running` or `not-running`. **Never kill a process that is already running** — the user may be using it.

## Step C: Start Only What Is Missing

For each service marked `not-running`:

1. **If type is `dev-server`** — attempt to start using known-safe commands, picking the first that matches the project:
   - `bun dev` (when `bun.lockb` exists and a `dev` script is defined)
   - `pnpm dev` (when `pnpm-lock.yaml` exists)
   - `npm run dev` or `npm start` (default fallback)
   - For monorepos, prefer the workspace-scoped command (e.g., `bun --filter <package> dev`).

   Start the process in the background, then re-run Step B until the service responds or a 60-second timeout elapses. If the service fails to start (compile error, port already taken by an unrelated process, missing env var), do NOT modify code or configuration to work around it — report the failure (each consumer skill defines how: bug file, review finding, or HALT).

2. **If type is `external`** (broker, database, cache, third-party API) — do NOT attempt to start. Document the gap in the consumer skill's report (review report, QA report, or task review) using the wording from the MCP registry's `Se indisponivel` field. Continue with whichever tests do not require that service.

## Step D: Verify Before Use

1. After Step C, re-run the health probe from Step B for each service that was started.
2. Only proceed to invoke MCP tools / run E2E tests once every required service is `running`.
3. If any service is still `not-running`, fall back to the consumer skill's gap-handling rules — never invoke an MCP that requires an unavailable service.

## Step E: Cleanup Policy

1. Services that were **already running** when Step B executed must remain running after the consumer skill finishes — they belong to the user.
2. Services that were **started by Step C** may be left running for follow-up work; explicit shutdown is not required unless the consumer skill states otherwise.
3. **Never** issue `kill`, `pkill`, or `killall` against dev-server or external-service processes as part of routine readiness — that is destructive and out of scope for this procedure.

## Error Handling

- **Port bound by an unrelated process:** report the conflict to the user via the consumer skill's report; do NOT terminate the conflicting process.
- **Dev server starts but health check fails after timeout:** treat as startup failure; the consumer skill defines the next step (HALT, bug file, or review finding).
- **MCP health tool itself fails:** treat the MCP as unavailable and follow its `Se indisponivel` handling from `do-shared/references/do-mcp-capabilities.md`.
- **Multiple candidate start commands:** pick exactly one based on the lock file; never run more than one dev-server starter for the same package.

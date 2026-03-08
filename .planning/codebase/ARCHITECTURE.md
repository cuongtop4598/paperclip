# Architecture

**Analysis Date:** 2026-03-08

## Pattern Overview

**Overall:** Monorepo with layered server-client architecture and pluggable adapter system

**Key Characteristics:**
- pnpm workspace monorepo with 7 workspace packages (`server`, `ui`, `cli`, `packages/*`, `packages/adapters/*`)
- Express REST API server with WebSocket realtime layer
- React SPA frontend communicating via `/api` prefix
- Adapter plugin system for multiple AI coding agent runtimes (Claude, Cursor, Codex, OpenCode, OpenClaw, Pi)
- Multi-tenant company model with agent hierarchy and RBAC
- Two deployment modes: `local_trusted` (implicit auth) and `authenticated` (BetterAuth sessions)

## Layers

**Database Layer (`packages/db`):**
- Purpose: Schema definitions, migrations, and DB client creation
- Location: `packages/db/src/`
- Contains: Drizzle ORM schema tables, migration files, DB factory
- Depends on: `drizzle-orm`, PostgreSQL (external or embedded)
- Used by: `server`, `cli`
- Key files: `packages/db/src/schema/index.ts` (barrel), `packages/db/src/schema/agents.ts`, `packages/db/src/schema/companies.ts`, `packages/db/src/schema/issues.ts`

**Shared Types & Validators (`packages/shared`):**
- Purpose: Shared TypeScript types, Zod validation schemas, constants
- Location: `packages/shared/src/`
- Contains: Types (`types/`), validators (`validators/`), constants, URL-key utilities
- Depends on: `zod`
- Used by: `server`, `ui`, `cli`, adapter packages
- Key files: `packages/shared/src/index.ts`, `packages/shared/src/types/`, `packages/shared/src/validators/`

**Adapter Utils (`packages/adapter-utils`):**
- Purpose: Shared adapter interfaces and execution types
- Location: `packages/adapter-utils/src/`
- Contains: `AdapterAgent`, `AdapterRuntime`, `AdapterExecutionResult`, `ServerAdapterModule` type definitions
- Depends on: nothing (pure types)
- Used by: all adapter packages, `server`
- Key files: `packages/adapter-utils/src/types.ts`, `packages/adapter-utils/src/server-utils.ts`

**Adapter Packages (`packages/adapters/*`):**
- Purpose: Pluggable AI agent runtime integrations
- Location: `packages/adapters/claude-local/`, `packages/adapters/cursor-local/`, `packages/adapters/openclaw-gateway/`, etc.
- Contains: Each adapter has `src/cli/` (CLI entry), `src/server/` (execution logic), `src/ui/` (UI config components), `src/index.ts` (shared metadata/models)
- Depends on: `@paperclipai/adapter-utils`, `@paperclipai/shared`
- Used by: `server` (execution), `ui` (config forms), `cli` (agent setup)
- Available adapters: `claude-local`, `cursor-local`, `codex-local`, `opencode-local`, `openclaw`, `openclaw-gateway`, `pi-local`

**Server (`server`):**
- Purpose: Express REST API, WebSocket realtime, auth, storage, heartbeat scheduler
- Location: `server/src/`
- Contains: Routes, services, middleware, adapters registry, auth, storage providers, secrets providers
- Depends on: `@paperclipai/db`, `@paperclipai/shared`, `@paperclipai/adapter-utils`, all adapter packages, `express`, `drizzle-orm`, `better-auth`, `ws`, `pino`
- Used by: `ui` (via HTTP API), `cli` (via HTTP client), agents (via Bearer token API)
- Key files: `server/src/index.ts` (entry), `server/src/app.ts` (Express app factory), `server/src/config.ts`

**UI (`ui`):**
- Purpose: React SPA dashboard for managing agents, projects, issues, approvals
- Location: `ui/src/`
- Contains: Pages, components, API client layer, React context providers, adapter UI modules
- Depends on: `react`, `react-router-dom`, `@tanstack/react-query`, `@radix-ui`, `tailwindcss`, `@paperclipai/shared`, adapter packages
- Used by: End users (board members) via browser
- Key files: `ui/src/main.tsx` (entry), `ui/src/App.tsx` (routing), `ui/src/api/client.ts`

**CLI (`cli`):**
- Purpose: Command-line tool for setup, diagnostics, agent management, and API interaction
- Location: `cli/src/`
- Contains: Commands (`commands/`), config management (`config/`), API client (`client/`), adapters, checks
- Depends on: `commander`, `@paperclipai/shared`
- Used by: Operators/developers
- Key files: `cli/src/index.ts` (entry with all command registrations)

## Data Flow

**Board User Request (UI to API):**

1. UI component calls API function from `ui/src/api/*.ts`
2. `ui/src/api/client.ts` issues `fetch()` to `/api/*` with credentials
3. Express middleware chain: `httpLogger` -> `privateHostnameGuard` -> `actorMiddleware` -> `boardMutationGuard` -> route handler
4. `actorMiddleware` (`server/src/middleware/auth.ts`) resolves actor identity (local_trusted: implicit board user; authenticated: BetterAuth session lookup)
5. Route handler in `server/src/routes/*.ts` calls service function from `server/src/services/*.ts`
6. Service queries/mutates DB via Drizzle ORM against `@paperclipai/db` schema
7. Service optionally publishes live event via `publishLiveEvent()` from `server/src/services/live-events.ts`
8. JSON response returned to UI

**Agent API Request:**

1. Agent sends HTTP request with `Authorization: Bearer <token>` header
2. `actorMiddleware` resolves actor as `type: "agent"` via API key hash lookup in `agent_api_keys` table or JWT verification (`server/src/agent-auth-jwt.ts`)
3. Route handler checks agent permissions via `server/src/routes/authz.ts`
4. Service layer processes request

**Heartbeat (Agent Execution Cycle):**

1. Server heartbeat scheduler (`server/src/index.ts`, interval-based) calls `heartbeatService.tickTimers()`
2. Tick enqueues heartbeat runs for agents with active timers
3. Adapter `execute()` is called via the adapter registry (`server/src/adapters/registry.ts`)
4. Adapter spawns/communicates with the AI coding tool (Claude CLI, Cursor, etc.)
5. Execution result (tokens, cost, session state) is recorded as `heartbeat_run_events`
6. Live events broadcast to connected WebSocket clients

**Realtime Updates (WebSocket):**

1. Client connects to WebSocket at upgrade path in `server/src/realtime/live-events-ws.ts`
2. Server authenticates the WebSocket connection (session cookie or Bearer token)
3. Server subscribes client to company-scoped event stream via `subscribeCompanyLiveEvents()`
4. Service layer calls `publishLiveEvent()` on mutations
5. `LiveUpdatesProvider` (`ui/src/context/LiveUpdatesProvider.tsx`) receives events and invalidates React Query caches

**State Management (UI):**
- Server state: `@tanstack/react-query` with 30s stale time, refetch-on-focus
- Global UI state: React context providers stacked in `ui/src/main.tsx` (Company, LiveUpdates, Breadcrumb, Panel, Sidebar, Dialog, Toast, Theme)
- URL state: `react-router-dom` with company prefix routing (`/:companyPrefix/*`)

## Key Abstractions

**Actor Model:**
- Purpose: Unified identity for request authorization
- Files: `server/src/middleware/auth.ts`, `server/src/types/express.d.ts`
- Pattern: Every request gets `req.actor` with `type: "board" | "agent" | "none"`, plus metadata like `companyId`, `userId`, `agentId`, `runId`

**Service Layer:**
- Purpose: Business logic functions that operate on the database
- Files: `server/src/services/*.ts`, barrel at `server/src/services/index.ts`
- Pattern: Each service is a factory function that takes `db: Db` and returns an object with methods. Example: `agentService(db)` returns `{ list, get, create, update, ... }`

**Adapter System:**
- Purpose: Pluggable AI agent runtime integrations
- Files: `server/src/adapters/registry.ts`, `packages/adapter-utils/src/types.ts`, `packages/adapters/*/`
- Pattern: Each adapter implements `ServerAdapterModule` interface with `execute()`, `testEnvironment()`, optional `sessionCodec`, `onHireApproved()`. Registry maps adapter type strings to modules. Each adapter also exports UI config components and CLI setup helpers.

**Storage Providers:**
- Purpose: Pluggable file storage (local disk or S3)
- Files: `server/src/storage/types.ts`, `server/src/storage/local-disk-provider.ts`, `server/src/storage/s3-provider.ts`, `server/src/storage/provider-registry.ts`
- Pattern: `StorageService` interface with `upload()`, `download()`, `delete()` methods. Provider selected via config.

**Secrets Providers:**
- Purpose: Pluggable secrets encryption/storage
- Files: `server/src/secrets/types.ts`, `server/src/secrets/local-encrypted-provider.ts`, `server/src/secrets/provider-registry.ts`, `server/src/secrets/external-stub-providers.ts`
- Pattern: Provider interface for encrypt/decrypt of company secrets. Default is `local_encrypted` using a master key file.

**HttpError Pattern:**
- Purpose: Typed HTTP errors thrown from services/routes
- Files: `server/src/errors.ts`
- Pattern: `HttpError` class with factory functions `badRequest()`, `unauthorized()`, `forbidden()`, `notFound()`, `conflict()`, `unprocessable()`. Caught by `errorHandler` middleware (`server/src/middleware/error-handler.ts`).

## Entry Points

**Server Entry:**
- Location: `server/src/index.ts`
- Triggers: `pnpm dev:server` / `tsx src/index.ts` / `node dist/index.js`
- Responsibilities: Load config, initialize DB (external Postgres or embedded), run migrations, configure auth, create Express app, start HTTP server, setup WebSocket, start heartbeat scheduler, start backup scheduler

**UI Entry:**
- Location: `ui/src/main.tsx`
- Triggers: Browser loads SPA (served by server as static or via Vite dev middleware)
- Responsibilities: Mount React app with provider tree, render router

**CLI Entry:**
- Location: `cli/src/index.ts`
- Triggers: `pnpm paperclipai <command>`
- Responsibilities: Parse CLI commands via `commander`, dispatch to command handlers

**Dev Runner:**
- Location: `scripts/dev-runner.mjs`
- Triggers: `pnpm dev` / `pnpm dev:watch`
- Responsibilities: Orchestrates building shared packages then starting server with Vite UI middleware

## Error Handling

**Strategy:** Throw `HttpError` instances from services/routes; catch in global error handler middleware

**Patterns:**
- Services throw `HttpError` via factory functions from `server/src/errors.ts`
- `server/src/middleware/error-handler.ts` catches errors, logs them, returns JSON `{ error: message }`
- Route handlers use `assertBoard()` and `assertCompanyAccess()` from `server/src/routes/authz.ts` for authorization guards
- UI `ApiError` class in `ui/src/api/client.ts` wraps non-OK fetch responses with status and body
- Validation middleware (`server/src/middleware/validate.ts`) validates request bodies against Zod schemas from `@paperclipai/shared`

## Cross-Cutting Concerns

**Logging:** `pino` logger via `server/src/middleware/logger.ts`; HTTP request logging via `pino-http`
**Validation:** Zod schemas in `packages/shared/src/validators/`, applied via `validate()` middleware in routes
**Authentication:** Dual-mode: `local_trusted` (implicit board user) or `authenticated` (BetterAuth with session cookies + agent API keys/JWTs)
**Authorization:** `actorMiddleware` sets `req.actor`; route-level guards via `assertBoard()`, `assertCompanyAccess()` in `server/src/routes/authz.ts`; agent permissions checked per-operation

---

*Architecture analysis: 2026-03-08*

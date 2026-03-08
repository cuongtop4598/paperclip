# External Integrations

**Analysis Date:** 2026-03-08

## AI Agent Adapters

Paperclip orchestrates multiple AI coding agents through a pluggable adapter system. Each adapter lives in `packages/adapters/` and is registered in `server/src/adapters/registry.ts`.

**Claude Code (local):**
- Adapter: `@paperclipai/adapter-claude-local` at `packages/adapters/claude-local/`
- Type: `claude_local`
- Runtime: Spawns `@anthropic-ai/claude-code` CLI locally
- Auth: `ANTHROPIC_API_KEY` env var
- Supports local agent JWT auth

**OpenAI Codex (local):**
- Adapter: `@paperclipai/adapter-codex-local` at `packages/adapters/codex-local/`
- Type: `codex_local`
- Runtime: Spawns `@openai/codex` CLI locally
- Auth: `OPENAI_API_KEY` env var
- Supports local agent JWT auth

**Cursor (local):**
- Adapter: `@paperclipai/adapter-cursor-local` at `packages/adapters/cursor-local/`
- Type: `cursor`
- Supports local agent JWT auth

**OpenCode (local):**
- Adapter: `@paperclipai/adapter-opencode-local` at `packages/adapters/opencode-local/`
- Type: `opencode_local`
- Dynamic model discovery via `listOpenCodeModels`
- Supports local agent JWT auth

**Pi (local):**
- Adapter: `@paperclipai/adapter-pi-local` at `packages/adapters/pi-local/`
- Type: `pi_local`
- Dynamic model discovery via `listPiModels`
- Supports local agent JWT auth

**OpenClaw (remote):**
- Adapter: `@paperclipai/adapter-openclaw` at `packages/adapters/openclaw/`
- Type: `openclaw`
- Remote execution (does not use local agent JWT)
- Has `onHireApproved` lifecycle hook

**OpenClaw Gateway:**
- Adapter: `@paperclipai/adapter-openclaw-gateway` at `packages/adapters/openclaw-gateway/`
- Type: `openclaw_gateway`
- Uses WebSocket (`ws` package) for communication
- Remote execution (does not use local agent JWT)

**Generic Process Adapter:**
- Registered at `server/src/adapters/process/index.ts`
- Fallback for unknown adapter types

**Generic HTTP Adapter:**
- Registered at `server/src/adapters/http/index.ts`

## Data Storage

**Database:**
- PostgreSQL 17
- ORM: Drizzle ORM (`drizzle-orm` 0.38.4) with `postgres` driver 3.4.5
- Client: `packages/db/src/client.ts`
- Schema: `packages/db/src/schema/` (35+ table definition files)
- Migrations: `packages/db/src/migrations/` (Drizzle Kit generated)
- Migration tool: `drizzle-kit` 0.31.9 (`packages/db/package.json`)
- Connection: `DATABASE_URL` env var
- Embedded mode: `embedded-postgres` 18.1.0-beta.16 for zero-config local development (auto-initialized in `server/src/index.ts`)
- Embedded port: default 54329, auto-detected if busy
- Automated backups: configurable interval/retention, implementation in `packages/db/src/backup-lib.ts`

**File Storage:**
- Provider pattern: `server/src/storage/provider-registry.ts`
- Two providers:
  - `local_disk` (default): `server/src/storage/local-disk-provider.ts`
  - `s3`: `server/src/storage/s3-provider.ts` using `@aws-sdk/client-s3`
- S3 config env vars: `PAPERCLIP_STORAGE_S3_BUCKET`, `PAPERCLIP_STORAGE_S3_REGION`, `PAPERCLIP_STORAGE_S3_ENDPOINT`, `PAPERCLIP_STORAGE_S3_PREFIX`, `PAPERCLIP_STORAGE_S3_FORCE_PATH_STYLE`
- Service layer: `server/src/storage/service.ts`

**Caching:**
- None (no Redis or in-memory cache layer detected)

## Secrets Management

**Provider pattern:** `server/src/secrets/provider-registry.ts`

**Active provider:**
- `local_encrypted` (default): `server/src/secrets/local-encrypted-provider.ts`
  - Master key file at `PAPERCLIP_SECRETS_MASTER_KEY_FILE`
  - Strict mode toggle: `PAPERCLIP_SECRETS_STRICT_MODE`

**Stub providers (not yet implemented):**
- `aws_secrets_manager` - AWS Secrets Manager (throws "not configured")
- `gcp_secret_manager` - GCP Secret Manager (throws "not configured")
- `vault` - HashiCorp Vault (throws "not configured")
- All stubs defined in `server/src/secrets/external-stub-providers.ts`

## Authentication & Identity

**Auth Provider:** Better Auth 1.4.18
- Implementation: `server/src/auth/better-auth.ts`
- Uses Drizzle adapter with PostgreSQL backend
- Email/password enabled, email verification disabled
- Tables: `authUsers`, `authSessions`, `authAccounts`, `authVerifications` (defined in `packages/db/src/schema/auth.ts`)
- Secret: `BETTER_AUTH_SECRET` env var (required in authenticated mode)
- Trusted origins derived from config + `BETTER_AUTH_TRUSTED_ORIGINS` env var

**Deployment Modes:**
- `local_trusted` (default): No auth required, auto-creates local board user, loopback-only binding
- `authenticated`: Better Auth with email/password, hostname guard, board claim flow

**Agent Authentication:**
- Custom JWT-based auth for local agents: `server/src/agent-auth-jwt.ts`
- HMAC-SHA256 signing
- JWT claims include: `sub` (agent ID), `company_id`, `adapter_type`, `run_id`
- Secret: `PAPERCLIP_AGENT_JWT_SECRET` env var

**Authorization:**
- Actor middleware: `server/src/middleware/auth.ts`
- Board mutation guard: `server/src/middleware/board-mutation-guard.ts`
- Private hostname guard: `server/src/middleware/private-hostname-guard.ts`
- Permission system: `server/src/services/agent-permissions.ts`
- Instance user roles: `packages/db/src/schema/instance_user_roles.ts`
- Company memberships: `packages/db/src/schema/company_memberships.ts`

## Realtime

**WebSocket Server:**
- Implementation: `server/src/realtime/live-events-ws.ts`
- Library: `ws` 8.19.0
- Mounted on HTTP server upgrade
- Provides company-scoped live event subscriptions
- Used by: `server/src/services/live-events.ts`

## Monitoring & Observability

**Error Tracking:**
- None (no Sentry, Datadog, etc.)

**Logging:**
- Pino 9.6.0 structured JSON logger
- HTTP request logging via `pino-http` 10.4.0
- Pretty printing via `pino-pretty` 13.1.3
- Logger instance: `server/src/middleware/logger.ts`

**Health Checks:**
- Health route: `server/src/routes/health.ts`
- Reports deployment mode, exposure, auth readiness

**Heartbeat System:**
- Scheduler: `server/src/services/heartbeat.ts`
- Configurable interval (default 30s)
- Reaps orphaned runs, ticks timers
- DB tables: `packages/db/src/schema/heartbeat_runs.ts`, `packages/db/src/schema/heartbeat_run_events.ts`

## CI/CD & Deployment

**Hosting:**
- Docker-based deployment (self-hosted)
- `Dockerfile` with multi-stage build
- Default port: 3100

**CI Pipeline:**
- GitHub Actions
- Workflows in `.github/workflows/`:
  - `pr-policy.yml` - PR policy checks
  - `pr-verify.yml` - PR verification (lint, type check, test)
  - `refresh-lockfile.yml` - Lockfile maintenance

**Release Process:**
- Changesets for version management
- `scripts/release.sh` for release automation
- `scripts/build-npm.sh` for npm package builds
- Published packages: `@paperclipai/server`, `@paperclipai/db`, `@paperclipai/shared`, `@paperclipai/adapter-utils`, all adapters, `paperclipai` CLI

**Documentation:**
- Mintlify docs at `docs/`
- Dev command: `npx mintlify dev`
- Config: `docs/docs.json`

## Environment Configuration

**Required env vars (authenticated mode):**
- `BETTER_AUTH_SECRET` - Auth signing secret
- `PAPERCLIP_DEPLOYMENT_MODE` - Must be `authenticated`

**Required env vars (with external Postgres):**
- `DATABASE_URL` - PostgreSQL connection string

**Optional env vars:**
- `PORT` - Server port (default: 3100)
- `HOST` - Bind host (default: 127.0.0.1)
- `SERVE_UI` - Serve built frontend (default: true)
- `PAPERCLIP_PUBLIC_URL` - Public-facing URL
- `PAPERCLIP_HOME` - Data directory base path
- `PAPERCLIP_STORAGE_PROVIDER` - `local_disk` or `s3`
- `PAPERCLIP_SECRETS_PROVIDER` - Secret backend
- `ANTHROPIC_API_KEY` - For Claude adapter
- `OPENAI_API_KEY` - For Codex adapter
- `PAPERCLIP_AGENT_JWT_SECRET` - Agent JWT signing
- `PAPERCLIP_DB_BACKUP_ENABLED` - Enable automated backups
- `PAPERCLIP_DB_BACKUP_INTERVAL_MINUTES` - Backup interval (default: 60)
- `PAPERCLIP_DB_BACKUP_RETENTION_DAYS` - Backup retention (default: 30)

**Secrets location:**
- `.env` file in project root (git-ignored)
- Paperclip home directory env file (resolved at runtime)

## Webhooks & Callbacks

**Incoming:**
- `/api/auth/*` - Better Auth callback routes (email/password sign-up/sign-in)
- WebSocket upgrade on `/ws/live-events` path (inferred from live-events-ws.ts)

**Outgoing:**
- OpenClaw adapter communicates with remote OpenClaw service
- OpenClaw Gateway adapter uses WebSocket to remote gateway

## LLM Reflection Endpoint

**`/llms/agent-configuration.txt`:**
- Route: `server/src/routes/llms.ts`
- Serves adapter configuration documentation for agent self-discovery
- Accessible by board users and agents with `canCreateAgents` permission

---

*Integration audit: 2026-03-08*

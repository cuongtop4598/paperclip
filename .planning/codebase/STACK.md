# Technology Stack

**Analysis Date:** 2026-03-08

## Languages

**Primary:**
- TypeScript 5.7.3 - All application code (server, UI, CLI, packages)

**Secondary:**
- Bash - Build scripts, smoke tests (`scripts/`, `scripts/smoke/`)
- SQL - Database migrations (`packages/db/src/migrations/`)

## Runtime

**Environment:**
- Node.js >= 20 (Docker uses `node:lts-trixie-slim`)
- ES Modules throughout (`"type": "module"` in all package.json files)
- Target: ES2023 (`tsconfig.json`)

**Package Manager:**
- pnpm 9.15.4 (declared via `packageManager` field in root `package.json`)
- Lockfile: `pnpm-lock.yaml` (present)
- Workspace protocol: `workspace:*` for internal dependencies

## Frameworks

**Core:**
- Express 5.1.0 - HTTP server (`server/package.json`)
- React 19.0.0 - Frontend UI (`ui/package.json`)
- React Router DOM 7.1.5 - Client-side routing (`ui/package.json`)

**Testing:**
- Vitest 3.0.5 - Test runner (root, server, UI, CLI)
- Supertest 7.0.0 - HTTP assertion testing (`server/package.json`)

**Build/Dev:**
- Vite 6.1.0 - UI bundler and dev server (`ui/package.json`)
- esbuild 0.27.3 - CLI bundler (`cli/esbuild.config.mjs`)
- tsc - TypeScript compiler for server and packages
- tsx 4.19.2 - TypeScript execution for dev mode (`server/`, `cli/`, `packages/db/`)

## Monorepo Structure

**Workspace layout** (defined in `pnpm-workspace.yaml`):
- `packages/*` - Shared libraries (db, shared, adapter-utils)
- `packages/adapters/*` - AI agent adapters (claude-local, codex-local, cursor-local, openclaw, openclaw-gateway, opencode-local, pi-local)
- `server` - Express API server (`@paperclipai/server`)
- `ui` - React frontend (`@paperclipai/ui`)
- `cli` - CLI tool (`paperclipai`)

**Release Management:**
- Changesets (`@changesets/cli` 2.30.0) for versioning
- Config in `.changeset/config.json`
- Current version: 0.2.7

## Key Dependencies

**Critical:**
- `drizzle-orm` 0.38.4 - Database ORM (`packages/db/`, `server/`, `cli/`)
- `postgres` 3.4.5 - PostgreSQL driver (`packages/db/`)
- `better-auth` 1.4.18 - Authentication framework (`server/`)
- `zod` 3.24.2 - Schema validation (`packages/shared/`, `server/`)
- `ws` 8.19.0 - WebSocket server for realtime events (`server/`)
- `commander` 13.1.0 - CLI framework (`cli/`)

**Infrastructure:**
- `embedded-postgres` 18.1.0-beta.16 - Embedded PostgreSQL for zero-config local dev (`server/`)
- `@aws-sdk/client-s3` 3.888.0 - S3-compatible file storage (`server/`)
- `pino` 9.6.0 / `pino-http` 10.4.0 / `pino-pretty` 13.1.3 - Structured logging (`server/`)
- `detect-port` 2.1.0 - Dynamic port detection (`server/`)
- `multer` 2.0.2 - File upload middleware (`server/`)
- `dotenv` 17.0.1 - Environment variable loading (`server/`, `cli/`)

**UI:**
- `@tanstack/react-query` 5.90.21 - Server state management
- `radix-ui` 1.4.3 / `@radix-ui/react-slot` 1.2.4 - Headless UI primitives
- `tailwindcss` 4.0.7 / `@tailwindcss/vite` 4.0.7 - Utility-first CSS
- `@tailwindcss/typography` 0.5.19 - Prose styling
- `class-variance-authority` 0.7.1 / `clsx` 2.1.1 / `tailwind-merge` 3.4.1 - Class utilities
- `lucide-react` 0.574.0 - Icon library
- `@mdxeditor/editor` 3.52.4 - Rich text/MDX editing
- `react-markdown` 10.1.0 / `remark-gfm` 4.0.1 - Markdown rendering
- `mermaid` 11.12.0 - Diagram rendering
- `cmdk` 1.1.1 - Command palette
- `@dnd-kit/core` 6.3.1 / `@dnd-kit/sortable` 10.0.0 - Drag and drop
- `@clack/prompts` 0.10.0 - CLI interactive prompts (`cli/`)
- `picocolors` 1.1.1 - Terminal coloring (`cli/`, adapters)

## Configuration

**Environment:**
- `.env` file present (loaded via dotenv)
- `.env.example` provides template: `DATABASE_URL`, `PORT`, `SERVE_UI`
- Additional Paperclip-specific env path resolved at `server/src/paths.ts`
- JSON config file support via `server/src/config-file.ts`
- Config merges env vars with file-based config, env vars take precedence (`server/src/config.ts`)

**Key environment variables (from `server/src/config.ts`):**
- `DATABASE_URL` - External PostgreSQL connection string (optional; falls back to embedded)
- `PORT` - Server port (default: 3100)
- `HOST` - Bind host (default: 127.0.0.1)
- `SERVE_UI` - Serve built UI from server (default: true)
- `BETTER_AUTH_SECRET` - Auth secret (required in authenticated mode)
- `PAPERCLIP_DEPLOYMENT_MODE` - `local_trusted` or `authenticated`
- `PAPERCLIP_DEPLOYMENT_EXPOSURE` - `private` or `public`
- `PAPERCLIP_STORAGE_PROVIDER` - `local_disk` or `s3`
- `PAPERCLIP_SECRETS_PROVIDER` - `local_encrypted`, `aws_secrets_manager`, `gcp_secret_manager`, `vault`
- `PAPERCLIP_HOME` - Base directory for data files
- `PAPERCLIP_PUBLIC_URL` - Public-facing URL
- `OPENAI_API_KEY` - For Codex adapter
- `ANTHROPIC_API_KEY` - For Claude adapter

**Build:**
- `tsconfig.json` (root) - Base TypeScript config: ES2023 target, NodeNext modules, strict mode
- `vitest.config.ts` (root) - Workspace test config spanning db, opencode-local, server, ui, cli
- `ui/vite.config.ts` - Vite config for React SPA
- `cli/esbuild.config.mjs` - esbuild bundling for CLI distribution

**Docker:**
- `Dockerfile` - Multi-stage build (base, deps, build, production)
- `docker-compose.yml` - PostgreSQL 17 Alpine + server
- `docker-compose.quickstart.yml` - Single-container with embedded PostgreSQL
- `Dockerfile.onboard-smoke` - Smoke test image

## Platform Requirements

**Development:**
- Node.js >= 20
- pnpm 9.15.4 (enabled via corepack)
- PostgreSQL 17 (via Docker or embedded-postgres for zero-config)

**Production:**
- Docker (recommended deployment method)
- PostgreSQL 17 (external or embedded)
- Installs `@anthropic-ai/claude-code` and `@openai/codex` globally in Docker image

---

*Stack analysis: 2026-03-08*

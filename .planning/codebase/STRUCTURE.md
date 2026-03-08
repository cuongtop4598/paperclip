# Codebase Structure

**Analysis Date:** 2026-03-08

## Directory Layout

```
paperclip/
├── cli/                        # CLI tool (@paperclipai/cli)
│   └── src/
│       ├── index.ts            # CLI entry point (commander setup)
│       ├── commands/            # Top-level commands (onboard, doctor, run, etc.)
│       │   └── client/          # API client subcommands (company, issue, agent, etc.)
│       ├── config/              # Config file loading and data-dir resolution
│       ├── client/              # HTTP client for Paperclip API
│       ├── adapters/            # CLI-side adapter helpers
│       │   ├── http/            # HTTP adapter CLI
│       │   └── process/         # Process adapter CLI
│       ├── checks/              # Diagnostic check implementations
│       ├── prompts/             # Interactive prompt helpers
│       ├── utils/               # CLI utilities
│       └── __tests__/           # CLI tests
├── server/                     # API server (@paperclipai/server)
│   └── src/
│       ├── index.ts            # Server entry point (DB init, startup, listeners)
│       ├── app.ts              # Express app factory (middleware + route mounting)
│       ├── config.ts           # Config loading (env + file + defaults)
│       ├── errors.ts           # HttpError class and factory functions
│       ├── routes/             # Express route handlers
│       │   ├── index.ts        # Route barrel exports
│       │   ├── agents.ts       # Agent CRUD, keys, heartbeat, hire
│       │   ├── issues.ts       # Issue CRUD, comments, attachments
│       │   ├── projects.ts     # Project CRUD
│       │   ├── companies.ts    # Company CRUD
│       │   ├── goals.ts        # Goal CRUD
│       │   ├── approvals.ts    # Approval workflows
│       │   ├── secrets.ts      # Company secrets management
│       │   ├── costs.ts        # Cost tracking
│       │   ├── activity.ts     # Activity feed
│       │   ├── dashboard.ts    # Dashboard aggregations
│       │   ├── health.ts       # Health check endpoint
│       │   ├── access.ts       # Access control endpoints
│       │   ├── llms.ts         # LLM-facing endpoints (llms.txt)
│       │   ├── assets.ts       # File upload/download
│       │   ├── sidebar-badges.ts # Sidebar badge counts
│       │   ├── authz.ts        # Authorization helper functions
│       │   └── issues-checkout-wakeup.ts
│       ├── services/           # Business logic layer
│       │   ├── index.ts        # Service barrel exports
│       │   ├── agents.ts       # Agent business logic
│       │   ├── issues.ts       # Issue business logic
│       │   ├── companies.ts    # Company business logic
│       │   ├── projects.ts     # Project business logic
│       │   ├── goals.ts        # Goal business logic
│       │   ├── approvals.ts    # Approval business logic
│       │   ├── secrets.ts      # Secrets management
│       │   ├── costs.ts        # Cost aggregation
│       │   ├── heartbeat.ts    # Heartbeat scheduling and execution
│       │   ├── activity.ts     # Activity queries
│       │   ├── activity-log.ts # Activity logging
│       │   ├── live-events.ts  # Pub/sub for realtime events
│       │   ├── hire-hook.ts    # Post-hire agent notification
│       │   ├── dashboard.ts    # Dashboard data
│       │   ├── sidebar-badges.ts
│       │   ├── access.ts       # Access control logic
│       │   ├── agent-permissions.ts # Agent permission normalization
│       │   ├── issue-approvals.ts
│       │   ├── run-log-store.ts
│       │   └── company-portability.ts
│       ├── middleware/         # Express middleware
│       │   ├── index.ts        # Middleware barrel
│       │   ├── auth.ts         # Actor identity resolution
│       │   ├── logger.ts       # Pino logger setup
│       │   ├── error-handler.ts # Global error handler
│       │   ├── validate.ts     # Zod request body validation
│       │   ├── board-mutation-guard.ts
│       │   └── private-hostname-guard.ts
│       ├── adapters/           # Server-side adapter registry
│       │   ├── index.ts        # Barrel exports
│       │   ├── registry.ts     # Adapter type -> module mapping
│       │   ├── utils.ts        # Process tracking utilities
│       │   ├── types.ts        # Re-exported adapter types
│       │   ├── http/           # HTTP adapter (webhook-style)
│       │   ├── process/        # Generic process adapter
│       │   ├── codex-models.ts
│       │   └── cursor-models.ts
│       ├── auth/               # Authentication
│       │   └── better-auth.ts  # BetterAuth integration
│       ├── secrets/            # Secrets providers
│       │   ├── types.ts
│       │   ├── local-encrypted-provider.ts
│       │   ├── external-stub-providers.ts
│       │   └── provider-registry.ts
│       ├── storage/            # File storage providers
│       │   ├── types.ts        # StorageService interface
│       │   ├── service.ts
│       │   ├── index.ts
│       │   ├── local-disk-provider.ts
│       │   ├── s3-provider.ts
│       │   └── provider-registry.ts
│       ├── realtime/           # WebSocket layer
│       │   └── live-events-ws.ts
│       ├── types/              # TypeScript type augmentations
│       ├── __tests__/          # Server tests
│       ├── agent-auth-jwt.ts   # Agent JWT verification
│       ├── board-claim.ts      # Board ownership claim flow
│       ├── config-file.ts      # YAML/JSON config file reader
│       ├── home-paths.ts       # Default path resolution (~/.paperclip)
│       ├── paths.ts            # Env file path resolution
│       ├── redaction.ts        # Sensitive data redaction
│       └── startup-banner.ts   # Console startup output
├── ui/                         # React SPA (@paperclipai/ui)
│   └── src/
│       ├── main.tsx            # React app entry (provider tree)
│       ├── App.tsx             # Router and route definitions
│       ├── index.css           # Global styles (Tailwind)
│       ├── pages/              # Page-level components
│       │   ├── Dashboard.tsx
│       │   ├── Agents.tsx
│       │   ├── AgentDetail.tsx
│       │   ├── Projects.tsx
│       │   ├── ProjectDetail.tsx
│       │   ├── Issues.tsx
│       │   ├── IssueDetail.tsx
│       │   ├── Goals.tsx
│       │   ├── GoalDetail.tsx
│       │   ├── Approvals.tsx
│       │   ├── ApprovalDetail.tsx
│       │   ├── Costs.tsx
│       │   ├── Activity.tsx
│       │   ├── Inbox.tsx
│       │   ├── Companies.tsx
│       │   ├── CompanySettings.tsx
│       │   ├── OrgChart.tsx
│       │   ├── NewAgent.tsx
│       │   ├── Auth.tsx
│       │   ├── BoardClaim.tsx
│       │   ├── InviteLanding.tsx
│       │   └── DesignGuide.tsx
│       ├── components/         # Reusable UI components
│       │   ├── Layout.tsx      # App shell (sidebar + content)
│       │   ├── Sidebar.tsx     # Navigation sidebar
│       │   ├── CommandPalette.tsx
│       │   ├── OnboardingWizard.tsx
│       │   ├── ui/             # Primitive UI components (shadcn-style)
│       │   └── [40+ domain components]
│       ├── api/                # API client functions
│       │   ├── client.ts       # Base fetch wrapper
│       │   ├── index.ts        # Barrel
│       │   ├── agents.ts
│       │   ├── issues.ts
│       │   ├── companies.ts
│       │   ├── projects.ts
│       │   ├── goals.ts
│       │   ├── approvals.ts
│       │   ├── auth.ts
│       │   ├── health.ts
│       │   └── [more domain APIs]
│       ├── context/            # React context providers
│       │   ├── CompanyContext.tsx      # Active company state
│       │   ├── LiveUpdatesProvider.tsx # WebSocket event handling
│       │   ├── DialogContext.tsx       # Modal dialog state
│       │   ├── PanelContext.tsx        # Side panel state
│       │   ├── SidebarContext.tsx      # Sidebar collapse state
│       │   ├── BreadcrumbContext.tsx   # Breadcrumb state
│       │   ├── ThemeContext.tsx        # Dark/light theme
│       │   └── ToastContext.tsx        # Toast notifications
│       ├── hooks/              # Custom React hooks
│       │   ├── useKeyboardShortcuts.ts
│       │   ├── useCompanyPageMemory.ts
│       │   └── useProjectOrder.ts
│       ├── lib/                # Shared utilities
│       │   ├── queryKeys.ts    # React Query key factories
│       │   ├── router.tsx      # Re-export of react-router-dom
│       │   ├── utils.ts        # cn() and other helpers
│       │   ├── timeAgo.ts
│       │   ├── groupBy.ts
│       │   ├── model-utils.ts
│       │   ├── status-colors.ts
│       │   ├── company-routes.ts
│       │   ├── project-order.ts
│       │   └── recent-assignees.ts
│       ├── adapters/           # UI-side adapter config panels
│       │   ├── index.ts        # Adapter UI registry
│       │   ├── registry.ts     # Type -> UI module mapping
│       │   ├── types.ts        # UI adapter interfaces
│       │   ├── transcript.ts   # Shared transcript rendering
│       │   ├── claude-local/
│       │   ├── cursor/
│       │   ├── openclaw/
│       │   ├── openclaw-gateway/
│       │   ├── opencode-local/
│       │   ├── codex-local/
│       │   ├── pi-local/
│       │   ├── http/
│       │   └── process/
│       └── public/             # Static assets
├── packages/                   # Shared workspace packages
│   ├── db/                     # Database package (@paperclipai/db)
│   │   └── src/
│   │       ├── schema/         # Drizzle ORM table definitions (35+ tables)
│   │       │   └── index.ts    # Schema barrel
│   │       └── migrations/     # SQL migration files
│   │           └── meta/       # Migration metadata
│   ├── shared/                 # Shared types/validators (@paperclipai/shared)
│   │   └── src/
│   │       ├── index.ts        # Barrel
│   │       ├── types/          # TypeScript type definitions
│   │       ├── validators/     # Zod validation schemas
│   │       ├── constants.ts
│   │       ├── api.ts
│   │       ├── config-schema.ts
│   │       ├── agent-url-key.ts
│   │       └── project-url-key.ts
│   ├── adapter-utils/          # Adapter interfaces (@paperclipai/adapter-utils)
│   │   └── src/
│   │       ├── index.ts
│   │       ├── types.ts        # Core adapter type definitions
│   │       └── server-utils.ts # Shared server-side adapter utilities
│   └── adapters/               # Individual adapter packages
│       ├── claude-local/       # @paperclipai/adapter-claude-local
│       ├── cursor-local/       # @paperclipai/adapter-cursor-local
│       ├── codex-local/        # @paperclipai/adapter-codex-local (not listed separately)
│       ├── opencode-local/     # @paperclipai/adapter-opencode-local (not listed separately)
│       ├── openclaw/           # @paperclipai/adapter-openclaw
│       ├── openclaw-gateway/   # @paperclipai/adapter-openclaw-gateway
│       └── pi-local/           # @paperclipai/adapter-pi-local (not listed separately)
├── scripts/                    # Build and dev scripts
│   ├── dev-runner.mjs          # Orchestrates dev server startup
│   ├── smoke/                  # Smoke test scripts
│   ├── build-npm.sh
│   ├── release.sh
│   └── [other scripts]
├── skills/                     # Claude skill definitions
│   ├── paperclip/              # Core Paperclip skill
│   ├── paperclip-create-agent/ # Agent creation skill
│   ├── create-agent-adapter/   # Adapter creation skill
│   ├── para-memory-files/      # Memory files skill
│   ├── release/                # Release skill
│   └── release-changelog/      # Changelog generation skill
├── docs/                       # Mintlify documentation site
│   ├── docs.json               # Mintlify config
│   ├── adapters/               # Adapter documentation
│   ├── api/                    # API reference docs
│   ├── cli/                    # CLI reference docs
│   ├── deploy/                 # Deployment guides
│   ├── guides/                 # User guides
│   ├── specs/                  # Technical specifications
│   └── start/                  # Getting started docs
├── doc/                        # Internal design documents
│   ├── plan/                   # Planning docs
│   ├── plans/                  # More planning docs
│   ├── plugins/                # Plugin design docs
│   ├── spec/                   # Specification docs
│   └── assets/                 # Design assets (logos, avatars)
├── docker/                     # Docker support files
├── releases/                   # Release artifacts/notes
├── .claude/                    # Claude Code configuration
│   └── skills/                 # Claude skill references
├── .changeset/                 # Changeset configuration
├── .github/                    # GitHub Actions workflows
├── package.json                # Root workspace package
├── pnpm-workspace.yaml         # Workspace definition
├── pnpm-lock.yaml              # Lockfile
├── tsconfig.json               # Root TypeScript config
├── vitest.config.ts            # Root Vitest config
├── Dockerfile                  # Production Docker build
├── Dockerfile.onboard-smoke    # Smoke test Docker build
├── docker-compose.yml          # Docker Compose for dev
└── docker-compose.quickstart.yml
```

## Directory Purposes

**`server/src/routes/`:**
- Purpose: Express route handlers that parse requests, call services, return JSON
- Contains: One file per domain entity (agents, issues, projects, etc.)
- Key files: `server/src/routes/agents.ts` (largest, most complex route), `server/src/routes/authz.ts` (shared auth helpers)

**`server/src/services/`:**
- Purpose: Business logic layer; all DB queries and mutations live here
- Contains: One service factory per domain entity
- Key files: `server/src/services/index.ts` (barrel with all exports), `server/src/services/heartbeat.ts` (agent execution orchestration)

**`server/src/middleware/`:**
- Purpose: Express middleware for cross-cutting concerns
- Contains: Auth, logging, validation, error handling, hostname guards
- Key files: `server/src/middleware/auth.ts` (actor resolution), `server/src/middleware/validate.ts` (Zod validation)

**`server/src/adapters/`:**
- Purpose: Registry mapping adapter type strings to their server-side modules
- Contains: Registry file plus built-in HTTP and process adapters
- Key files: `server/src/adapters/registry.ts` (central mapping of all adapters)

**`ui/src/pages/`:**
- Purpose: Top-level page components, one per route
- Contains: 24 page components covering all board features
- Key files: `ui/src/pages/AgentDetail.tsx`, `ui/src/pages/IssueDetail.tsx`

**`ui/src/components/`:**
- Purpose: Reusable UI components
- Contains: Domain-specific components (AgentConfigForm, IssuesList, etc.) and primitive `ui/` components
- Key files: `ui/src/components/Layout.tsx` (app shell), `ui/src/components/ui/` (shadcn-style primitives)

**`ui/src/api/`:**
- Purpose: Typed API client functions wrapping fetch calls
- Contains: One file per domain entity matching server routes
- Key files: `ui/src/api/client.ts` (base `api.get/post/patch/delete` helper)

**`ui/src/context/`:**
- Purpose: React context providers for global state
- Contains: 8 providers mounted in `ui/src/main.tsx`
- Key files: `ui/src/context/CompanyContext.tsx` (active company selection), `ui/src/context/LiveUpdatesProvider.tsx` (WebSocket event handling)

**`packages/db/src/schema/`:**
- Purpose: Drizzle ORM table definitions
- Contains: 35+ table definition files, one per entity
- Key files: `packages/db/src/schema/index.ts` (barrel), `packages/db/src/schema/agents.ts`, `packages/db/src/schema/issues.ts`

## Key File Locations

**Entry Points:**
- `server/src/index.ts`: Server startup (DB, auth, HTTP server, WebSocket, schedulers)
- `ui/src/main.tsx`: React app mount with provider tree
- `cli/src/index.ts`: CLI command registration and dispatch
- `scripts/dev-runner.mjs`: Dev environment orchestrator

**Configuration:**
- `server/src/config.ts`: Server config loading (env + file + defaults)
- `server/src/config-file.ts`: YAML/JSON config file parser
- `packages/shared/src/config-schema.ts`: Config schema definitions
- `tsconfig.json`: Root TypeScript config
- `vitest.config.ts`: Test runner config
- `pnpm-workspace.yaml`: Workspace package definitions

**Core Logic:**
- `server/src/services/agents.ts`: Agent CRUD, key management, session handling
- `server/src/services/heartbeat.ts`: Agent heartbeat scheduling and execution
- `server/src/services/issues.ts`: Issue CRUD and lifecycle
- `server/src/adapters/registry.ts`: Adapter type resolution
- `packages/adapter-utils/src/types.ts`: Adapter interface contracts

**Testing:**
- `server/src/__tests__/`: Server-side tests
- `cli/src/__tests__/`: CLI tests
- `vitest.config.ts`: Root test config

## Naming Conventions

**Files:**
- `kebab-case.ts` for all TypeScript source files: `agent-auth-jwt.ts`, `live-events-ws.ts`
- `PascalCase.tsx` for React page components: `AgentDetail.tsx`, `IssueDetail.tsx`
- `camelCase.ts` for UI API/lib modules: `queryKeys.ts`, `timeAgo.ts`
- `snake_case.ts` for DB schema files: `agent_api_keys.ts`, `heartbeat_runs.ts`

**Directories:**
- `kebab-case` for all directories: `claude-local`, `adapter-utils`, `live-events`

**Packages:**
- Scoped under `@paperclipai/`: `@paperclipai/server`, `@paperclipai/db`, `@paperclipai/shared`
- Adapter packages: `@paperclipai/adapter-{name}`: `@paperclipai/adapter-claude-local`

## Where to Add New Code

**New API Endpoint:**
- Route handler: `server/src/routes/{entity}.ts` (add to existing or create new file)
- Service logic: `server/src/services/{entity}.ts`
- Export from barrel: `server/src/services/index.ts` and `server/src/routes/index.ts`
- Mount in app: `server/src/app.ts` (add `api.use()` call)
- Validation schema: `packages/shared/src/validators/{schema}.ts`
- UI API client: `ui/src/api/{entity}.ts`

**New UI Page:**
- Page component: `ui/src/pages/{PageName}.tsx`
- Route definition: `ui/src/App.tsx` (add `<Route>` inside `boardRoutes()`)
- API functions: `ui/src/api/{entity}.ts`
- Query keys: `ui/src/lib/queryKeys.ts`

**New UI Component:**
- Domain component: `ui/src/components/{ComponentName}.tsx`
- Primitive/base component: `ui/src/components/ui/{component-name}.tsx`

**New Adapter:**
- Package: `packages/adapters/{adapter-name}/` with `src/cli/`, `src/server/`, `src/ui/`, `src/index.ts`
- Register in server: `server/src/adapters/registry.ts`
- Register in UI: `ui/src/adapters/registry.ts`
- Add workspace dep in `server/package.json` and `ui/package.json`

**New Database Table:**
- Schema: `packages/db/src/schema/{table_name}.ts`
- Export from barrel: `packages/db/src/schema/index.ts`
- Generate migration: `pnpm db:generate`
- Apply migration: `pnpm db:migrate`

**New Shared Type/Validator:**
- Type: `packages/shared/src/types/{type}.ts`
- Validator: `packages/shared/src/validators/{schema}.ts`
- Export from barrel: `packages/shared/src/index.ts`

**New CLI Command:**
- Command handler: `cli/src/commands/{command-name}.ts`
- Register in: `cli/src/index.ts`
- For API client subcommands: `cli/src/commands/client/{entity}.ts`

**Utilities:**
- Server-side: `server/src/` (top-level for cross-cutting, or in relevant subdirectory)
- UI shared helpers: `ui/src/lib/`
- Cross-package: `packages/shared/src/`

## Special Directories

**`skills/`:**
- Purpose: Claude Code skill definitions for AI-assisted development
- Generated: No (manually authored)
- Committed: Yes

**`docs/`:**
- Purpose: Mintlify-powered public documentation site
- Generated: No (manually authored)
- Committed: Yes

**`doc/`:**
- Purpose: Internal design documents, plans, and specs
- Generated: No (manually authored)
- Committed: Yes

**`.changeset/`:**
- Purpose: Changeset files for version management
- Generated: Via `pnpm changeset` command
- Committed: Yes

**`.planning/`:**
- Purpose: GSD planning and codebase analysis documents
- Generated: By analysis tools
- Committed: Varies

**`releases/`:**
- Purpose: Release notes and artifacts
- Generated: During release process
- Committed: Yes

---

*Structure analysis: 2026-03-08*

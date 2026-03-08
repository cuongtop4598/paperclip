# Coding Conventions

**Analysis Date:** 2026-03-08

## Naming Patterns

**Files:**
- Use `kebab-case.ts` for all TypeScript files: `error-handler.ts`, `activity-log.ts`, `run-log-store.ts`
- Use `PascalCase.tsx` for React components: `StatusBadge.tsx`, `KanbanBoard.tsx`, `MetricCard.tsx`
- Exception: `ui/` primitives use lowercase: `agent-config-primitives.tsx`
- UI primitive components from shadcn/radix live at `ui/src/components/ui/` and use `kebab-case.tsx`: `button.tsx`, `dropdown-menu.tsx`
- Test files use `*.test.ts` suffix (never `.spec.ts`)

**Functions:**
- Use `camelCase` for all functions: `loadConfig()`, `hashToken()`, `normalizeMaxConcurrentRuns()`
- Route factory functions follow `{entity}Routes(db)` pattern: `companyRoutes(db)`, `issueRoutes(db)`
- Service factory functions follow `{entity}Service(db)` pattern: `companyService(db)`, `heartbeatService(db)`
- Error factory functions are short verb phrases: `badRequest()`, `notFound()`, `unauthorized()`, `conflict()`
- React components use `PascalCase` function names: `export function Dashboard()`, `export function StatusBadge()`

**Variables:**
- Use `camelCase` for variables and parameters
- Use `UPPER_SNAKE_CASE` for constants: `MAX_ATTACHMENT_BYTES`, `HEARTBEAT_MAX_CONCURRENT_RUNS_DEFAULT`
- Environment variable references use `PAPERCLIP_` prefix: `PAPERCLIP_API_URL`, `PAPERCLIP_DEPLOYMENT_MODE`

**Types:**
- Use `PascalCase` for types and interfaces: `Config`, `LogActivityInput`, `ErrorContext`
- Zod schemas use `camelCase` with `Schema` suffix: `createCompanySchema`, `updateIssueSchema`
- Inferred types from Zod use `PascalCase`: `type CreateCompany = z.infer<typeof createCompanySchema>`
- Constant arrays use `UPPER_SNAKE_CASE`: `COMPANY_STATUSES`, `DEPLOYMENT_MODES`, `AGENT_ADAPTER_TYPES`
- Corresponding union types use `PascalCase`: `type CompanyStatus`, `type DeploymentMode`

## Code Style

**Formatting:**
- No explicit Prettier/ESLint config detected -- formatting is convention-based
- Double quotes for strings in TypeScript (consistent across codebase)
- 2-space indentation
- Trailing commas in multi-line constructs
- Line length generally under 120 characters, but no hard enforcement

**Linting:**
- No ESLint, Biome, or Prettier config files present
- TypeScript `strict: true` in `tsconfig.json` serves as primary code quality gate
- `forceConsistentCasingInFileNames: true` enforced

**TypeScript Configuration:**
- Target: `ES2023`
- Module: `NodeNext` with `NodeNext` module resolution
- Strict mode enabled with `isolatedModules: true`
- All packages share the root `tsconfig.json` base config

## Import Organization

**Order:**
1. Node.js built-ins with `node:` prefix: `import fs from "node:fs"`, `import path from "node:path"`
2. Third-party packages: `express`, `drizzle-orm`, `zod`, `pino`
3. Workspace packages: `@paperclipai/db`, `@paperclipai/shared`, `@paperclipai/adapter-utils`
4. Local relative imports with `.js` extension: `"../errors.js"`, `"./authz.js"`

**Path Aliases:**
- UI uses `@/` alias for `src/`: `import { Link } from "@/lib/router"`
- Server and packages use relative paths with `.js` extension (required by NodeNext resolution)

**Extension Rules:**
- All relative imports in server/packages MUST include `.js` extension: `import { HttpError } from "../errors.js"`
- This is required by `"module": "NodeNext"` resolution
- UI imports do NOT use `.js` extension (Vite handles resolution)

**Module System:**
- All packages use `"type": "module"` in `package.json`
- ESM throughout -- no CommonJS

## Error Handling

**HTTP Error Pattern:**
- Use `HttpError` class from `server/src/errors.ts` with factory functions
- Factory functions: `badRequest()`, `unauthorized()`, `forbidden()`, `notFound()`, `conflict()`, `unprocessable()`
- Throw errors directly in route handlers -- the global `errorHandler` middleware catches them
- `ZodError` instances are caught and returned as 400 with validation details

```typescript
// Pattern: throw error factories in route handlers
import { forbidden, notFound } from "../errors.js";

router.get("/:id", async (req, res) => {
  assertCompanyAccess(req, companyId);  // throws forbidden() internally
  const item = await svc.getById(id);
  if (!item) throw notFound("Item not found");  // or inline: res.status(404).json(...)
  res.json(item);
});
```

**Authorization Pattern:**
- Use `assertBoard(req)` and `assertCompanyAccess(req, companyId)` from `server/src/routes/authz.ts`
- These throw `forbidden()` or `unauthorized()` -- no manual checking needed
- `getActorInfo(req)` extracts actor metadata for activity logging

**Validation Pattern:**
- Use Zod schemas from `@paperclipai/shared` package
- Apply via `validate()` middleware from `server/src/middleware/validate.ts`
- The middleware calls `schema.parse(req.body)` and replaces `req.body` with the parsed result
- ZodError propagates to the global error handler

```typescript
router.post("/:companyId/export", validate(companyPortabilityExportSchema), async (req, res) => {
  // req.body is now typed and validated
  const result = await portability.exportBundle(companyId, req.body);
  res.json(result);
});
```

## Logging

**Framework:** Pino (`pino` + `pino-http` + `pino-pretty`)

**Configuration:** `server/src/middleware/logger.ts`
- Console output: `info` level with pretty formatting (colorized)
- File output: `debug` level to `server.log` in configured log directory
- HTTP request logging via `pino-http` middleware with custom log levels

**Patterns:**
- Use `logger` import for direct logging: `logger.info(...)`, `logger.warn(...)`
- HTTP errors (4xx/5xx) automatically log request body, params, query via `customProps`
- Error context is attached to `res.__errorContext` for structured logging
- Sensitive data is redacted before logging using `sanitizeRecord()` from `server/src/redaction.ts`

## Comments

**When to Comment:**
- Route handlers are generally self-documenting -- minimal comments
- Constants at module top have inline comments when purpose is non-obvious
- `// Re-export` comments on shim/barrel files explaining why they exist
- Edge cases get brief inline comments: `// Common malformed path when companyId is empty`

**JSDoc/TSDoc:**
- Not used systematically -- the codebase relies on TypeScript types for documentation
- Interface fields are self-documenting via naming

## Function Design

**Size:** Functions are generally short (under 50 lines). Service functions may be longer but are well-factored.

**Parameters:**
- Use typed object parameters for functions with more than 2-3 args: `opts: { deploymentMode: ...; ... }`
- Route factory functions take `db: Db` as first parameter, plus optional config object
- Service factory functions return an object with methods (closure pattern, not classes)

**Return Values:**
- Route handlers call `res.json()` or `res.status().json()` directly
- Service functions return plain objects or arrays
- Async functions use `async/await` consistently (no raw Promises)

## Module Design

**Exports:**
- Named exports everywhere -- no default exports
- `export function` for functions, `export type` for types, `export const` for constants
- Barrel files (`index.ts`) re-export from sub-modules

**Barrel Files:**
- `packages/shared/src/index.ts`: Central barrel exporting all shared types, validators, and constants
- `server/src/middleware/index.ts`: Re-exports all middleware
- `server/src/services/index.ts`: Re-exports all services
- `packages/adapter-utils/src/index.ts`: Re-exports adapter types

**Service Pattern (Server):**
- Services are factory functions that take `db: Db` and return an object of methods
- Services are instantiated per-route file, not globally
- `server/src/services/index.ts` provides convenience re-exports

```typescript
// Pattern: service factory returning method object
export function companyService(db: Db) {
  return {
    async list() { ... },
    async getById(id: string) { ... },
    async create(input: CreateCompany) { ... },
  };
}
```

**Route Pattern (Server):**
- Route factory functions return Express `Router` instances
- Routes are mounted in `server/src/app.ts` under `/api`
- Each route file instantiates its needed services at the top

```typescript
export function companyRoutes(db: Db) {
  const router = Router();
  const svc = companyService(db);
  router.get("/", async (req, res) => { ... });
  return router;
}
```

## UI Conventions

**Component Pattern:**
- React function components with named exports: `export function Dashboard()`
- No class components
- Props destructured inline: `export function StatusBadge({ status }: { status: string })`
- Hooks at the top of component body

**State Management:**
- React Query (`@tanstack/react-query`) for server state
- Query keys defined centrally in `ui/src/lib/queryKeys.ts`
- Context providers for app-level state: `CompanyContext`, `DialogContext`, `BreadcrumbContext`

**Styling:**
- Tailwind CSS v4 with `cn()` utility from `ui/src/lib/utils.ts` (uses `clsx` + `tailwind-merge`)
- shadcn/ui component primitives in `ui/src/components/ui/`
- Icons from `lucide-react`

**API Client Pattern:**
- API modules in `ui/src/api/` with entity-based files: `agents.ts`, `issues.ts`, `companies.ts`
- Central HTTP client in `ui/src/api/client.ts`

---

*Convention analysis: 2026-03-08*

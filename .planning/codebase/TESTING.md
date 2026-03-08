# Testing Patterns

**Analysis Date:** 2026-03-08

## Test Framework

**Runner:**
- Vitest 3.x
- Root config: `vitest.config.ts`
- Per-workspace configs: `server/vitest.config.ts`, `cli/vitest.config.ts`, `ui/vitest.config.ts`, `packages/db/vitest.config.ts`, `packages/adapters/opencode-local/vitest.config.ts`

**Assertion Library:**
- Vitest built-in `expect` (Chai-compatible API)

**Run Commands:**
```bash
pnpm test                # Run vitest in watch mode (all workspaces)
pnpm test:run            # Run all tests once (CI mode)
vitest run               # Run directly
```

## Test File Organization

**Location:**
- Server tests: `server/src/__tests__/*.test.ts` (centralized `__tests__` directory)
- CLI tests: `cli/src/__tests__/*.test.ts` (centralized `__tests__` directory)
- Adapter tests: co-located with source files, e.g., `packages/adapters/opencode-local/src/server/models.test.ts`

**Naming:**
- Pattern: `{feature-name}.test.ts`
- Descriptive names matching the feature under test: `error-handler.test.ts`, `redaction.test.ts`, `agent-auth-jwt.test.ts`, `invite-join-manager.test.ts`

**Structure:**
```
server/src/__tests__/
  adapter-models.test.ts
  agent-auth-jwt.test.ts
  claude-local-adapter.test.ts
  error-handler.test.ts
  health.test.ts
  invite-join-manager.test.ts
  openclaw-adapter.test.ts
  private-hostname-guard.test.ts
  redaction.test.ts
  storage-local-provider.test.ts
  ...

cli/src/__tests__/
  allowed-hostname.test.ts
  common.test.ts
  context.test.ts
  data-dir.test.ts
  http.test.ts
  ...
```

## Test Environments

**Server/CLI/Packages:**
- `environment: "node"` in vitest config

**UI:**
- `environment: "jsdom"` in `ui/vitest.config.ts`

**Workspace Projects:**
- Root `vitest.config.ts` defines project list: `["packages/db", "packages/adapters/opencode-local", "server", "ui", "cli"]`

## Test Structure

**Suite Organization:**
```typescript
import { describe, expect, it, vi } from "vitest";

describe("featureName", () => {
  it("does the expected thing", () => {
    // arrange
    const input = { ... };
    // act
    const result = functionUnderTest(input);
    // assert
    expect(result).toBe(expected);
  });
});
```

**Setup/Teardown for Environment-Dependent Tests:**
```typescript
const ORIGINAL_ENV = { ...process.env };

beforeEach(() => {
  process.env = { ...ORIGINAL_ENV };
  delete process.env.SOME_VAR;
});

afterEach(() => {
  process.env = { ...ORIGINAL_ENV };
});
```

**Test Naming:**
- Use descriptive sentence-style descriptions: `"returns null when secret is missing"`, `"blocks unknown hostnames with remediation command"`
- Start with verb: `"detects..."`, `"returns..."`, `"throws..."`, `"uses..."`, `"rejects..."`

## Mocking

**Framework:** Vitest built-in `vi`

**Global Fetch Mocking Pattern (most common):**
```typescript
const fetchMock = vi.fn().mockResolvedValue(
  new Response(JSON.stringify({ ok: true }), { status: 200 }),
);
vi.stubGlobal("fetch", fetchMock);

// Assert
expect(fetchMock).toHaveBeenCalledTimes(1);
const [url, init] = fetchMock.mock.calls[0] as [string, RequestInit];
expect(url).toContain("/api/test");
```

**Module Mocking Pattern:**
```typescript
vi.mock("../adapters/registry.js", () => ({
  findServerAdapter: vi.fn(),
}));

vi.mock("../services/activity-log.js", () => ({
  logActivity: vi.fn().mockResolvedValue(undefined),
}));

// Import AFTER mock declaration
const { findServerAdapter } = await import("../adapters/registry.js");

// Use in test
vi.mocked(findServerAdapter).mockReturnValue({ type: "openclaw", ... } as any);
```

**Express Request/Response Mocking Pattern:**
```typescript
function makeReq(): Request {
  return {
    method: "GET",
    originalUrl: "/api/test",
    body: { a: 1 },
    params: { id: "123" },
    query: { q: "x" },
  } as unknown as Request;
}

function makeRes(): Response {
  const res = {
    status: vi.fn(),
    json: vi.fn(),
  } as unknown as Response;
  (res.status as unknown as ReturnType<typeof vi.fn>).mockReturnValue(res);
  return res;
}
```

**Supertest Integration Test Pattern:**
```typescript
import express from "express";
import request from "supertest";

describe("GET /health", () => {
  const app = express();
  app.use("/health", healthRoutes());

  it("returns 200 with status ok", async () => {
    const res = await request(app).get("/health");
    expect(res.status).toBe(200);
    expect(res.body).toEqual({ status: "ok" });
  });
});
```

**Fake Timers:**
```typescript
beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

it("rejects expired tokens", () => {
  vi.setSystemTime(new Date("2026-01-01T00:00:00.000Z"));
  const token = createToken(...);
  vi.setSystemTime(new Date("2026-01-01T00:00:05.000Z"));
  expect(verifyToken(token)).toBeNull();
});
```

**What to Mock:**
- Global `fetch` for HTTP adapter tests (use `vi.stubGlobal("fetch", fetchMock)`)
- Module-level dependencies via `vi.mock()` for service isolation
- Express req/res objects for middleware tests
- Timers when testing time-dependent behavior

**What NOT to Mock:**
- Pure functions (test directly with real inputs)
- Zod schemas (validate with real data)
- File system in temp-dir tests (use real fs with `os.tmpdir()`)

## Fixtures and Factories

**Test Data:**
- Inline object literals for simple cases
- Helper factory functions for complex setups, defined within test files

```typescript
// Factory for adapter execution context
function buildContext(
  config: Record<string, unknown>,
  overrides?: Partial<AdapterExecutionContext>,
): AdapterExecutionContext {
  return {
    runId: "run-123",
    agent: {
      id: "agent-123",
      companyId: "company-123",
      name: "Test Agent",
      adapterType: "openclaw",
      adapterConfig: {},
    },
    // ...defaults
    ...overrides,
  };
}
```

```typescript
// Factory for mock Db
function mockDbWithAgent(agent: { id: string; companyId: string; name: string; adapterType: string }): Db {
  return {
    select: () => ({
      from: () => ({
        where: () => Promise.resolve([{ ...agent, adapterConfig: {} }]),
      }),
    }),
  } as unknown as Db;
}
```

**Temp File Pattern:**
```typescript
function createTempPath(name: string): string {
  const dir = fs.mkdtempSync(path.join(os.tmpdir(), "paperclip-cli-common-"));
  return path.join(dir, name);
}
```

**Location:**
- No shared fixtures directory -- factories are defined per test file
- Temp directories cleaned up in `afterEach` hooks

## Coverage

**Requirements:** No enforced coverage thresholds detected

**View Coverage:**
```bash
vitest run --coverage      # If coverage provider is configured
```

## Test Types

**Unit Tests:**
- Pure function tests: `redaction.test.ts`, `adapter-models.test.ts`, `invite-join-manager.test.ts`
- Test single functions in isolation with direct input/output assertions
- No database required

**Integration Tests (lightweight):**
- Supertest-based route tests: `health.test.ts`, `private-hostname-guard.test.ts`
- Create minimal Express app, mount route, make HTTP assertions
- No real database -- mock service layer or test routes that don't need DB

**Service Tests with Mocked Dependencies:**
- `hire-hook.test.ts` -- mocks adapter registry and activity log
- Test service orchestration logic without real adapters or database

**Adapter Tests:**
- `openclaw-adapter.test.ts`, `claude-local-adapter.test.ts` -- test adapter execute/testEnvironment functions
- Mock `fetch` globally to simulate SSE streams and HTTP responses
- Verify outbound request structure, headers, and response parsing

**File System Tests:**
- `storage-local-provider.test.ts` -- uses real temp directories
- `context.test.ts`, `common.test.ts` -- real temp files for CLI config

**E2E Tests:**
- Not present in the test suite
- Smoke tests exist as shell scripts in `scripts/smoke/` but are not part of vitest

## Common Patterns

**Async Testing:**
```typescript
it("round-trips bytes through storage service", async () => {
  const service = createStorageService(createLocalDiskStorageProvider(root));
  const stored = await service.putFile({ ... });
  const fetched = await service.getObject("company-1", stored.objectKey);
  expect(fetchedBody.toString("utf8")).toBe("hello image bytes");
});
```

**Error Testing:**
```typescript
// Throwing errors
it("throws when company is required but unresolved", () => {
  expect(() =>
    resolveCommandContext({ ... }, { requireCompany: true }),
  ).toThrow(/Company ID is required/);
});

// Async rejection
it("rejects when model is missing", async () => {
  await expect(
    ensureOpenCodeModelConfiguredAndAvailable({ model: "" }),
  ).rejects.toThrow("OpenCode requires `adapterConfig.model`");
});

// HttpError matching
await expect(service.getObject("company-b", stored.objectKey))
  .rejects.toMatchObject({ status: 403 });

// Custom error class matching
await expect(client.post("/api/issues/1/checkout", {})).rejects.toMatchObject({
  status: 409,
  message: "Issue checkout conflict",
  details: { issueId: "1" },
} satisfies Partial<ApiRequestError>);
```

**SSE Stream Testing:**
```typescript
function sseResponse(lines: string[]) {
  const encoder = new TextEncoder();
  const stream = new ReadableStream<Uint8Array>({
    start(controller) {
      for (const line of lines) {
        controller.enqueue(encoder.encode(line));
      }
      controller.close();
    },
  });
  return new Response(stream, {
    status: 200,
    headers: { "content-type": "text/event-stream" },
  });
}
```

**Cleanup Patterns:**
```typescript
afterEach(() => {
  vi.restoreAllMocks();    // Restore all mocked functions
  vi.unstubAllGlobals();   // Restore global stubs (fetch, etc.)
});

// For temp files
afterEach(async () => {
  await Promise.all(tempRoots.map((root) => fs.rm(root, { recursive: true, force: true })));
  tempRoots.length = 0;
});
```

**Partial Object Matching:**
```typescript
expect(logActivity).toHaveBeenCalledWith(
  expect.anything(),
  expect.objectContaining({
    action: "hire_hook.succeeded",
    entityId: "a1",
    details: expect.objectContaining({ source: "approval" }),
  }),
);
```

---

*Testing analysis: 2026-03-08*

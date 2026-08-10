---
inclusion: fileMatch
fileMatchPattern: ["src/app/api/**/*.ts", "src/lib/**/*.ts"]
---
# API & Data Layer Conventions

## Reference docs
#[[file:docs/LearnUpon_API_Documentation.md]]
#[[file:docs/caching.md]]

---

## Rules

### RULE: Use LMS wrappers — never raw `fetch()` for LearnUpon calls
- **Trigger:** Adding or modifying code that calls LearnUpon APIs
- **Check:** Does the code use raw `fetch()` to call a LearnUpon endpoint?
- **Instruction:** Use `luFetch()` (public API, Basic Auth) from `src/lib/learnupon-client.ts` or `luSessionFetch()` (session API, cookie) from `src/lib/learnupon-session.ts`. Both are portal-aware via `getActivePortalConfig()`. Exception: non-LMS external services (e.g., Resend) and `/api/health` (lightweight connectivity probe) may use raw `fetch()`.

### RULE: Never read `process.env.LMS_*` directly
- **Trigger:** Adding or modifying code that needs portal credentials or URLs
- **Check:** Does the code reference `process.env.LMS_` anywhere?
- **Instruction:** Use `getActivePortalConfig()` from `src/lib/portal.ts` instead. The only place that reads these env vars is the portal config module itself. Routes that bypass `luFetch()` (like `/api/users/all` using `pipelineFetch` directly) must still call `getActivePortalConfig()`.

### RULE: Consider cacheability for new LMS data
- **Trigger:** Adding a new endpoint that fetches portal-wide, rarely-changing data from LearnUpon
- **Check:** Is this data shared across all users and stable within a session (e.g., groups, journey structures, course trees)?
- **Instruction:** If yes, add a cache function to `src/lib/platform-cache.ts` (24h TTL). User-specific data (enrollments, memberships, journey state per user) must always be fetched live — never cached.

### RULE: Add `maxDuration = 300` for paginating/fan-out routes
- **Trigger:** Adding or modifying an API route
- **Check:** Does the route call any of these: `getEnrollmentsByEmail()`, `getMarkCompletesByUserId()`, `getAllUsers()`, `getGroupMemberships()`, any function that loops on `LU-Has-Next-Page`, `parallelWithLimit()`, or `luSessionFetch()` in a loop?
- **Instruction:** If yes, add `export const maxDuration = 300` at route file level. Exception: routes making a single LMS call and returning (e.g., `GET /api/groups`, `GET /api/courses`) don't need this. The explicit export documents intent and defends against platform changes.

### RULE: Use `parallelWithLimit(tasks, 5)` for fan-out
- **Trigger:** Implementing a pattern where multiple independent LMS requests run concurrently
- **Check:** Are tasks independent and order doesn't matter?
- **Instruction:** Use `parallelWithLimit(tasks, 5)` from `src/lib/parallel.ts`. The pipeline's semaphore also caps at 5, so effective concurrency won't exceed 5 regardless of the limit passed.

### RULE: Every API route starts with `withAuthAndRole()`
- **Trigger:** Adding or modifying an API route handler
- **Check:** Is `withAuthAndRole("VIEWER"|"OPERATOR"|"ADMIN")` the first line of the handler?
- **Instruction:** It must be. Exceptions: `/api/auth/[...nextauth]` (is the auth provider), `/api/health` (unauthenticated connectivity probe for external monitoring), `/api/csp-report` (browser-initiated CSP reports without session context).

### RULE: Use shared utilities — never duplicate logic inline
- **Trigger:** Writing helper logic in a route handler
- **Check:** Does equivalent logic already exist in `src/lib/`? Specifically: `fetchCourseTreeFromLMS()` (workflow-parser.ts), `buildCourseEnrollmentMaps()` (enrollment-utils.ts), `handleSchemaBreakError()` (api-helpers.ts), `handleApiError()` (api-helpers.ts).
- **Instruction:** Import and use the shared utility. If the logic doesn't exist yet but a second route would need it, extract to a shared module immediately.

### RULE: Use `handleApiError()` for error responses
- **Trigger:** Writing a catch block in an API route handler
- **Check:** Does the catch block manually construct error responses for `LearnUponError`, `CircuitBreakerError`, `AbortError`, or unknown errors?
- **Instruction:** Use `handleApiError(error)` from `src/lib/api-helpers.ts`. It handles all these cases consistently (including 503 for circuit breaker).

### RULE: Use `handleSchemaBreakError()` for SchemaBreakError
- **Trigger:** Writing a catch block in a route that uses session API / response guards
- **Check:** Does the route catch `SchemaBreakError`?
- **Instruction:** Use `handleSchemaBreakError(error)` from `src/lib/api-helpers.ts`. It logs, schedules the alert email via `after()` (keeps serverless invocation alive), and returns 502. Never use bare fire-and-forget for serverless background work — always use `after()` from `next/server`.

### RULE: Don't add manual rate-limit handling in routes
- **Trigger:** Implementing retry or rate-limit logic in a route handler
- **Check:** Is the code manually checking for 429 status or adding delay logic?
- **Instruction:** Don't. The request pipeline handles 429s (retry with backoff) and circuit breaks (503) automatically. Route handlers should not contain rate-limit awareness — the adaptive throttler manages pacing.

### RULE: Internal endpoint paths from `lu-internal-paths.ts` only
- **Trigger:** Writing code that calls a LearnUpon internal (`/angie/...`) endpoint
- **Check:** Is the path hardcoded as a string literal in the route handler?
- **Instruction:** Import from `src/lib/lu-internal-paths.ts` instead. All internal paths are centralized there — when a path changes, one file update fixes everything.

### RULE: Response guards required for internal endpoint JSON reads
- **Trigger:** Parsing JSON from an internal LearnUpon endpoint response
- **Check:** Does the code access fields on the parsed JSON without running it through a guard from `src/lib/lu-response-guards.ts`?
- **Instruction:** Run through the appropriate guard (e.g., `guardDashboard(data, "source")`) before accessing fields. Guards throw `SchemaBreakError` on shape mismatch — fail loud, never silently fall back to empty data.

### RULE: Place guards inside platform-cache fetcher callbacks
- **Trigger:** Adding a guard for a response that will be cached
- **Check:** Is the guard running at route level (after cache hit) or inside the cache fetcher callback (only on cache miss)?
- **Instruction:** Place inside the fetcher callback. The guard validates once on fetch, and cached data is already validated.

### RULE: Audit user-initiated actions
- **Trigger:** Adding or modifying a route that performs a user-initiated action (reads, mutations, deletions, pre-flight checks)
- **Check:** Does the route call `await audit(userEmail, "ACTION_NAME", outcome, targetId)`?
- **Instruction:** Add audit logging for any action where you'd want a log of who did what. Use outcomes: `"success"`, `"failure"`, `"not_found"`, `"denied"`.

### RULE: Consistent mutation response shape
- **Trigger:** Adding or modifying a POST/DELETE/PUT route
- **Check:** Does the route return `{ success: true }` on success and `{ error: "message" }` on failure?
- **Instruction:** Follow this shape. Client-side `useMutation` hook expects it.

### RULE: Client-side fetching uses `apiFetch()` only
- **Trigger:** Adding client-side code that fetches data
- **Check:** Does the code use raw `fetch()`?
- **Instruction:** Use `apiFetch()` from `src/lib/fetch-client.ts`. This is also enforced by ESLint (raw `fetch` is a lint error in client code). For fetch-on-mount patterns, use `useApiQuery()` hook.

### RULE: Pass AbortSignal for client-side requests
- **Trigger:** Adding a component that fetches data on mount or on dependency change
- **Check:** Is an `AbortController` signal passed to cancel on unmount/navigation?
- **Instruction:** Use `useApiQuery()` hook (handles abort automatically) or manually pass the signal from a component-level `AbortController`. Check `src/lib/client-cache.ts` before fetching — use `getCached(key)` / `setCached(key, data)` for data stable within 5 minutes.

### RULE: Use `useMutation` + `MutationButton` for client-side mutations
- **Trigger:** Adding a button that triggers a POST/DELETE/PUT action
- **Check:** Is mutation state (loading/error) managed manually with useState?
- **Instruction:** Use `useMutation` hook from `src/hooks/use-mutation.ts` and `MutationButton` from `src/components/ui/mutation-button.tsx`. For complex multi-step operations (bulk delete, batch enrollment), use step-based state machine pattern with `SequentialRunner`.

### RULE: `SequentialRunner` for progress-reporting operations
- **Trigger:** Building a multi-step operation where the UI needs to show progress (e.g., "3 of 15 complete")
- **Check:** Does the operation need progress reporting, pause/resume, or sequential guarantees?
- **Instruction:** Use `SequentialRunner` from `src/lib/sequential-runner.ts`. Use `parallelWithLimit` only when speed matters and tasks are independent with no progress UI needed.

---

## Knowledge: Concurrency utilities reference

### `parallelWithLimit(tasks, limit)` — from `src/lib/parallel.ts`
Use for fan-out patterns where tasks are independent and order doesn't matter:
- Checking multiple journeys for enrollment state
- Fetching group triggers for multiple groups
- Any batch of GET requests where you want results fast

### `SequentialRunner` — from `src/lib/sequential-runner.ts`
Use for operations that must execute one-at-a-time with user-facing progress, pause/resume, and cancellation:
- Bulk progress reports (fetching progress for N users sequentially)
- Bulk deletions (deleting enrollments one-by-one with confirmation)
- Any multi-step operation where the UI needs to show "3 of 15 complete"

Features: configurable delay (`delayMs`), `onProgress(completed, total)` callback, pause/resume, cancellation, ordered results.

## Knowledge: Rate limit behavior

LearnUpon returns these headers on every response:
- `X-LU-Rate-Limit-Remaining-Minute` — remaining calls this minute
- `X-LU-Rate-Limit-Remaining-Week` — remaining calls this week (rolling)

The pipeline reads these automatically. The adaptive throttler:
- > 30 remaining: full speed (0ms delay)
- 10-30 remaining: 200ms between calls
- 5-10 remaining: 500ms between calls
- < 5 remaining: 2s between calls

## Knowledge: RBAC roles

Defined in `src/lib/roles.ts`:
- `VIEWER` — read-only access (default for all @ebli.com)
- `OPERATOR` — future: limited write access
- `ADMIN` — full access (mutations, delete, cache management)

Admin emails are hardcoded in `roles.ts`. Consider moving to env var if team grows.

---

## Verifying your work

After adding or modifying an API route:

- **Build check:** `npm run build` must pass (catches type errors and import issues)
- **Lint check:** `npm run lint` must pass (catches raw fetch, server-only imports in client code)
- **Cache changes:** If you modified `platform-cache.ts`, hit `GET /api/cache/invalidate` in dev and confirm the new key appears in the stats response
- **Pipeline behavior:** If you modified `request-pipeline.ts`, check `GET /api/pipeline/status` to confirm circuit breaker state, queue depth, and throttle level look sane
- **Schema guards:** If you added a new response guard, hit the endpoint manually in dev and confirm the guard passes (no 502)

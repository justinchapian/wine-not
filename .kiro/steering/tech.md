---
inclusion: always
---
# Tech Stack & Architecture

---

## Rules

### RULE: All outbound LMS calls go through the request pipeline
- **Trigger:** Adding or modifying code that makes HTTP requests to LearnUpon
- **Check:** Does the code call LearnUpon without going through `luFetch()` or `luSessionFetch()` (which both use `pipelineFetch()` from `src/lib/request-pipeline.ts`)?
- **Instruction:** Use `luFetch()` for public API (Basic Auth) or `luSessionFetch()` for session API (cookie). Never raw `fetch()` for LMS calls. See `api-conventions.md` RULE: "Use LMS wrappers" for the full specification and exceptions.

### RULE: Never read `process.env.LMS_*` in route handlers
- **Trigger:** Adding or modifying code that needs portal credentials or URLs
- **Check:** Does the code reference `process.env.LMS_` anywhere outside `src/lib/portal.ts`?
- **Instruction:** Use `getActivePortalConfig()` from `src/lib/portal.ts`. Routes bypassing `luFetch()` (like `/api/users/all`) must still call `getActivePortalConfig()` and set `Vary: Cookie` on edge-cached responses. See `api-conventions.md` RULE: "Never read process.env.LMS_*" for full details.

### RULE: All client-side API calls use `apiFetch()`
- **Trigger:** Adding client-side code that makes HTTP requests
- **Check:** Does the code use raw `fetch()`?
- **Instruction:** Use `apiFetch()` from `src/lib/fetch-client.ts`. Enforced by ESLint — raw `fetch` is a lint error in client code. See `api-conventions.md` RULE: "Client-side fetching uses apiFetch() only."

### RULE: Use `Button` with `variant` prop — never raw `<button>`
- **Trigger:** Adding or modifying UI that includes buttons
- **Check:** Does the code use raw `<button>` elements with inline Tailwind classes?
- **Instruction:** Use the `Button` component from `src/components/ui/button.tsx` with the appropriate `variant` prop. See the Button variants table in Knowledge section below.

### RULE: Never duplicate utility functions inline in route handlers
- **Trigger:** Writing helper logic in a route handler or component
- **Check:** Does equivalent logic already exist in `src/lib/`?
- **Instruction:** Import the shared utility. If the logic doesn't exist yet but a second route would need it, extract to `src/lib/<domain>.ts`. See `api-conventions.md` RULE: "Use shared utilities" for the specific utility list.

### RULE: Don't weaken or remove security headers without explicit user approval
- **Trigger:** Modifying `next.config.ts`
- **Check:** Does the change remove, weaken, or bypass any header in the `headers()` function?
- **Instruction:** Preserve all security headers. If a change requires weakening CSP or removing a header, stop and get explicit user confirmation first. The current headers enforce: X-Frame-Options DENY, nosniff, strict CSP, HSTS, disabled browser APIs.

### RULE: Portal-bypassing routes must call `getActivePortalConfig()` and set `Vary: Cookie`
- **Trigger:** Adding or modifying a route that uses `pipelineFetch` directly instead of `luFetch()`/`luSessionFetch()`
- **Check:** Does the route (1) call `getActivePortalConfig()` for credentials/URLs and (2) set `Vary: Cookie` on any edge-cached response headers?
- **Instruction:** Both are required. Without `getActivePortalConfig()`, the route ignores portal switching. Without `Vary: Cookie`, Vercel CDN serves stale data from the wrong portal.

### RULE: Size convention for buttons
- **Trigger:** Adding buttons to page content (toolbars, tables, section headers)
- **Check:** Is the button using default size inside page content areas?
- **Instruction:** Use `size="sm"` for all buttons within page content. Default size is only appropriate for full-page-level CTAs or standalone form submit buttons. Tab/toggle pattern: `variant={isActive ? "default" : "secondary"}` with `size="sm"`.

---

## Knowledge: Stack

- Next.js 16 (App Router, "use client" components)
- React 19
- TypeScript 5
- Tailwind CSS 4
- Vercel (Hobby plan, Fluid Compute)

### Key libraries
- `next-auth` v4 — Google OAuth with domain restriction
- `jose` — JWT encryption for LMS session credentials
- `xlsx` — Excel report generation (client-side)
- `@tanstack/react-virtual` — Virtualized user table (3000 rows)
- `lucide-react` — Icon library used across UI components

## Knowledge: Dual-auth model

The app has two independent authentication layers:

**Layer 1 — Google OAuth (app access):**
- NextAuth v4 with Google provider, restricted to `@ebli.com` domain
- Gates all app routes via `middleware.ts`
- Session includes role from `ROLE_MAP`

**Layer 2 — LearnUpon session (internal API access):**
- Admin manually enters their LearnUpon email/password in the app UI
- Credentials encrypted into a JWT cookie (`lms-session-creds`) using `jose` + `NEXTAUTH_SECRET`
- Server-side: decrypts cookie → logs in to LearnUpon → obtains session cookie + XSRF token
- Session is cached per-user, auto-refreshed on expiry, cleared on auth failure
- Required for ALL learning journey operations (no public API for these)

**Key files:**
- `src/app/api/lms-session/login/route.ts` — encrypts + stores credentials, probes session for conflict detection
- `src/app/api/lms-session/logout/route.ts` — clears the credential cookie
- `src/app/api/lms-session/status/route.ts` — checks if session is active (fast JWT check, or deep LU probe with `?probe=true`)
- `src/lib/get-lms-session.ts` — decrypts cookie, gets/refreshes session, detects conflicts, logs all failure paths
- `src/lib/learnupon-session.ts` — login flow (with retry), session cache, `luSessionFetch()`, error classes

**LMS session connect/disconnect UX:**
- `lms-status-dot.tsx` component shows green (connected) / orange (disconnected) in the nav, with a hover tooltip explaining the status
- Admin clicks to connect → enters LMS credentials → stored in encrypted cookie
- Features requiring session API show "Connect to LMS" prompt if no session exists
- Session expires naturally (LearnUpon timeout) → auto-refreshes transparently using stored creds
- On login failure: shows specific error (credential issue, session conflict with other tabs, or transient failure) with actionable guidance

**Login resilience (August 2026):**
- `login()` retries once (1.5s delay) on transient network errors — auth errors and structural breaks fail immediately
- Session conflict detection: after fresh login, probes LU immediately — if 401 comes back instantly, returns 409 `SESSION_CONFLICT`
- All failure paths in `getLmsSession()` are logged with type-specific messages (was previously silent for non-structural errors)
- Error classes: `LearnUponSessionError` (auth), `LearnUponLoginStructureError` (schema change), `LearnUponSessionConflictError` (cross-tab conflict)

### Public API vs. Session API

| Operation | API | Wrapper |
|-----------|-----|---------|
| User search, all users | Public | `luFetch()` |
| Enrollment CRUD (search, create, delete, mark complete) | Public | `luFetch()` |
| Course search | Public | `luFetch()` |
| Group list, group memberships | Public | `luFetch()` |
| Group users (all members of a group, paginated) | Session | `luSessionFetch()` |
| Learning journey list (dashboard) | Session | `luSessionFetch()` |
| Journey course trees (workflow) | Session | `luSessionFetch()` |
| Journey enrollment state (`/users/states.json`) | Session | `luSessionFetch()` |
| Journey enroll/unenroll | Session | `luSessionFetch()` |
| Group triggers (auto-enrollment rules) | Session | `luSessionFetch()` |

**Quick heuristic:** If it touches learning journeys, group auto-enrollment triggers, or group user lists, it needs the session API. Everything else uses the public API.

## Knowledge: Architecture patterns

### API calls to LearnUpon
All outbound calls go through the request pipeline:
- `luFetch()` (in `src/lib/learnupon-client.ts`) — for public API (Basic Auth)
- `luSessionFetch()` (in `src/lib/learnupon-session.ts`) — for internal session API (cookie)
- Both use `pipelineFetch()` from `src/lib/request-pipeline.ts`
- Both resolve the portal URL dynamically via `getActivePortalConfig()` from `src/lib/portal.ts`

The pipeline provides:
- Concurrency limiting (5 concurrent max)
- Adaptive throttling (reads rate limit headers, adjusts speed bidirectionally)
- Retry with exponential backoff (429/503)
- Circuit breaker (5 failures in 30s → open 60s)
- GET request deduplication

### Portal switcher (Production / Sandbox)

The app supports two LearnUpon portals: production (API: `ebli.learnupon.com`, internal: `eblireads.learnupon.com`) and sandbox (`eblisandbox.learnupon.com` for both). Defaults to production.

**How it works:**
- Active portal stored in a cookie (`lms-portal`, non-httpOnly, 1-year maxAge)
- Server-side: `getActivePortalConfig()` from `src/lib/portal.ts` reads the cookie and returns the matching credentials/URLs
- Client-side: `getPortalBaseUrl()` from `src/lib/constants.ts` reads the cookie for UI links
- Both `luFetch()` and `luSessionFetch()` call `getActivePortalConfig()` internally — no call-site changes needed
- Switching portals clears: platform cache (`invalidateAll()`), LMS session, client localStorage

**Key files:**
- `src/lib/portal.ts` — `PortalConfig` type, `getActivePortalConfig()`, `getActivePortalId()`, `isSandboxConfigured()`
- `src/lib/constants.ts` — `getPortalBaseUrl()` (client-safe, reads cookie)
- `src/app/api/portal/switch/route.ts` — POST, ADMIN only, sets cookie + clears caches
- `src/app/api/portal/status/route.ts` — GET, VIEWER, returns active portal + sandbox availability
- `src/components/portal-switcher.tsx` — Nav UI toggle with confirmation dialog

**Design choice:** Portal resolution via `cookies()` from `next/headers` inside the existing `luFetch()`/`luSessionFetch()` functions means zero changes to the 15+ route handlers that call them. The cookie is the single source of truth.

**Routes that bypass `luFetch()` but are still portal-aware:**
- `/api/users/all` — uses `pipelineFetch` directly (streaming pagination), calls `getActivePortalConfig()` itself. Also sets `Vary: Cookie` on the edge cache header so Vercel CDN keeps separate cached responses per portal.
- `/api/health` — lightweight connectivity probe, calls `getActivePortalConfig()` directly.

### Caching layers
1. **Server-side platform cache** (`src/lib/platform-cache.ts`): 24h TTL ceiling, in-memory, shared across all users within a serverless instance. Practical lifetime = instance warmth (minutes to hours depending on traffic). Caches:
   - Journey list (all journeys with id/versionId/name)
   - Course trees per journey (ordered course IDs + names)
   - All groups (id/title/description/memberCount)
   - Group triggers per group (journey associations triggered by group membership)
2. **Client-side cache** (`src/lib/client-cache.ts`): 5-min TTL for LJ progress data per user. Prevents re-fetches when toggling sections.
3. **Edge cache** (Cache-Control header on `/api/users/all`): 1h s-maxage + 23h stale-while-revalidate. Uses `Vary: Cookie` so portal switches get separate cache entries on Vercel CDN.
4. **localStorage** (search page): 24h cache for all-users list.

**Cache invalidation:** `POST /api/cache/invalidate` (ADMIN only) clears all platform cache. Use when structural changes are made in LearnUpon (new groups, journey edits, etc.). `GET /api/cache/invalidate` returns current cache stats.

**What is NEVER cached:** User-specific data that changes constantly (enrollments, search results, group memberships, journey state per user). See `docs/caching.md` §What's NOT Cached for the full list and rationale.

### Smart enrollment check (call reduction)
The `/api/learning-journeys/progress` and `/api/learning-journeys/user` routes use a pre-filter (`src/lib/journey-pre-filter.ts`) to avoid checking all journeys for enrollment. Logic:
- Get the user's course enrollments (already needed downstream)
- For each journey, check if the user is enrolled in its first course (from cached course trees)
- Only call the expensive `/users/states.json` endpoint for journeys that pass the filter
- This reduces API calls by ~70-80% (e.g., 5 checks instead of 25)
- Bypass with `?fullCheck=true` for edge cases (user unenrolled from first course but retained in journey)
- UI shows "Checked X of Y journeys. Run full check" link when in smart mode

### Mutations
- Simple mutations: use `useMutation` hook + `MutationButton` component
- Complex multi-step operations: use step-based state machines with dialog components (already protected by design)

## Knowledge: Button variants (`src/components/ui/button.tsx`)

| Variant | Use for |
|---------|---------|
| `default` | Primary actions (submit, confirm, active tab state) |
| `destructive` | Dangerous actions (bulk delete, confirm delete) |
| `destructive-outline` | Inline destructive actions in tables (single-row delete) |
| `outline` | Neutral secondary actions (cancel, back, refresh, CSV download) |
| `secondary` | Inactive toggle/tab state |
| `success` | Positive/additive actions (enroll, create) |
| `warning` | High-impact caution actions (group switch, calculate impact) |
| `ghost` | Minimal-chrome buttons (icon-only, inline links) |
| `link` | Text that looks like a link but behaves as a button |

## Knowledge: Group switch preview

`POST /api/groups/switch-preview` is the most complex fan-out endpoint. It:
1. Takes a user email + current group ID + target group ID
2. Fetches the user's current enrollments (public API)
3. Looks up group triggers for both groups (cached)
4. Compares which learning journeys are triggered by each group
5. Returns what journeys/enrollments would be gained, lost, or retained

Uses both public API (enrollments) and session API (group triggers) plus platform cache. Heavy on cross-referencing — modify with care.

## Knowledge: Pipeline status endpoint

`GET /api/pipeline/status` returns the current state of the request pipeline:
- Queue depth (pending requests)
- Circuit breaker state (closed/open/half-open)
- Adaptive throttle level (current delay between requests)
- Concurrency slots in use

Useful for debugging slow responses or confirming the circuit breaker has tripped. Requires VIEWER role (minimum).

## Knowledge: Pure function utility modules (`src/lib/`)

For domain logic that is:
- Deterministic (same input → same output, no side effects)
- Used by API routes AND potentially by tests
- Complex enough to warrant isolated unit/property testing

Place in `src/lib/<domain>.ts` as exported pure functions. Examples:

- `src/lib/journey-engagement.ts` — `computeAttribution()`, `isJourneyNotStarted()`, `computeEngagementFromDate()`
- `src/lib/timeline-utils.ts` — shared timeline rendering (`toLocalDateKey()`, `groupCompletionsByDate()`, `computeDotPositions()`, etc.)
- `src/lib/enrollment-utils.ts` — `isTimestampOnTheHour()`, `statusPriority()`, `detectAdminCompletion()`, `buildCourseEnrollmentMaps()`
- `src/lib/workflow-parser.ts` — `walkWorkflowTree()`, `fetchCourseTreeFromLMS()` (shared fetcher callback for `getCourseTree()`)

This pattern keeps route handlers thin (they call the utility with data they already have) and makes property-based testing straightforward.

## Knowledge: File structure (key files, not exhaustive)
```
src/
  app/                    # Next.js App Router pages + API routes
    api/                  # Server-side API routes
      cache/              # Cache management endpoints
      pipeline/           # Pipeline status endpoint
      portal/             # Portal switcher (status + switch)
      enrollments/        # Enrollment CRUD
      learning-journeys/  # LJ progress, list, enroll, courses
      lms-session/        # LMS credential management
      users/              # User search, all users, groups
    enrollments/          # Enrollments page (user detail)
    login/                # Auth page
    page.tsx              # Search page (home)
  components/             # Shared React components
    ui/                   # Base UI components (button, input, select, mutation-button)
    enrollment-table.tsx  # Enrollment table with sort/filter/bulk actions (extracted from page)
    user-table.tsx        # Virtualized user directory table (extracted from page)
  hooks/                  # Custom React hooks
    use-mutation.ts       # Mutation state (loading/error/execute)
    use-role.ts           # Current user's RBAC role
    use-sortable.ts       # Generic sortable table state (sort field, direction, compare)
    use-api-query.ts      # Fetch-on-mount with abort-on-unmount, loading/error, dedup via apiFetch
  lib/                    # Server + client utilities
    request-pipeline.ts   # Outbound API protection (server-only)
    platform-cache.ts     # Server-side platform cache (journeys, courses, groups, triggers)
    portal.ts             # Portal config resolution (reads cookie, returns credentials/URLs)
    journey-pre-filter.ts # Smart enrollment check (first-course pre-filter)
    journey-engagement.ts # Pure function: positional attribution + engagement computation
    timeline-utils.ts     # Pure function: shared timeline rendering (dots, markers, overlaps)
    enrollment-utils.ts   # Pure function: admin completion detection, status priority, enrollment map builder
    workflow-parser.ts    # Pure function: walkWorkflowTree + fetchCourseTreeFromLMS (shared by 4 routes)
    api-helpers.ts        # Route helpers: withAuthAndRole, handleApiError, handleSchemaBreakError
    learnupon-client.ts   # Public API wrapper (portal-aware via getActivePortalConfig)
    learnupon-session.ts  # Session API wrapper (portal-aware via getActivePortalConfig)
    constants.ts          # Client-safe portal URL helper (getPortalBaseUrl)
    client-cache.ts       # Client-side TTL cache
    fetch-client.ts       # Client-side fetch wrapper (apiFetch — GET dedup + error events)
  types/                  # TypeScript type definitions
    learnupon.ts          # Raw LU types + normalized App types (AppUser, AppEnrollment, etc.)
    journey.ts            # Shared journey progress types (JourneyProgress, JourneyProgressCourse)
    user.ts               # Client-side user types (CachedUser, UserSortField, SortDir)
```

## Knowledge: Dev setup

### Dev command
```powershell
$env:NODE_TLS_REJECT_UNAUTHORIZED="0"; npm run dev
```

### Environment variables
See `.env.example`. Required: `LMS_PORTAL_URL`, `LMS_API_USERNAME`, `LMS_API_PASSWORD`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `NEXTAUTH_SECRET`. Optional (enables portal switcher): `LMS_SANDBOX_PORTAL_URL`, `LMS_SANDBOX_API_USERNAME`, `LMS_SANDBOX_API_PASSWORD`. Optional (when internal hostname differs from API hostname): `LMS_PORTAL_BASE_URL`, `LMS_SANDBOX_PORTAL_BASE_URL`.

**Dual-URL pitfall (August 2026):** LearnUpon uses different hostnames for the public API vs. the internal web UI. For EBLI: public API is `ebli.learnupon.com/api/v1/` but internal session endpoints live on `eblireads.learnupon.com`. If `LMS_PORTAL_BASE_URL` is not set, the app derives the internal URL from the API URL (strips `/api/v1/`) — which produces the wrong hostname. Always set `LMS_PORTAL_BASE_URL` when the hostnames differ.

## Knowledge: Security headers

`next.config.ts` sets security headers on all responses:
- `X-Frame-Options: DENY` — prevents clickjacking
- `X-Content-Type-Options: nosniff` — prevents MIME sniffing
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy: camera=(), microphone=(), geolocation=()` — disables unused browser APIs
- `X-DNS-Prefetch-Control: on`
- `Strict-Transport-Security: max-age=31536000; includeSubDomains` — enforces HTTPS
- `Content-Security-Policy` — enforces resource loading restrictions: same-origin only (`default-src 'self'`), allows Next.js inline scripts/styles, blocks iframes (`frame-ancestors 'none'`), plugins (`object-src 'none'`), and base/form hijacking. Violations are actively blocked and reported to `/api/csp-report`.

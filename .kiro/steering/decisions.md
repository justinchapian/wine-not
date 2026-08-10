---
inclusion: always
---
# Key Decisions & Rejected Approaches

> **Content type: Knowledge reference.** This file records active design decisions and rejected approaches. Rules derived from these decisions live in their respective steering files (those files enforce *what*). This file explains *why* — and prevents agents from proposing approaches that have already been evaluated and rejected. See `feature-planning.md` RULE: "Decision graduation" for when entries can be removed.

## Design Decisions (user-defined, don't change without asking)

### Engagement thresholds
- Active: completion within last 7 days
- Slowing: last completion 7-14 days ago
- Stalled: last completion 14-30 days ago
- Stopped: no completions in 30+ days
- These are business rules defined by the user, not arbitrary values

### Engagement timeline visualization
- Proportional dot timeline (not bar graph, not heatmap, not git-graph)
- One dot per day with completions; dot size scales with count (continuous linear scale, 10 steps from count 1 to 10+, via `getDotSizePx()` in `timeline-utils.ts`)
- Size caps at 10+ completions; dots with ≥10 show a numeric count label above them for at-a-glance magnitude
- Count labels appear on ALL dot types (regular, admin, certification star, first completion, journey complete)
- True time-scale axis — gaps represent real elapsed inactivity
- Week markers aligned to calendar Sundays (not counted from start date)
- Month dividers as thicker lines; week lines as thinner lines
- Month markers always positioned on the 1st of each calendar month (use `new Date(year, month, 1)` constructor — never `setMonth` to avoid day-rollover bugs)
- 6-month view starts on the 1st of the month 6 months ago (not today minus 180 days)
- Month labels use short month name only ("Jan", "Feb") — never include 2-digit year suffix (avoids confusion with day-of-month)
- Full history modal shows per-month completion counts above the timeline (centered between month markers)
- Cluster overlap resolution: spreads overlapping dots with small MIN_GAP (0.8% in 6-month, 0.6% in history), but skips spreading if dots already have sufficient natural spacing
- Shows up to 6 months, or full timeline if enrollment is shorter
- Full history modal duration uses calendar month calculation (not days/30 approximation)
- For completed journeys: end date = completedAt (not today)
- Tooltips on hover showing date + count (pure CSS, no state)
- Pace calculation: first course start date to completedAt (or now)

### User search page
- Page titled "Users" (not "User Search") — acts as a user directory with inline filtering
- Show all ~3000 users by default (no minimum character search)
- Default filter: Active users (cuts display in half)
- Virtualized table for performance
- 24h localStorage cache + Vercel edge cache with freshness indicator ("Last refreshed Xh ago")
- All filter/sort/query state persisted in URL params
- Dismissible first-visit hint guides new users on click-to-view and bulk select
- Filter count badge + "Clear all" link when filters are active
- LMS connection banner appears above search (not buried below results)
- Group memberships loaded on-demand via "Load Groups" button (not eagerly with page load)
  - Requires LMS session (internal endpoint `/groups/{id}/users.json`, 10 per page)
  - Too expensive to load eagerly (~15-40s for all groups, chews rate limits)
  - Once loaded: adds "Groups" column to table, group filter dropdown replaces the button
  - Data lives in component state (session lifetime — lost on refresh, acceptable)
  - Sort by group name supported

### Enrollments page
- All collapsible sections start CLOSED
- User info banner is persistent (not collapsible)
- User info banner does NOT fetch enrollments independently (receives data from parent)
- "User Engagement" collapsible section shows multi-row engagement graph (between UserInfoBanner and Group Memberships)
- User engagement graph data: LJ progress fetched via client-cache pattern (same cache key as LearningJourneyReports — no duplicate fetches)
- `lastSignInAt` shared from UserInfoBanner via `onUserLoaded` callback (avoids duplicate user fetch)

### User engagement graph (multi-journey)
- Multi-row 6-month proportional dot timeline — one row per learning journey + "Other" row
- Same X-axis, month/week markers, and dot sizing as existing single-journey timeline
- "Other" row: completions from courses NOT appearing in any of the user's current learning journey course trees
- Last login: distinct vertical dashed line (blue, dashed) spanning all rows with "Last login" label
- Row height: 28px per journey; track auto-sizes to content
- Journey labels truncated at 130px with title tooltip for full name; completed journeys show "✓"
- Journeys with zero completions in the 6-month window are omitted from the graph (avoids empty rows)
- Empty state: shows engagement badge + last login date + "No completions in the last 6 months" message
- "View Entire History" button opens full-history modal

### Course completion time box plot
- Horizontal box and whisker plot showing distribution of completion times (dateStarted → dateCompleted)
- Positioned below the engagement graph within the same "User Engagement" collapsible section
- Excludes ALL admin-completed courses (`markedCompleteByAdmin` OR `suspectedAdminComplete`) — shows count of excluded
- Uses dateStarted (NOT dateEnrolled) as the start point — measures actual active time, not wait time
- Requires at least 2 valid data points to render; otherwise shows explanatory empty state
- Standard box plot: Q1–Q3 box, median line (blue), mean diamond (amber), whiskers to min/max within 1.5×IQR fences, outlier dots (red) beyond fences
- Duration displayed in hours (<1 day), days with decimal (<7 days), or rounded days (≥7 days)
- Pure CSS tooltips on mean marker and outlier dots (no state, same pattern as timeline dots)
- Axis ticks auto-generated with nice step sizes (1, 2, 5 multiples)
- Component: `src/components/completion-time-box-plot.tsx`
- No additional API calls — consumes existing `enrollments[]` already fetched centrally on the enrollments page

### User engagement history modal (all-time)
- Timeline span: earliest enrollment date to today
- Duration threshold: >18 months uses histogram bars (per-month counts), ≤18 months uses dots
- Histogram bars: 6px wide, height normalized to the max count across all rows, centered on month midpoint
- Dot mode: same multi-row layout as 6-month view but with 0.6 MIN_GAP and year in tooltips
- Last login vertical dashed line included in both modes
- Month labels show every 3rd marker in histogram mode (prevents crowding), every 2nd in dot mode
- Row labels wider (160px) for better readability in the full-width modal

### UI patterns
- "Journey Progress" label on segmented status bar (shows position + per-course status at a glance)
- "Completion Activity" label on dot timeline (shows when work happened over time)
- Both use matching `text-xs font-semibold text-foreground` style

### Journey progress bar (segmented)
- One rectangle per course, equal width, colored by enrollment status
- Color mapping (aligned with StatusBadge and timeline): green-500 = completed/passed, blue-500 = in_progress, gray-300 = not_started, white+border = not_enrolled, red-500 = failed, yellow-500 = pending_review
- Crossover indicator: diagonal stripe overlay (gray stripes for confirmed, amber stripes for possible)
- Admin completion indicator: purple dot in top-right corner of segment (aligns with purple-500 admin dots in timeline)
- Current position: blue downward-pointing arrow above the segment with "Current" label
- Contextual legend below bar (only shows statuses/markers present in the data)
- Used in JourneyCard (full legend) and JourneyDetailModal (no legend, table provides detail)
- Component: `src/components/journey-progress-bar.tsx`
- No additional API calls needed — consumes existing `courses` array from progress API (status, isCrossover, isAdminCompleted, isCurrent already present)

### Platform caching

> **See also:** `docs/caching.md` for the full architecture reference (TTL behavior, what's cached, how to add new cached data)
- In-memory cache-through pattern (no Redis, no cron, no external dependencies)
- Platform-wide data (groups, journeys, course trees, triggers) cached — shared across all users within a serverless instance
- User-specific data (enrollments, memberships) always fetched live
- Manual invalidation via admin endpoint when LearnUpon structure changes
- 24h TTL ceiling; practical lifetime = Vercel instance warmth
- Cache is NOT portal-scoped — switching portals clears the entire cache via `invalidateAll()`

### Portal switcher
- Cookie-based portal resolution (`lms-portal` cookie, non-httpOnly, 1-year expiry)
- Defaults to production when cookie is absent
- Server-side: `getActivePortalConfig()` reads cookie via `cookies()` from `next/headers` inside `luFetch()`/`luSessionFetch()` — zero call-site changes to existing route handlers
- Client-side: `getPortalBaseUrl()` reads cookie from `document.cookie` for UI links
- Switching clears: platform cache, LMS session (different portal = different session), client localStorage (users-*/lms-* keys)
- ADMIN-only action (VIEWER cannot switch portals)
- Sandbox mode shows amber "SANDBOX" badge in nav bar (visually distinct from subtle "PROD" to prevent accidental production operations)
- Sandbox credentials are separate env vars — portal switcher UI only appears when all three sandbox vars are configured
- **Pitfall:** Routes that bypass `luFetch()` (like `/api/users/all` using `pipelineFetch` directly) must still call `getActivePortalConfig()` — never read `process.env.LMS_*` directly. Edge-cached responses need `Vary: Cookie` to avoid serving stale data from the wrong portal.

### LearnUpon dual-hostname discovery (August 2026)
- **Problem:** LearnUpon uses different hostnames for the public REST API vs. internal web UI/session endpoints. For EBLI production: API lives on `ebli.learnupon.com`, session/internal endpoints live on `eblireads.learnupon.com`. The original code derived the internal base URL from the API URL by stripping `/api/v1/`, which produced the wrong hostname.
- **Symptom:** LMS session login returned 401 ("invalid credentials") because credentials were POSTed to the wrong host. Public API (users, enrollments) worked fine because it uses the API URL directly with Basic Auth.
- **Fix:** Added `LMS_PORTAL_BASE_URL` env var (and `LMS_SANDBOX_PORTAL_BASE_URL` for sandbox). When set, overrides the derived base URL. Optional with backwards-compatible fallback to `deriveBaseUrl()` for portals where both hostnames are the same (like sandbox: `eblisandbox.learnupon.com` for both).
- **Why it was hidden:** Sandbox uses the same hostname for both API and web UI, so testing against sandbox always worked. The discrepancy only surfaced when testing production login after adding sandbox support.

### Internal UI endpoint resilience
- All `/angie/...` paths centralized in `src/lib/lu-internal-paths.ts` — never inline in route handlers
- Response shape guards (`src/lib/lu-response-guards.ts`) validate structure before field access — fail loud with 502, never silent fallback
- Login structural failures (`LearnUponLoginStructureError`) logged distinctly from credential failures or "user not connected"

### LMS login resilience (August 2026)
- **Approach:** Diagnostic-first, fix-second. Root cause of inconsistent login failures is uncertain (could be transient network, session conflicts, rate limiting, or cold starts). Instead of engineering for one hypothesis, we added retry + logging + conflict detection to cover all cases and surface what's actually failing.
- **Retry:** `login()` retries once on transient network errors (1.5s delay). Auth errors and structural breaks are NOT retried — they won't resolve on retry.
- **Conflict detection:** After fresh login, immediately probe LU. If 401 returns instantly → session conflict (another browser tab invalidated ours). Returns HTTP 409 with `SESSION_CONFLICT` code.
- **Logging:** All failure paths in `getLmsSession()` now log with type-specific messages. Previously swallowed silently — impossible to diagnose production failures.
- **User-facing errors:** Specific, actionable messages (credential issue vs. conflict vs. transient). Includes hint about logging out of LU in other tabs.
- **Status probe:** `/api/lms-session/status?probe=true` actually tests LU session validity (without param: fast JWT check only, backwards compatible).
- **Not an alert:** Login failures don't trigger email alerts — the admin can't do anything about it remotely. The improved error messages guide the user to self-fix.
- No artificial delay/human pacing for bot detection — pipeline's concurrency limiter (5) + adaptive throttler provide sufficient natural jitter; LearnUpon doesn't run bot detection on internal JSON endpoints
- Schema break detection feeds into email alerting (Task 13) when implemented

### Email alerting (schema breaks)
- Provider: Resend (free tier, one env var `RESEND_API_KEY`, ~50KB package)
- Recipients: derived from `ROLE_MAP` in `src/lib/roles.ts` (all ADMIN emails) — no separate config
- Deduplication: 1-hour in-memory cooldown per endpoint per serverless instance
- Delivery: uses `after()` from `next/server` to schedule the email send after the response is returned — keeps the serverless invocation alive until the Resend API call completes, without blocking the 502 response to the client
- Graceful degradation: missing API key → console warning, no crash
- Health check: `GET /api/health/internal-endpoints` — manual trigger only, requires ADMIN + LMS session
- Runbook: `.kiro/steering/schema-break-runbook.md` (manual inclusion) for repeatable fix workflow across sessions

### Smart enrollment check
- Default: pre-filter journeys by first-course enrollment before making expensive per-journey API calls
- User explicitly accepted the edge case tradeoff (completed journeys with first course removed may be missed)
- Full Check bypass (`?fullCheck=true`) available via UI button for when admin suspects something is off
- UI shows "Checked X of Y journeys. Run full check" to keep admin informed

### Engagement attribution: positional via `currentPosition`
- Courses at positions 1 through `currentPosition` are classified as **journey-enrolled**; courses beyond `currentPosition` with any enrollment status (not_started, in_progress, completed, passed) are classified as **crossover** — meaning another journey enrolled the user. Courses beyond `currentPosition` with status `not_enrolled` are simply future courses, NOT crossover.
- **Refinement (August 2026):** Initial implementation used `position > currentPosition` alone for crossover classification. This incorrectly marked future un-enrolled courses as crossover. Fixed to require `position > currentPosition AND status !== "not_enrolled"` — a course is only crossover if another journey has actually enrolled the user in it.
- Engagement level (Active/Slowing/Stalled/Stopped) computed from journey-enrolled completion dates only — crossover completions excluded
- Uses the existing `currentPosition` value from the progress API (derived from LearnUpon's sequential workflow state)
- **Rejected alternative: contiguous boundary walk** — walking enrollment records from position 1 and stopping at the first gap. This would handle edge cases where `currentPosition` doesn't perfectly reflect enrollment boundaries, but adds complexity and extra data joins with no observed real-world benefit. `currentPosition` already represents where the journey's auto-enrollment has reached.
- The simpler positional approach can be upgraded later if edge cases surface in production
- All attribution logic runs server-side as a pure function (`src/lib/journey-engagement.ts`) — no additional API calls, no client-side re-derivation

### Crossover course identification: full model ✅ IMPLEMENTED (August 2026)

> **See also:** `docs/crossover-courses.md` for the complete knowledge article (3 principles, implementation details, edge cases)
- **Full knowledge article:** `docs/crossover-courses.md`
- **Principle 1 — Shared course set:** A course can only be a crossover if it appears in 2+ learning journeys. Derived from cached course trees at zero API cost via `getSharedCourseIds()` in `platform-cache.ts`. Hard constraint, not heuristic.
- **Principle 2 — Enrollment ID monotonicity:** Journeys enroll courses sequentially (auto-increment IDs). Within one journey, enrollment IDs at positions 1, 2, 3... must be monotonically increasing. A break in monotonicity proves another source created that enrollment first. LearnUpon reuses existing enrollments rather than creating duplicates.
- **Principle 3 — `dateEnrolled` correlation:** If a course's `dateEnrolled` predates the journey's `enrolledAt`, it was pre-existing → confirmed crossover. If dates are ambiguous, check next-course-enrollment tiebreaker.
- **Next-enrollment tiebreaker:** If this course's `dateCompleted` is within 5 minutes of the next course's `dateEnrolled`, this journey drove the completion (sequential auto-enrollment triggered) → not crossover.
- **Three-value classification (`CrossoverStatus`):** `false` = journey-enrolled, `true` = confirmed crossover, `"possible"` = ambiguous (enrollment ID breaks but can't confirm source).
- **Timeline behavior:** Confirmed crossovers (`true`) excluded from timelines. Possible crossovers (`"possible"`) included (benefit of the doubt — don't strip data on uncertainty).
- **Engagement computation:** Confirmed crossovers excluded from engagement. Possible crossovers included.
- **Source attribution (deferred):** Not yet implemented. When needed, cross-reference crossover enrollment timing against other journeys containing that course to identify the source. All data is already available.
- **EBLI context:** All courses are enrolled through learning journeys — no self-enrollments. Standalone enrollments may exist for test users or admin accounts but are not part of the normal flow.
- **Admin completion exception:** Bulk admin operations create enrollments out of sequential order (already detected by admin-complete heuristic). Excluded from enrollment ID sequence analysis via anchor-point logic (only non-shared courses establish monotonic baseline).
- **Display:** Course status table shows "↗ Yes" (gray) for confirmed, "↗ Possible" (amber) for ambiguous. Excel exports show "Yes"/"Possible"/blank.

## Abandoned/Rejected approaches
- **Cross-tab session coordination (BroadcastChannel + localStorage)**: Copilot suggested coordinating session ownership across browser tabs using BroadcastChannel API. Rejected because the actual conflict is between our app and the user's *direct LearnUpon browser tab* (different origin), not between multiple tabs of our app. Our app uses one shared HttpOnly JWT cookie — it can't conflict with itself. No client-side JavaScript can detect or prevent LearnUpon's server-side session invalidation from another origin.
- **Login failure email alerts**: Considered alerting admins when users can't connect. Rejected because the admin can't do anything about it remotely — the fix is always "user closes their other LU tab." The improved error messages guide users to self-fix.
- **Impersonation feature**: Requires same browser session, can't proxy from server. Abandoned.
- **Portal switcher via parameter passing (Option A)**: Passing portal ID as a parameter through `luFetch()`/`luSessionFetch()` would require changing signatures on 15+ route handler call sites. Rejected in favor of cookie-based resolution via `cookies()` from `next/headers` (Option B) — zero call-site changes.
- **Per-portal cache scoping**: Considered prefixing all platform cache keys with the portal ID. Rejected as over-engineering — switching portals infrequently, and `invalidateAll()` on switch is simpler with no stale-data risk.
- **Single-color percentage bar for journey progress**: Only communicated position OR completion — couldn't show both. Replaced with segmented status bar (one colored rectangle per course).
- **Bar graph timeline**: Hard to interpret — replaced with dot timeline
- **Heatmap**: Too empty with sparse data (10-30 completions over 6 months)
- **Git-graph style**: Doesn't clearly show time intervals
- **Bucketed timeline (per-week/month)**: Loses proportional gaps that show real inactivity
- **Inbound rate limiting**: Overkill for <10 internal admins on Hobby plan
- **@upstash/ratelimit**: Requires Redis subscription, unnecessary at this scale
- **Redis/KV for caching**: Adds infrastructure cost and latency for data that fits trivially in memory. Unnecessary at this scale.
- **Vercel Cron for cache warming**: Not needed — cache-through pattern handles cold starts gracefully. First request of a session absorbs the cost.
- **Caching "all courses" from REST API**: Course names/IDs already come through journey course tree cache or enrollment data; no standalone use case.
- **Artificial delay / human pacing for internal endpoints**: Pipeline concurrency (5 max) + adaptive throttler already produce natural jitter. Adding 100-300ms blanket delays would degrade bulk operations from ~10s to ~40s. LearnUpon is B2B SaaS with API-layer rate limiting, not consumer-facing bot detection on JSON endpoints.
- **Generic "utility shield" fallback pattern** (returning empty arrays on parse failure): Hides broken features behind misleading "no data" UI. Fail-loud with 502 + schema break alerting is superior for an admin tool.
- **Gmail API for alerting**: Requires OAuth scope expansion, service account JSON, complex token refresh. Way too much machinery for ~1-2 alert emails per month.
- **Nodemailer + Google SMTP**: Requires app-specific password, breaks with 2FA changes, Google periodically blocks "less secure apps."
- **Automated cron health check**: Over-engineering — the alert email already tells you something broke. Manual health check on demand gives full picture without unnecessary invocations.
- **Bare fire-and-forget for serverless background work**: Calling an async function without `await` on Vercel serverless means the runtime can freeze/terminate before the promise completes. Replaced with `after()` from `next/server` which keeps the invocation alive until the callback finishes. All alert email sends now use `after()`.



---
title: Roadmap
intent: planned-work
area: docs
---
# Roadmap — Planned Work & Tech Debt

Items here are planned but not yet implemented. When an item ships, **remove it from this file** (the code is the record — see anti-drift rules in `feature-planning.md`).

---

## Planned Features

### Admin Dashboard Stats

Engagement metrics, time to proficiency, progression stats. No design work done yet.

### Active Users Filter (tentative)

When looking at admin dashboard data, only show active users in learner status. Needs more thought on whether this is useful.

---

## Tech Debt

### Lint violations cleanup

40 pre-existing lint errors/warnings remain across the codebase. None are runtime bugs but they degrade signal-to-noise in lint output.

**Priority 2 — React 19 compiler compatibility (26 errors):**
- `react-hooks/set-state-in-effect` (23): calling `setState` synchronously in `useEffect` across 14 components. Patterns work today but prevent React Compiler from auto-memoizing these components. Fixing means replacing `useEffect` + `setState` with derived state, `useSyncExternalStore`, or data-fetching libraries.
- `react-hooks/immutability` (2): accessing a hoisted function before its declaration inside `useEffect`. Blocks compiler data-flow tracking. Fix by moving the function above the effect or converting to `useCallback`.
- `react-hooks/incompatible-library` (1): `useVirtualizer` from TanStack Virtual returns functions that can't be memoized. Not fixable without upstream library change — compiler skips this component automatically.

Approach: Do this when actively enabling React Compiler for production. Measure render performance before/after to validate the lift is worthwhile. High effort (~14 components), moderate risk (lifecycle behavior changes). The `incompatible-library` violation is informational and requires no action.

**Priority 4 — Structural (6 files over 400-line limit):**
- `src/app/page.tsx` — 709 lines. Decompose into section components per `code-structure.md` extraction rules. The main search page orchestrator with all-users fetch, filtering, virtualized table coordination. Needs 3-4 extracted section components.
- `src/components/enrollment-table.tsx` — 632 lines. Large table with inline sort, filter, bulk actions, CSV export. Consider extracting the CSV logic or bulk-action toolbar.
- `src/components/enroll-in-course-dialog.tsx` — 481 lines. Multi-mode enrollment dialog. Could split journey-mode and search-mode into sub-components.
- `src/components/learning-journey-reports.tsx` — 480 lines. Journey cards with detail modals. Could extract the modal or individual card rendering.
- `src/components/user-engagement-history-modal.tsx` — 414 lines. Borderline. The histogram computation could be a utility.
- `src/components/switch-group-dialog.tsx` — 410 lines. Borderline. Multi-step dialog with impact preview.

Approach: `page.tsx` is the highest-value target (309 lines over limit). Do as a focused extraction session. The 410-481 line files are borderline — address incrementally when making related changes to those files.

**Also present (8 warnings, non-blocking):**
- `react-hooks/exhaustive-deps` (8): missing dependencies in effect/callback arrays. Some are intentional fire-once effects, some are genuine oversights. Review case-by-case alongside the Priority 2 work.

### `next-auth.d.ts` role type is optional but used without null checks

`role?: Role` in the type augmentation, but `withAuthAndRole` injects it reliably. TypeScript won't catch misuse outside the helper pattern. Consider making it non-optional.

### GitHub Actions CI pipeline

No CI exists. Deploys go straight from `git push` to Vercel production with no automated gate. Add a GitHub Actions workflow that runs on every push and PR:

```
lint (eslint)
  ↓
type check (tsc --noEmit)
  ↓
test (vitest --run)
  ↓
build (next build)
```

Requirements:
- Runs on `push` to `main` and on all pull requests
- Uses Node 20 (matches Vercel runtime)
- Caches `node_modules` for speed
- Fails the PR if any step fails
- Consider: add `--coverage` flag to vitest and fail if coverage drops below a threshold (once baseline is established)

File: `.github/workflows/ci.yml`

This creates the "prove it" gate — the AI proposes a change, CI verifies it didn't break anything. Without this, `npm run build` passing locally is the only verification, and it catches type errors but not behavioral regressions.

---

## AI-Driven Development Improvements

Items that improve how AI agents work with this codebase — constraints and context that a human developer would "just know" but an AI needs to be told.

### Portal context drift warning in `safety.md`

Add a rule about confirming portal context before mutations when the user has switched portals mid-session. The dual-portal architecture makes this a real trap for AI agents that lose track of which portal is active.

### Cache invalidation post-mutation table in `api-conventions.md`

Add a mutation→cache-invalidation mapping table. Currently no explicit connection exists between "I mutated data" and "which cache entries are now stale." Prevents the classic bug where a new mutation endpoint works but the UI shows stale cached data.

### Audit logging rule in `api-conventions.md`

Add guidance on *what* must be audited (enrollments, portal switches, group changes, bulk operations). Currently `audit()` exists but there's no rule about when it's required.

### Engagement invariants in domain docs

Extract the business rules from the property-based test assertions into a brief readable section in `admin-completion-detection.md` or `crossover-courses.md`. Currently these invariants exist only implicitly in 300+ lines of test code.

### Rate limit budget note in `tech.md`

Document the actual LearnUpon rate limit, what the throttle is tuned to, and the impact of bulk operations on other portal users. Gives AI the constraint needed to propose batched/staggered approaches.

### API response shape snapshots

Add reference shapes for guarded LearnUpon endpoints (as `docs/api-shapes/` or a single file). Aids schema break debugging and gives AI context for writing new guards without hitting the live API.

### Kiro post-save lint hook

A `PostFileSave` hook running ESLint on changed files during the session. Faster in-session feedback loop for structural violations. Not a CI substitute.

---

## Deferred (bring up when relevant)

### Resend domain verification

Need DNS TXT records on ebli.com (Bluehost/Unified Layer) to send alert emails to all ADMINs. Until verified, alerts only deliver to the Resend account owner. Once verified, switch `getAlertRecipients()` in `src/lib/alert-email.ts` back to pulling from `ROLE_MAP`.

### RBAC via environment variable

Move `ROLE_MAP` from hardcoded values in `src/lib/roles.ts` to an env var (JSON string). Consider when team grows beyond current admins. Tradeoff: loses git history of role changes.

### Automated engagement alerts

Daily Vercel cron to detect stalled users and send Slack/email digest. Defer until engagement tools are stable and usage patterns are clear.

### Retry/backoff as user-facing feature

If pipeline retry exhausts, show "try again in X seconds" instead of generic error. Nice UX but not blocking.

### Inbound rate limiting

If usage grows beyond 10 admins, add per-user soft caps. Not needed at current scale.

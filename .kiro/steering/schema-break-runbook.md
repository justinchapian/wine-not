---
inclusion: manual
---
# Schema Break Runbook

> **Content type: Procedural runbook (step-by-step).** This is not a rule set — it's an incident response procedure. Load it when responding to a schema break alert email or when manually verifying internal endpoint health.

Use this when you receive a schema break alert email, or when you want to verify all internal endpoint health.

## 1. Run Health Check

Hit the health check endpoint (requires admin login + LMS session connected):

```
GET /api/health/internal-endpoints
```

This probes all 5 internal endpoint types and runs their response guards. Returns a clear pass/fail report with timing.

## 2. Interpret Results

Each endpoint returns one of:

- **"ok"** — endpoint responds 200 and passes its guard. Working correctly.
- **"schema_break"** — endpoint responds 200 but the response structure has changed. This is the most common and dangerous failure mode. The `error` field contains the first 500 chars of the unexpected response.
- **"http_error" with 404** — the path has moved. LearnUpon restructured their internal routes.
- **"http_error" with 401/403** — session mechanism changed. The cookie or CSRF flow may have been altered.
- **"network_error"** — fetch failed entirely. LMS may be down, or there's a connectivity issue.

## 3. Fix Based on Error Type

### Path Changed (http_error 404)

1. Open LearnUpon in a browser (`eblireads.learnupon.com`), open DevTools Network tab
2. Navigate to the feature that uses this endpoint:
   - **dashboard** → Learning Journeys admin page
   - **workflow/courses** → Open a specific learning journey
   - **user-states** → View enrolled users in a journey
   - **group-triggers** → Group settings → auto-enrollment triggers
3. Find the new path in the network requests (filter by XHR/Fetch)
4. Update the path in `src/lib/lu-internal-paths.ts`

### Schema Changed (schema_break)

1. The health check response `error` field contains the actual response sample
2. Compare with what the guard expects — see `src/lib/lu-response-guards.ts`
3. Common changes:
   - Field renamed (e.g., `external_workflow_id` → `workflow_id`)
   - Envelope changed (e.g., `data` wrapper removed or renamed)
   - Nested structure flattened or deepened
4. Update the guard function to match the new structure
5. Update the parsing logic in the affected route handler(s) that access the changed fields
6. **Cross-check:** verify the guard's field checks match what the route handler maps *after* the guard passes. A mismatch (guard checks field X, route reads field Y) means the guard was written incorrectly — not that the LMS changed.

### Login Structure Changed (login-structure alert)

1. Try signing in to LearnUpon manually in a browser
2. Check DevTools for `/users/sign_in.json`:
   - Does it still exist? (path change)
   - Does it still return `Set-Cookie` with `_LearnUpon_session`? (cookie name change)
3. Check if the CSRF meta tag (`csrf-token`) is still in the HTML at `/users/sign_in`
4. Update `src/lib/learnupon-session.ts` `login()` function accordingly

## 4. Verify Fix

1. Re-run health check: `GET /api/health/internal-endpoints` — all should be "ok"
2. Clear platform cache: `POST /api/cache/invalidate` (stale cached data from before the break)
3. Test the affected feature in the app UI

## 5. Key Files Reference

| File | What to change |
|------|---------------|
| `src/lib/lu-internal-paths.ts` | Endpoint paths (when paths move) |
| `src/lib/lu-response-guards.ts` | Response shape validators (when schemas change) |
| `src/lib/learnupon-session.ts` | Login flow (when auth mechanism changes) |
| `src/lib/alert-email.ts` | Alerting logic (recipients, cooldown, format) |
| `src/app/api/health/internal-endpoints/route.ts` | Health check probes |

## 6. After Fixing

- The alert won't fire again for the same endpoint for 1 hour (deduplication cooldown)
- If you fix the issue within that hour, normal operation resumes silently
- If a different endpoint breaks, it gets its own separate alert immediately

## 7. Troubleshooting: Alert Email Not Received

If a schema break occurs but no email arrives:

1. **Check `RESEND_API_KEY`** — must be set in Vercel environment variables (Production + Preview). If missing, the app logs `[Alert] RESEND_API_KEY not configured` and silently skips.
2. **Check Resend domain verification** — until `ebli.com` is verified, emails can only deliver to the Resend account owner (`justin@ebli.com`). Other ADMIN emails will silently fail.
3. **Check `after()` usage** — alert emails must be sent via `after()` from `next/server` (not bare fire-and-forget). Without `after()`, Vercel serverless freezes the invocation before the HTTP call to Resend completes.
4. **Check cooldown** — if the same endpoint already triggered an alert within the last hour, the deduplication logic suppresses the second one. Wait 1 hour or redeploy (clears in-memory cooldown map).

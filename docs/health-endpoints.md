---
title: Health & Monitoring Endpoints
intent: architecture-reference
area: docs
---
# Health & Monitoring Endpoints

> **Content type: Knowledge reference.** This is an architecture reference for the health and monitoring endpoints. No auditable rules — consult when adding monitoring, debugging connectivity, or responding to alerts.

## Overview

Three endpoints provide visibility into the app's connection to LearnUpon and the health of the outbound request pipeline. They serve different audiences and answer different questions.

| Endpoint | Auth Required | Purpose |
|----------|--------------|---------|
| `GET /api/health` | None | Connectivity + pipeline status (for uptime monitors) |
| `GET /api/health/internal-endpoints` | ADMIN + LMS session | Schema validation of internal LearnUpon endpoints |
| `GET /api/pipeline/status` | VIEWER | Real-time pipeline metrics (for the admin UI) |

---

## GET /api/health

**Auth:** None (public)
**Source:** `src/app/api/health/route.ts`

The primary uptime probe. Designed to be called by external monitoring (e.g., Vercel cron, UptimeRobot) without authentication.

### What it checks

1. **LearnUpon connectivity** — makes a raw `fetch()` to the LMS public API (`users?page=1`) with Basic Auth and a 10-second timeout. This catches credential rotation failures and network issues.
2. **Pipeline infrastructure** — reads the current state of the request pipeline (circuit breaker, throttler, semaphore, deduplicator) via `getPipelineStatus()`. If the circuit breaker is open, the endpoint reports `"degraded"` even when raw connectivity succeeds.

### Response shape

```json
{
  "status": "healthy | degraded",
  "learnupon": "reachable | responded with {status} | unreachable: {message}",
  "portal": "portal-id",
  "pipeline": {
    "remainingMinute": 28,
    "remainingWeek": 4500,
    "currentDelayMs": 200,
    "circuitState": "closed | open | half-open",
    "activeConcurrent": 2,
    "waitingInQueue": 0,
    "deduplicatedInFlight": 1
  },
  "timestamp": "2025-01-15T10:30:00.000Z"
}
```

### Status logic

| Condition | `status` | HTTP |
|-----------|----------|------|
| LearnUpon reachable AND circuit closed/half-open | `healthy` | 200 |
| LearnUpon reachable BUT circuit open | `degraded` | 503 |
| LearnUpon non-200 response | `degraded` | 503 |
| LearnUpon unreachable (timeout/network) | `degraded` | 503 |

### What it does NOT check

- Response schemas (use `/api/health/internal-endpoints` for that)
- Session-based internal endpoints (only tests the public API)
- Full pipeline execution (it reads pipeline state but does not send the probe request through the pipeline — this is intentional to avoid the probe itself being blocked by a stuck pipeline)

---

## GET /api/health/internal-endpoints

**Auth:** ADMIN role + active LMS session (cookie-based)
**Source:** `src/app/api/health/internal-endpoints/route.ts`
**Max duration:** 60 seconds

A manual diagnostic tool that probes LearnUpon's internal (undocumented) endpoints and validates their response shapes against the app's guard functions.

### When to use

- After receiving a schema break alert email
- After a LearnUpon platform update
- When learning journey features behave unexpectedly

### What it probes

| Probe | Endpoint | Guard |
|-------|----------|-------|
| Dashboard | Internal dashboard listing | `guardDashboard()` |
| Workflow | Journey workflow structure | `guardWorkflow()` |
| Courses | Journey course tree | `guardCourseList()` |
| User States | User progression per journey | `guardUserStates()` |
| Group Triggers | Automation triggers per group | `guardGroupTriggers()` |

### Response shape

```json
{
  "timestamp": "2025-01-15T10:30:00.000Z",
  "sessionValid": true,
  "status": "healthy | degraded",
  "results": [
    {
      "endpoint": "dashboard",
      "status": "ok | schema_break | http_error | network_error",
      "responseTimeMs": 245,
      "error": null,
      "httpStatus": null
    }
  ],
  "summary": {
    "total": 5,
    "healthy": 5,
    "broken": 0,
    "httpErrors": 0,
    "networkErrors": 0
  }
}
```

### Error categories

- `ok` — Endpoint responded and passed schema guard
- `schema_break` — Responded but shape doesn't match expectations (LearnUpon changed something)
- `http_error` — Non-200 response
- `network_error` — Request failed (timeout, DNS, etc.) or no sample data available to probe

---

## GET /api/pipeline/status

**Auth:** VIEWER role (minimum)
**Source:** `src/app/api/pipeline/status/route.ts`

Lightweight real-time snapshot of the request pipeline's internal state. Used by the admin UI to display rate limit budgets and pacing info.

### Response shape

```json
{
  "remainingMinute": 28,
  "remainingWeek": 4500,
  "currentDelayMs": 200,
  "circuitState": "closed",
  "activeConcurrent": 2,
  "waitingInQueue": 0,
  "deduplicatedInFlight": 1
}
```

### Field reference

| Field | Meaning |
|-------|---------|
| `remainingMinute` | Calls remaining in the current minute (from LU response headers). `null` if no headers received yet. |
| `remainingWeek` | Calls remaining in the current week. `null` if unknown. |
| `currentDelayMs` | Adaptive delay applied before each request (0–2000ms). |
| `circuitState` | `closed` (normal), `open` (all requests rejected), `half-open` (testing one request). |
| `activeConcurrent` | Requests currently holding a semaphore slot (max 5). |
| `waitingInQueue` | Requests waiting for a semaphore slot. |
| `deduplicatedInFlight` | Unique GET URLs currently in-flight (consumers share the same promise). |

---

## Monitoring Recommendations

- **Uptime monitor:** Poll `GET /api/health` every 1–5 minutes. Alert on non-200 or `status !== "healthy"`.
- **Circuit breaker alert:** Check `pipeline.circuitState` in the health response. If `"open"`, the app is rejecting all LMS calls — investigate immediately.
- **Rate limit awareness:** If `pipeline.remainingMinute < 5`, the app is in heavy-pacing mode (2s delay per request). Consider deferring bulk operations.
- **Schema breaks:** Run `/api/health/internal-endpoints` manually after LearnUpon updates or when the schema break alert email fires.

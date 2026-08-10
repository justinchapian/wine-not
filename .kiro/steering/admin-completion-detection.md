---
inclusion: always
---
# Admin Completion Detection — LearnUpon Behavior & Implementation

> **Content type: Knowledge reference.** This file contains no auditable rules — it documents domain behavior and implementation details for admin completion detection. Consult during planning when working with enrollment status, admin completion logic, or engagement attribution.

## Overview

This documents how we detect whether a course enrollment was completed by an admin (bulk action) vs. organically by the learner. LearnUpon does not provide a single reliable API field for this, so we use a combination of the MarkComplete API and timestamp heuristics.

## LearnUpon Completion Mechanisms

There are multiple ways an enrollment can reach "completed" status in LearnUpon:

### 1. Organic completion (learner does the work)
- Learner progresses through course modules
- `percentage_complete` reaches 100
- `date_completed` is an irregular timestamp (e.g., `2023-08-08T20:03:51Z`)
- `date_enrolled` and `date_started` reflect the actual user interaction times

### 2. Admin Mark Complete (via LearnUpon UI, one-at-a-time)
- Admin uses the "Mark Complete" button in LearnUpon admin UI
- Creates a record in the `/markcompletes` API endpoint
- `date_completed` is set to a round hour (e.g., `T05:00:00Z`, `T15:00:00Z`)
- `date_started` is also set to the same round hour
- `date_enrolled` retains the original enrollment timestamp (not on the hour)
- `action_source_type` is `"UI"` in the markcomplete record
- `user_id` in the markcomplete record identifies which admin did it

### 3. Admin Mark Complete (via API)
- Same as #2 but `action_source_type` is `"API"` in the markcomplete record
- Used by our own app's "Mark Complete" and "Bulk Mark Complete" features

### 4. Bulk enrollment with pre-set completion (no markcomplete record)
- Admin uses LearnUpon's bulk enrollment feature and sets status to "completed" at creation time
- NO markcomplete API record is created — LearnUpon treats this differently internally
- All three dates (`date_enrolled`, `date_started`, `date_completed`) are on the exact hour
- Example: enrolled `2026-07-06T04:00:00Z`, started `2026-07-07T04:00:00Z`, completed `2026-07-07T04:00:00Z`
- `percentage_complete` is null (learner never accessed content)
- Hundreds of enrollments share the exact same timestamps

### 5. Bulk completion of existing enrollments (no markcomplete record)
- Admin bulk-completes enrollments that were previously bulk-enrolled
- `date_enrolled` and `date_started` are on the exact hour (from the original bulk enrollment)
- `date_completed` is on the exact hour (from the bulk completion, possibly years later)
- Example: enrolled `2021-12-30T00:00:00Z`, started `2021-12-31T00:00:00Z`, completed `2025-07-01T11:00:00Z`
- `percentage_complete` is null
- NO markcomplete API record

## LearnUpon Behavior Changes (Relaunch After Admin Completion)

### Old behavior (observed on 2023 markcomplete-via-UI completions)
- User relaunches a course that was admin-completed
- Status remains `completed`
- `date_completed` is NOT overwritten (keeps the admin-set round-hour timestamp)
- `percentage_complete` updates from null to a real value as user progresses (e.g., 100)
- The markcomplete API record still exists

### New behavior (observed on 2025+ bulk enrollments)
- User relaunches a course that was bulk-completed
- Status changes from `completed` to `in_progress`
- `date_completed` becomes null (or will be overwritten with organic timestamp on re-completion)
- `percentage_complete` tracks real progress (20%, 40%, 80%, etc.)
- If the user finishes, `date_completed` gets an organic (non-round-hour) timestamp

## Our Detection Logic

Located in: `src/lib/enrollment-utils.ts` (shared utility — used by `enrollments/search`, `learning-journeys/progress`, and `learning-journeys/bulk-progress` routes)

### Confirmed admin completion (`markedCompleteByAdmin: true`)

An enrollment is confirmed admin-completed if ANY of these are true:
1. The MarkComplete API (`GET /markcompletes?user_id=X`) returns a record matching this enrollment ID
2. Status is `completed` or `passed` AND both `date_enrolled` and `date_started` have minutes=0 and seconds=0 (UTC)

### Suspected admin completion (`suspectedAdminComplete: true`)

An enrollment is suspected admin-completed if:
- Both `date_enrolled` and `date_started` have minutes=0 and seconds=0 (UTC)
- BUT status is NOT `completed`/`passed` (typically `in_progress`)
- This means it was bulk-enrolled+completed, then the user relaunched the course

### Why this is bulletproof

The "enrolled AND started both on the exact hour" pattern is exclusively produced by bulk admin operations. Verified across 3,774 enrollments for a single course and thousands more across multiple users:
- Zero organic enrollments have both `date_enrolled` and `date_started` on the exact hour
- A real user enrollment always has seconds/minutes in the timestamp from server processing jitter
- Even if a user happened to complete at `:00:00` (1-in-3600 chance), their enrollment and start dates would NOT also be on the hour

### Why we don't check `date_completed` in the heuristic

We only need `date_enrolled` + `date_started` on the hour because:
- If status is still `completed`, the admin completion stands regardless of what `date_completed` shows
- If the user relaunched (old behavior), `date_completed` stays on the hour but we already caught it via the status check
- If the user relaunched (new behavior), `date_completed` becomes organic — but enrolled+started are still on the hour, which triggers `suspectedAdminComplete`

## API Details

### GET /markcompletes

- Endpoint: `GET /api/v1/markcompletes?user_id={userId}`
- Paginated (500 per page, uses `LU-Has-Next-Page` header)
- Returns records for enrollments that were explicitly marked complete via admin UI or API
- Does NOT return records for bulk enrollments created with pre-set completion status
- Key fields: `enrollment_id`, `user_id` (admin who did it, null if API), `action_source_type` ("UI" or "API"), `date_completed`, `status`

### Timestamp patterns observed

| Admin action type | date_enrolled | date_started | date_completed |
|---|---|---|---|
| Markcomplete via UI | Organic (not on hour) | On the hour | On the hour |
| Bulk enroll + complete | On the hour | On the hour | On the hour |
| Bulk complete later | On the hour | On the hour | On the hour (different day) |
| Relaunched (old LU) | Organic or on-hour | On the hour | On the hour (preserved) |
| Relaunched (new LU) | On the hour | On the hour | Organic or null |

### Common round hours seen in admin completions
- `T00:00:00Z` (midnight UTC)
- `T04:00:00Z` (midnight Eastern)
- `T05:00:00Z` (midnight Eastern during DST)
- `T11:00:00Z` (7am Eastern)
- `T15:00:00Z` (11am Eastern)

## UI Presentation

In the Course Enrollments table, the "Admin Completed" column shows:
- **Amber "Yes" badge** — confirmed admin completion (markcomplete record or completed+on-hour)
- **Orange "Suspected" badge** — bulk-enrolled on the hour but user relaunched (status is in_progress)
- **Dash (—)** — organic enrollment/completion

The column is sortable (Yes > Suspected > —) and included in CSV export.

## Files involved

- `src/types/learnupon.ts` — `AppEnrollment.markedCompleteByAdmin` and `AppEnrollment.suspectedAdminComplete` fields
- `src/lib/enrollment-utils.ts` — `detectAdminCompletion()`, `isAdminCompleted()`, `isTimestampOnTheHour()`, `buildCourseEnrollmentMaps()` (includes admin detection)
- `src/lib/learnupon-client.ts` — `getMarkCompletesByUserId()` function, `normalizeEnrollment()` defaults
- `src/app/api/enrollments/search/route.ts` — calls `detectAdminCompletion()` for each enrollment
- `src/components/enrollment-table.tsx` — UI display, sort logic, CSV export

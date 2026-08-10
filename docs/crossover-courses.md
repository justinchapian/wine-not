---
title: Crossover Courses
intent: knowledge-article
area: docs
---
# Crossover Courses — Full Understanding

> **Content type: Knowledge article.** This documents the crossover course identification model — domain knowledge for planning. Display rules and engagement attribution decisions are authoritatively recorded in `decisions.md`. Consult this article when working with crossover detection, engagement attribution, or multi-journey progress.

## What Are Crossover Courses?

A **crossover course** is a course that appears in a user's learning journey but was not enrolled by that journey. Instead, another learning journey (or, rarely, an admin action) created the enrollment first. When this journey reaches that course in its sequential progression, it finds the user already enrolled — it reuses the existing enrollment rather than creating a new one.

Crossover courses **count toward overall journey completion** (they're real completions), but they should be **excluded from engagement metrics** because the work wasn't driven by this journey.

---

## Why Crossover Courses Exist

EBLI's LearnUpon setup uses multiple learning journeys that share some courses. For example:

- Journey A: `[Course 1, Course 2, Course 3, Course 4, Course 5]`
- Journey B: `[Course 6, Course 3, Course 7, Course 8]`

If a user is enrolled in both journeys and completes Course 3 through Journey A first, when Journey B later reaches Course 3 in its sequence, it finds the user already enrolled and completed. That completion is a crossover from Journey A's perspective when viewed in Journey B (and vice versa depending on timing).

---

## Three Principles of Crossover Identification

### Principle 1: Shared Course Set (Static, Cacheable)

A course can **only** be a crossover if it appears in at least two learning journeys. If a course is unique to one journey, it is structurally impossible for it to be a crossover — no other journey could have created that enrollment.

**Implementation:** Cross-reference all cached journey course trees to produce:
```
Map<courseId, Set<journeyId>>  →  courses appearing in 2+ journeys = crossover candidates
```

This is a hard constraint (not heuristic), costs zero API calls (uses cached course trees), and eliminates false positives. Any course NOT in this set is guaranteed non-crossover regardless of enrollment patterns.

### Principle 2: Enrollment ID Monotonicity (Per-User, Per-Journey)

LearnUpon enrollment IDs are globally auto-incrementing integers. Journeys enroll courses sequentially — when a user is enrolled in a journey, the system immediately enrolls them in course 1, then progressively enrolls in course 2 after course 1 is completed, and so on.

This means: **within a single journey, enrollment IDs for courses at positions 1, 2, 3... must be monotonically increasing** (each successive course was enrolled at a later point in time, receiving a higher auto-increment ID).

If a course at position P has an enrollment ID that is **lower** than the enrollment ID at an earlier position, that enrollment was created before this journey reached that position. Another journey (or admin action) created it first — confirming it as a crossover.

LearnUpon does not create duplicate enrollments. When Journey B reaches a course the user is already enrolled in (from Journey A), it reuses the existing enrollment. The enrollment ID remains frozen at whatever value was assigned when first created.

### Principle 3: `dateEnrolled` Timestamp Correlation

The `dateEnrolled` field provides the same reasoning as enrollment IDs in a more human-readable form:

- If `dateEnrolled` for a course at position 5 is **earlier** than `dateEnrolled` for position 1, that course was enrolled before this journey existed for this user → crossover.
- The journey's own `enrolledAt` timestamp (from states.json) establishes when the user joined the journey. Any course with `dateEnrolled` significantly earlier than the journey's `enrolledAt` is a pre-existing enrollment.

Both enrollment IDs and timestamps are available in data we already fetch. Enrollment IDs are algorithmically cleaner (guaranteed monotonic, no timezone issues); timestamps are better for display and debugging.

---

## Source Journey Attribution

Once a crossover is identified, we can determine which journey created the enrollment:

1. Identify the crossover course's `dateEnrolled` timestamp (or enrollment ID)
2. Look at which other journeys contain this course (from the shared course map)
3. For each candidate source journey, check if the user is enrolled in it
4. Compare timing: the journey whose sequential enrollment progression contains the crossover course's `dateEnrolled` in the expected monotonic window is the source

This requires no new API calls when multi-journey progress data is already loaded (which it is on the user's enrollments page and in bulk reports).

---

## Current Implementation (Positional Approach)

Located in `src/lib/journey-engagement.ts` → `computeAttribution()`:

```
isCrossover = position > currentPosition AND status !== "not_enrolled"
```

This catches courses the user hasn't reached yet in this journey but is already enrolled in (or has completed) from another source. It's effective for courses **beyond** the current position but cannot identify crossovers within the already-progressed range (positions 1 through currentPosition).

### What the positional approach catches
- Courses ahead of the user's current position that show enrollment/completion from another journey
- Distinguishes true crossovers from simple "future courses" (not_enrolled)

### What it cannot see
- Courses within positions 1..currentPosition that were actually pre-existing enrollments from another journey (the journey reached them, reused the enrollment, and progressed past)
- Which specific journey created a crossover enrollment

---

## Enhanced Model (Implemented August 2026)

### Layer 1: Shared Course Map
- Derived from cached course trees via `getSharedCourseIds()` in `src/lib/platform-cache.ts`
- Returns `Set<number>` of course IDs appearing in 2+ journeys
- Cached alongside journey data, invalidated when journey cache is invalidated
- Provides hard constraint: only shared courses can be crossover

### Layer 2: Enrollment ID Sequence Analysis
- For each journey a user is in, walks course positions and collects enrollment IDs from anchor points (non-shared courses establish the monotonic baseline)
- Identifies monotonicity breaks → shared course whose enrollment ID is lower than an earlier non-shared course's ID
- Confirms crossover when enrollment `dateEnrolled` predates the journey's `enrolledAt`
- Pure function, no new API calls (enrollment data already fetched)

### Layer 3: Next-Enrollment Tiebreaker
- When a shared course breaks monotonicity but dates are ambiguous (enrolled after journey start), checks if the course's `dateCompleted` correlates with the next course's `dateEnrolled` within 5 minutes
- If correlation exists → this journey drove the completion (sequential auto-enrollment triggered it) → not crossover
- If no correlation → marks as `"possible"` (ambiguous)

### Three-Value Classification (`CrossoverStatus`)
```typescript
export type CrossoverStatus = false | true | "possible";
```

| Value | Meaning | Timeline | Engagement | UI Display |
|-------|---------|----------|------------|------------|
| `false` | Journey-enrolled | Included | Included | — |
| `true` | Confirmed crossover | Excluded | Excluded | "↗ Yes" (gray) |
| `"possible"` | Ambiguous | Included | Included | "↗ Possible" (amber) |

### Layer 4: Source Attribution (Deferred)
- Cross-reference crossover enrollment timing against other journey progressions
- Answers "this course was enrolled by Journey X"
- Data already available when viewing multi-journey progress
- Not yet implemented — documented for future enhancement if needed

---

## Edge Cases

| Case | Handling |
|------|----------|
| Admin bulk-enrolled course | Detected by existing admin-complete heuristic (dateEnrolled + dateStarted both on exact hour). Exclude from enrollment ID sequence analysis. Mark as "admin-enrolled" rather than journey-attributed |
| Two journeys reach same course simultaneously | The first journey's progression creates the enrollment; the second reuses it. Attribution goes to whichever journey progressed first |
| Completed journey (all positions reached) | Current positional approach sees no crossovers. Enhanced approach (enrollment ID sequencing) CAN detect pre-existing enrollments even within completed range |
| Course at position 1 pre-existing from another journey | Journey `enrolledAt` from states.json establishes the baseline. If course 1's `dateEnrolled` is earlier than journey `enrolledAt`, it was pre-existing. The journey reused it on enrollment |
| Standalone enrollments (not from any journey) | Not applicable at EBLI — all courses are enrolled through learning journeys. May appear for test users or admin accounts. Would show as enrollment ID breaks without any source journey match |

---

## Display Rules

- **Completion timelines (dot timeline, history modal):** Confirmed crossovers (`isCrossover === true`) are **excluded**. Possible crossovers (`"possible"`) are **included** (benefit of the doubt — don't strip data on uncertainty). Engagement is measured by work done through this journey.
- **Journey detail modal (course status table):** All crossover states shown distinctly:
  - Confirmed: muted gray styling, "↗ Yes" in gray, "Completed via another journey" tooltip
  - Possible: slightly muted styling, "↗ Possible" in amber, "Possibly completed via another journey" tooltip
  - Neither: normal styling, dash in crossover column
- **Excel reports:** Crossover column: "Yes" for confirmed, "Possible" for ambiguous, blank for non-crossover.
- **Engagement badge:** Computed from journey-enrolled completion dates only — confirmed crossover completions excluded, possible crossover completions included.

---

## Relationship to Other Features

- **Smart pre-filter (`journey-pre-filter.ts`):** Uses first-course enrollment to skip non-enrolled journeys. Unrelated to crossover identification but uses the same enrollment data.
- **Admin completion detection:** Identifies admin-completed enrollments which should be excluded from enrollment ID sequence analysis (their enrollment IDs may not follow natural journey sequencing).
- **Engagement attribution:** Consumes crossover classification to exclude crossover completions from engagement level computation.
- **Bulk progress reports:** Already have multi-journey data per user — ideal context for source attribution when implemented.

---

## Key Files

| File | Role |
|------|------|
| `src/lib/journey-engagement.ts` | Core attribution logic (`computeAttribution()`, `CrossoverStatus` type, `EnhancedAttributionOptions`) — pure function |
| `src/lib/platform-cache.ts` | Cached journey course trees + `getSharedCourseIds()` derived cache |
| `src/app/api/learning-journeys/progress/route.ts` | Wires enhanced attribution into API response, builds enrollment ID/date maps |
| `src/app/api/learning-journeys/bulk-progress/route.ts` | Same as progress route but for bulk multi-user reports |
| `src/components/journey-detail-modal.tsx` | Displays crossover column: "↗ Yes" (gray), "↗ Possible" (amber) |
| `src/components/journey-timeline.tsx` | Excludes confirmed crossovers (`=== true`) from 6-month timeline dots |
| `src/components/journey-history-modal.tsx` | Excludes confirmed crossovers from full history timeline |
| `src/components/download-progress-report-dialog.tsx` | Single-user Excel: Crossover column shows Yes/Possible/blank |
| `src/components/bulk-progress-report-dialog.tsx` | Bulk Excel: Crossover column shows Yes/Possible/blank |
| `src/lib/__tests__/journey-engagement.test.ts` | Property-based tests + enhanced detection unit tests |
| `src/lib/__tests__/crossover-classification-fix.test.ts` | Bug regression tests for crossover classification |

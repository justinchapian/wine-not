---
inclusion: fileMatch
fileMatchPattern: ["src/app/**/*.ts", "src/app/**/*.tsx", "src/components/**/*.tsx", "src/hooks/**/*.ts", "src/lib/**/*.ts", "src/types/**/*.ts"]
---
# Code Structure & Sustainability Rules

## Purpose

This document establishes structural conventions that prevent code duplication and maintainability drift as the app grows. These are enforced by ESLint where possible and by this steering file where automation isn't feasible.

---

## Rules

### RULE: Import shared types — never define inline
- **Trigger:** Adding or modifying code that uses `JourneyProgress`, `CourseStatus`, `AppUser`, `AppEnrollment`, or any type listed in the Shared types table below
- **Check:** Is the type imported from `@/types/` or is it defined inline in the component/route?
- **Instruction:** Import from the authoritative location (see table below). Never define these types inline. If the shared type needs a new field, add it to the source type — don't shadow it locally.

### RULE: Check `src/lib/` before writing helper logic inline
- **Trigger:** Writing a utility/helper function inside a route handler or component
- **Check:** Does equivalent logic already exist in `src/lib/`? (See Shared utilities table below.)
- **Instruction:** Import and use the shared utility. If it doesn't exist yet but a second route/component would need the same logic, extract to a shared module immediately. Exception: truly one-off transformations with no reuse potential can stay inline.

### RULE: Use shared hooks before writing fetch/sort/mutation boilerplate
- **Trigger:** Adding a new component that fetches data on mount, manages mutation state, or implements sortable tables
- **Check:** Would `useApiQuery`, `useMutation`, or `useSortable` cover this use case?
- **Instruction:** Use the appropriate hook. Don't write manual `useState` + `useEffect` + `fetch` patterns. For tables with sorting, use `useSortable` instead of duplicating handleSort/SortHeader. See hooks table below for the full list.

### RULE: Extract components at threshold
- **Trigger:** A component file is growing or you're adding significant new UI logic
- **Check:** Does ANY of these apply? (1) File exceeds ~300 lines. (2) The piece is logically self-contained and could be reused or tested independently. (3) It has its own state management distinct from the page. (4) It represents a clear "section" of a page.
- **Instruction:** Extract to a separate file. Exception: small page-specific helper components (<20 lines) that are tightly coupled to the parent's state and used only once can stay inline. Extract immediately when they grow or gain independent state.

### RULE: Page files are orchestrators — no inline rendering for complex sections
- **Trigger:** Adding rendering logic to a page file (`src/app/**/page.tsx`)
- **Check:** Is the page directly rendering a complex table, multi-step dialog, or section with its own state?
- **Instruction:** Pages should fetch shared data, manage top-level state, and compose section components. Complex tables, dialogs, and stateful sections belong in extracted component files.

### RULE: Start API routes with `withAuthAndRole()` + `isErrorResponse` check
- **Trigger:** Adding or modifying an API route handler
- **Check:** Is `const auth = await withAuthAndRole("ROLE")` followed by an `isErrorResponse(auth)` check the first lines?
- **Instruction:** It must be. See `api-conventions.md` RULE: "Every API route starts with withAuthAndRole()" for exceptions.

### RULE: Add `maxDuration = 300` for paginating/fan-out routes
- **Trigger:** Adding or modifying an API route
- **Check:** Does the route paginate, fan out, or call `parallelWithLimit`?
- **Instruction:** Add `export const maxDuration = 300` at route file level. See `api-conventions.md` RULE: "Add maxDuration = 300" for the full trigger list.

### RULE: Use course tree fetching via shared callback
- **Trigger:** Adding code that needs a journey's course tree
- **Check:** Is the code calling LearnUpon directly for course tree data instead of using the cache pattern?
- **Instruction:** Use `getCourseTree(id, versionId, () => fetchCourseTreeFromLMS(id, versionId, session.cookie))` from `src/lib/workflow-parser.ts`. This leverages the platform cache.

### RULE: Use `buildCourseEnrollmentMaps()` for enrollment-to-course mapping
- **Trigger:** Building a map of courseId → enrollment status or courseId → enrollment data
- **Check:** Is the code manually iterating enrollments to build a lookup map?
- **Instruction:** Use `buildCourseEnrollmentMaps(enrollments, markedIds)` from `src/lib/enrollment-utils.ts`. It handles admin completion detection and status priority automatically.

### RULE: Add tests when modifying shared utilities with existing test files
- **Trigger:** Adding or modifying exported logic in a file under `src/lib/` that has a corresponding test file in `src/lib/__tests__/`
- **Check:** Does the change add a new code path, parameter, error condition, or behavioral branch?
- **Instruction:** If yes, add a test case covering the new behavior in the existing test file. If no test file exists and the function is non-trivial (has branching, state, or time-dependent logic), create one following the conventions in the Testing section below.

---

## Knowledge: ESLint enforcement (automated)

The following patterns are **lint errors** — they fail `npm run lint`:

1. **Raw `fetch()` in client code** — Use `apiFetch()` from `@/lib/fetch-client`. Enforced via `no-restricted-syntax` in `eslint.config.mjs`.
2. **Server-only imports in client code** — Importing `@/lib/learnupon-client`, `@/lib/learnupon-session`, `@/lib/request-pipeline`, `@/lib/platform-cache`, or `@/lib/get-lms-session` in components/pages is a lint error.
3. **File size** — Files exceeding 400 lines (excluding blanks/comments) produce a lint warning. This catches growing monoliths early.

## Knowledge: Shared types

| Type | Location | Used by |
|------|----------|---------|
| `JourneyProgress`, `JourneyProgressCourse`, `JourneyProgressMeta` | `src/types/journey.ts` | API routes, all journey-related components |
| `AppUser`, `AppEnrollment`, `AppGroupMembership` | `src/types/learnupon.ts` | API routes, components |
| `CachedUser`, `UserSortField`, `SortDir` | `src/types/user.ts` | Users page, UserTable component |

## Knowledge: Shared server-side utilities

| Utility | Location | What it does |
|---------|----------|--------------|
| `fetchCourseTreeFromLMS()` | `src/lib/workflow-parser.ts` | Fetches + parses a journey's course tree. Pass as callback to `getCourseTree()`. |
| `walkWorkflowTree()` | `src/lib/workflow-parser.ts` | Extracts ordered course IDs from a workflow tree JSON structure. |
| `buildCourseEnrollmentMaps()` | `src/lib/enrollment-utils.ts` | Builds courseId→status and courseId→enrollmentData maps from a user's enrollments. Includes admin completion detection. |
| `detectAdminCompletion()` | `src/lib/enrollment-utils.ts` | Determines if a single enrollment was admin-completed (markcomplete API + heuristic). |
| `statusPriority()` | `src/lib/enrollment-utils.ts` | Returns numeric priority for enrollment status (for picking "best" when duplicates exist). |
| `handleSchemaBreakError()` | `src/lib/api-helpers.ts` | Logs, alerts, and returns 502 for SchemaBreakError catches. |
| `handleApiError()` | `src/lib/api-helpers.ts` | Converts LearnUponError/CircuitBreakerError/AbortError/unknown into appropriate NextResponse (includes 503 for circuit breaker). |

## Knowledge: Shared hooks

| Hook | Location | When to use |
|------|----------|-------------|
| `useMutation` | `src/hooks/use-mutation.ts` | Any POST/DELETE action with loading/error state |
| `useApiQuery` | `src/hooks/use-api-query.ts` | Fetch data on mount/dependency change with auto-abort |
| `useSortable` | `src/hooks/use-sortable.ts` | Any sortable table (provides sortField, sortDir, handleSort, compare) |
| `useRole` | `src/hooks/use-role.ts` | Check current user's RBAC role |

**All existing components now use these hooks** — `add-to-journey-dialog`, `add-to-group-dialog`, and `bulk-progress-report-dialog` use `useApiQuery` for on-mount fetching; `enrollment-table` and `page.tsx` (Users) use `useSortable` for sort state. No manual fetch-on-mount or sort boilerplate remains in the codebase.

### `useApiQuery` — dialog pattern

For dialogs that fetch data when opened:
```tsx
const { data, loading, error } = useApiQuery<T>(url, {
  enabled: isOpen,
  select: (json) => json.items || [],
  onSuccess: (data) => { /* pre-select, etc. */ },
});

// Separate useEffect for resetting UI state on open
useEffect(() => {
  if (isOpen) { setSelection(null); setSearch(""); }
}, [isOpen]);
```

### `useSortable` — table pattern

For sortable tables with field-specific value extraction:
```tsx
const { sortField, sortDir, handleSort } = useSortable<MySortField>({
  defaultField: "name",
  defaultDir: "asc",
});

// Use handleSort in SortHeader onClick; keep custom field extraction in sort comparator
```

The hook's generic `compare()` works for simple cases. For complex extraction (derived values, multi-field priority), use `sortField` and `sortDir` from the hook in your own `useMemo` comparator.

## Knowledge: Testing conventions

### What must be tested

| Module type | Test required? | Approach |
|-------------|---------------|----------|
| Pure utility functions (`src/lib/`) with non-trivial logic | Yes | Unit + property-based (fast-check) where inputs vary widely |
| Infrastructure modules (pipeline, cache, guards, fetch-client) | Yes | Unit tests with mocked `fetch`, fake timers for time-dependent behavior |
| API route handlers that only call utilities and return | No | The underlying utilities are tested |
| React components | No (at this scale) | — |

### Test file conventions

- Location: `src/lib/__tests__/<module-name>.test.ts`
- Framework: vitest (`describe`/`it`/`expect`)
- Property-based: `fast-check` for functions with wide input domains
- Mocking: `vi.mock()` for external deps, `vi.useFakeTimers()` for time
- No watch mode: `vitest --run` (single execution)
- Reset singleton state in `beforeEach` (caches, circuit breakers, etc.)

---

## Verifying your work

After adding or modifying components/utilities:

- **Build:** `npm run build` must pass (catches type errors, missing imports, server/client boundary violations)
- **Lint:** `npm run lint` must pass (catches raw fetch in client code, server-only imports, file size warnings)
- **Type check for shared types:** If you added a field to a type in `@/types/`, verify all consumers compile (the build will catch this, but check that the field is actually used where intended)
- **Hook behavior:** If you modified a shared hook, verify that existing components using it still behave correctly (check the "Used by" column in tables above)

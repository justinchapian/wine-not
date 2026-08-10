---
inclusion: fileMatch
fileMatchPattern: ["src/**"]
---
# Code Structure & Sustainability Rules

## Purpose

This document establishes structural conventions that prevent code duplication and maintainability drift as the app grows. These are enforced by ESLint where possible and by this steering file where automation isn't feasible.

---

## Rules

### RULE: Import shared types — never define inline
- **Trigger:** Adding or modifying code that uses a shared type
- **Check:** Is the type imported from the authoritative location or defined inline?
- **Instruction:** Import from the authoritative location. Never define shared types inline. If the shared type needs a new field, add it to the source type — don't shadow it locally.

### RULE: Check `src/lib/` before writing helper logic inline
- **Trigger:** Writing a utility/helper function inside a route handler or component
- **Check:** Does equivalent logic already exist in `src/lib/`?
- **Instruction:** Import and use the shared utility. If it doesn't exist yet but a second route/component would need the same logic, extract to a shared module immediately. Exception: truly one-off transformations with no reuse potential can stay inline.

### RULE: Use shared hooks before writing fetch/sort/mutation boilerplate
- **Trigger:** Adding a new component that fetches data on mount, manages mutation state, or implements sortable tables
- **Check:** Would an existing hook cover this use case?
- **Instruction:** Use the appropriate hook. Don't write manual `useState` + `useEffect` + `fetch` patterns.

### RULE: Extract components at threshold
- **Trigger:** A component file is growing or you're adding significant new UI logic
- **Check:** Does ANY of these apply? (1) File exceeds ~300 lines. (2) The piece is logically self-contained and could be reused or tested independently. (3) It has its own state management distinct from the page. (4) It represents a clear "section" of a page.
- **Instruction:** Extract to a separate file. Exception: small page-specific helper components (<20 lines) that are tightly coupled to the parent's state and used only once can stay inline.

### RULE: Page files are orchestrators — no inline rendering for complex sections
- **Trigger:** Adding rendering logic to a page file (`src/app/**/page.tsx`)
- **Check:** Is the page directly rendering a complex table, multi-step dialog, or section with its own state?
- **Instruction:** Pages should fetch shared data, manage top-level state, and compose section components. Complex sections belong in extracted component files.

### RULE: Add tests when modifying shared utilities with existing test files
- **Trigger:** Adding or modifying exported logic in a file under `src/lib/` that has a corresponding test file
- **Check:** Does the change add a new code path, parameter, error condition, or behavioral branch?
- **Instruction:** If yes, add a test case covering the new behavior in the existing test file.

---

## Knowledge: Shared types

_To be filled in as the app takes shape._

## Knowledge: Shared utilities

_To be filled in as the app takes shape._

## Knowledge: Shared hooks

_To be filled in as the app takes shape._

## Knowledge: Testing conventions

- Location: `src/lib/__tests__/<module-name>.test.ts`
- Framework: vitest (`describe`/`it`/`expect`)
- No watch mode: `vitest --run` (single execution)

---

## Verifying your work

After adding or modifying components/utilities:

- **Build:** `npm run build` must pass
- **Lint:** `npm run lint` must pass

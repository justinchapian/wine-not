---
inclusion: always
---
# Tech Stack & Architecture

---

## Rules

### RULE: Use `Button` with `variant` prop — never raw `<button>`
- **Trigger:** Adding or modifying UI that includes buttons
- **Check:** Does the code use raw `<button>` elements with inline Tailwind classes?
- **Instruction:** Use the `Button` component from the UI library with the appropriate `variant` prop.

### RULE: Don't weaken or remove security headers without explicit user approval
- **Trigger:** Modifying framework config (e.g., `next.config.ts`)
- **Check:** Does the change remove, weaken, or bypass any security header?
- **Instruction:** Preserve all security headers. If a change requires weakening CSP or removing a header, stop and get explicit user confirmation first.

### RULE: External dependencies must be behind interfaces
- **Trigger:** Adding code that calls an external service (AI provider, wine database, image recognition, etc.)
- **Check:** Is the code calling the external service directly from domain logic or route handlers?
- **Instruction:** All external dependencies must be accessed through a replaceable interface/abstraction. Domain logic should depend on the interface, not the implementation. This enables provider changes without rewriting business logic. See `docs/ai-integration-principles.md` for AI-specific guidance.

### RULE: Separate data categories — never collapse distinct concepts into one record
- **Trigger:** Adding or modifying data models/schemas
- **Check:** Does the change conflate any of these: canonical beverage data, user experience, AI observation, user preference, hypothesis, or recommendation?
- **Instruction:** Keep these as distinct entities (see `docs/data-model-principles.md`). A recommendation is not a preference. An AI observation is not a user statement. A hypothesis is not a known fact. The data model must preserve these boundaries.

---

## Knowledge: Stack

**Decided:**
- Mobile-first PWA
- Tailwind CSS for styling
- TypeScript

**Not yet decided (see `decisions.md`):**
- Frontend framework
- Backend framework / full-stack approach
- Database
- AI provider(s)
- Hosting platform (Vercel is easy but not committed)
- Auth mechanism

## Knowledge: Architectural principles (from Vision §35)

### Modular monolith over microservices

Start with cleanly separated concerns within a single deployable unit. Avoid distributing across multiple services prematurely.

Clean separation means:
- Domain logic (palate model, discovery categories, recommendations)
- AI integration (behind provider interfaces)
- External API integrations (wine data, image recognition — behind provider interfaces)
- Database access (repository pattern or equivalent)
- Authentication
- Frontend

These are logical boundaries, not deployment boundaries.

### Provider interfaces

External dependencies must have clear interfaces that can be swapped:

- `AIProvider` (or more granular — see open question in `decisions.md`)
- `WineDataProvider`
- `ImageRecognitionProvider`

### Backend independence

The backend/API should remain independent enough that a native client could be added later. Even if deployed as a full-stack app initially, the API surface should be clean and documented.

## Knowledge: Platform requirements (from Vision §30)

The PWA must:
- Work well on mobile (primary interaction device is Android phone)
- Support camera/photo input
- Be installable on the home screen
- Feel app-like (transitions, touch-friendly, no "website" feel)
- Have fast, touch-friendly interactions

Native Android is deferred unless actual usage demonstrates necessity.

## Knowledge: Dev setup

_To be filled in once the project is scaffolded._

## Knowledge: File structure

_To be filled in once the project is scaffolded._

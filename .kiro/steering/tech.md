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
- **Trigger:** Modifying `next.config.ts`
- **Check:** Does the change remove, weaken, or bypass any security header?
- **Instruction:** Preserve all security headers. If a change requires weakening CSP or removing a header, stop and get explicit user confirmation first.

---

## Knowledge: Stack

_To be filled in once tech choices are decided._

## Knowledge: Architecture patterns

_To be filled in as the app takes shape._

## Knowledge: File structure

_To be filled in once project is scaffolded._

## Knowledge: Dev setup

_To be filled in once environment is configured._

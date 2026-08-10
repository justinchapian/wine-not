---
inclusion: always
---
# Steering Index

Read this first. It tells you which files to consult for any task. Rules live in individual steering files (look for `### RULE:` blocks). This file only routes you to the right place.

## Steering Files (`.kiro/steering/`)

| File | Inclusion | Summary |
|------|-----------|---------|
| `INDEX.md` | always | This file. Task-to-file routing. |
| `product.md` | always | What the app does, who uses it, constraints. |
| `tech.md` | always | Stack, architecture, file structure. |
| `code-structure.md` | fileMatch: `src/**` | Shared types/utilities/hooks, extraction rules. |
| `decisions.md` | always | Active design decisions + rejected approaches. |
| `feature-planning.md` | always | Doc governance: post-feature checklist, anti-drift gates. |
| `deployment.md` | always | Hosting, npm safety, pre-push checklist. |
| `safety.md` | always | Operational safety: plan before mutating, scope discipline. |

## Documentation (`docs/`)

| File | Intent | Summary |
|------|--------|---------|
| `AGENT_GUIDE.md` | Agent contract | How AI agents navigate docs, resolve conflicts, verify work. |
| `ROADMAP.md` | Planned work | Future features + tech debt. Items removed after shipping. |

## Root Files

| File | Purpose |
|------|---------|
| `README.md` | Human quick-start. |
| `AGENTS.md` | Agent-facing notes (framework version warnings, etc.). |

## Task → File Quick Reference

| Task | Read these files |
|------|-----------------|
| Add a new feature | `product.md` → `tech.md` → `code-structure.md` |
| Understand a design choice | `decisions.md` |
| Plan a new feature | `feature-planning.md` → `docs/ROADMAP.md` |
| Modify docs or steering | `feature-planning.md` (anti-drift gates) |
| Risky change (auth, data, paid APIs) | `safety.md` → relevant technical steering |
| Deploy or fix build | `deployment.md` |
| What's planned / tech debt | `docs/ROADMAP.md` |

## Contradiction Resolution

1. **Code** is authoritative over documentation
2. **Steering files** are authoritative over `docs/` files
3. Within steering: `decisions.md` records the "why" — if a rule conflicts with a recorded decision, the decision wins

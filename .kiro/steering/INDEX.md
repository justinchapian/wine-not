---
inclusion: always
---
# Steering Index

Read this first. It tells you which files to consult for any task. Rules live in individual steering files (look for `### RULE:` blocks). This file only routes you to the right place.

## Steering Files (`.kiro/steering/`)

| File | Inclusion | Summary |
|------|-----------|---------|
| `INDEX.md` | always | This file. Task-to-file routing. |
| `product.md` | always | What the app does, who uses it, core philosophy, guiding question. |
| `tech.md` | always | Stack, architectural principles, platform requirements. |
| `code-structure.md` | fileMatch: `src/**` | Shared types/utilities/hooks, extraction rules, testing. |
| `decisions.md` | always | Open architectural questions + future design decisions + rejected approaches. |
| `feature-planning.md` | always | Doc governance: post-feature checklist, anti-drift gates, rule format spec. |
| `deployment.md` | always | Hosting, npm safety, pre-push checklist. |
| `safety.md` | always | Operational safety: plan before mutating, scope discipline. |

## Documentation (`docs/`)

| File | Intent | Summary |
|------|--------|---------|
| `VISION.md` | Source of truth | Full 41-point product vision. The definitive reference for product intent. |
| `ROADMAP.md` | Planned work | 10-phase development progression. Items removed after shipping. |
| `data-model-principles.md` | Architecture | 6 data categories, evidence chain, canonical identity, multi-user privacy. |
| `ai-integration-principles.md` | Architecture | Provider independence, cost philosophy, capabilities, explainability, personality. |
| `AGENT_GUIDE.md` | Agent contract | How AI agents navigate docs, resolve conflicts, verify work. |

## Root Files

| File | Purpose |
|------|---------|
| `README.md` | Human quick-start. |
| `AGENTS.md` | Agent-facing notes (framework version warnings, etc.). |

## Task → File Quick Reference

| Task | Read these files |
|------|-----------------|
| Understand the product | `product.md` → `docs/VISION.md` |
| Add a new feature | `product.md` → `tech.md` → `code-structure.md` |
| Work on the data model / schema | `docs/data-model-principles.md` → `tech.md` |
| Work on AI integration | `docs/ai-integration-principles.md` → `tech.md` |
| Understand a design choice | `decisions.md` |
| Plan a new feature | `feature-planning.md` → `docs/ROADMAP.md` |
| Modify docs or steering | `feature-planning.md` (anti-drift gates) |
| Risky change (auth, data, paid APIs) | `safety.md` → relevant technical docs |
| Deploy or fix build | `deployment.md` |
| What's planned / tech debt | `docs/ROADMAP.md` |

## Contradiction Resolution

1. **Code** is authoritative over documentation (if code works correctly but docs disagree, docs are stale)
2. **Steering files** are authoritative over `docs/` files
3. Within steering: `decisions.md` records the "why" — if a rule conflicts with a recorded decision, the decision wins
4. `docs/VISION.md` is the source of truth for product philosophy — if steering contradicts the vision's principles, the vision wins

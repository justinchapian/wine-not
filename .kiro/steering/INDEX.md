---
inclusion: always
---
# Steering Index

Read this first. It tells you which files to consult for any task. Rules live in individual steering files (look for `### RULE:` blocks). This file only routes you to the right place.

## Steering Files (`.kiro/steering/`)

| File | Inclusion | Summary |
|------|-----------|---------|
| `INDEX.md` | always | This file. Task→file routing. |
| `product.md` | always | What the app does, who uses it, constraints. |
| `tech.md` | always | Stack, dual-auth, pipeline, caching, file structure. |
| `api-conventions.md` | fileMatch: `src/app/api/**`, `src/lib/**` | Rules for API routes, error handling, mutations, RBAC. |
| `code-structure.md` | fileMatch: `src/**` | Shared types/utilities/hooks, extraction rules, checklists. |
| `decisions.md` | always | Active design decisions + rejected approaches. |
| `feature-planning.md` | always | Doc governance: post-feature checklist, anti-drift gates, decision graduation. |
| `deployment.md` | always | Vercel, npm safety, pre-push checklist. |
| `admin-completion-detection.md` | always | Domain: admin vs. organic completion detection. |
| `safety.md` | always | Operational safety: plan before mutating, scope discipline. |
| `schema-break-runbook.md` | manual | Incident response for schema breaks. Load on demand. |

## Documentation (`docs/`)

| File | Intent | Summary |
|------|--------|---------|
| `AGENT_GUIDE.md` | Agent contract | How AI agents navigate docs, resolve conflicts, verify work. |
| `ROADMAP.md` | Planned work | Future features + tech debt. Items removed after shipping. |
| `crossover-courses.md` | Knowledge article | Crossover identification model (3 principles + classification). |
| `caching.md` | Architecture ref | Platform cache: what's cached, TTL, invalidation, extension. |
| `health-endpoints.md` | Architecture ref | Health & monitoring endpoints: connectivity, schema validation, pipeline status. |
| `LearnUpon_API_Documentation.md` | External API ref | LearnUpon public API docs. |

## Root Files

| File | Purpose |
|------|---------|
| `README.md` | Human quick-start. Points to steering. |
| `AGENTS.md` | Next.js 16 version warning (breaking changes vs. training data). |

## Task → File Quick Reference

| Task | Read these files |
|------|-----------------|
| Add a new API route | `api-conventions.md` → `code-structure.md` |
| Add a new UI component | `code-structure.md` → `tech.md` (button variants, hooks) |
| Fix a bug in LMS calls | `tech.md` (pipeline) → `api-conventions.md` |
| Understand a design choice | `decisions.md` |
| Plan a new feature | `feature-planning.md` → `docs/ROADMAP.md` |
| Modify docs or steering | `feature-planning.md` (anti-drift gates) |
| Risky change (mutations, pipeline, auth) | `safety.md` → relevant technical steering |
| Modify caching | `docs/caching.md` → `tech.md` |
| Deploy or fix build | `deployment.md` |
| Crossover/engagement logic | `admin-completion-detection.md` → `docs/crossover-courses.md` |
| Schema break alert | `schema-break-runbook.md` (load manually) |
| What's planned / tech debt | `docs/ROADMAP.md` |

## Contradiction Resolution

1. **Code** is authoritative over documentation (if code works correctly but docs disagree, docs are stale)
2. **Steering files** are authoritative over `docs/` files
3. Within steering: `decisions.md` records the "why" — if a rule conflicts with a recorded decision, the decision wins

## Structure Rules

- This file must stay under ~50 lines of content (excluding tables). Detail belongs in `AGENT_GUIDE.md`.
- See `feature-planning.md` for anti-drift rules, decision graduation, and doc update triggers.

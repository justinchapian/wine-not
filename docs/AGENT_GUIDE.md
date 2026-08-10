---
title: Agent Guide
intent: agent-contract
area: docs
---
# Agent Guide — How AI Agents Should Navigate This Codebase

> **Content type: Procedural guide.** This file routes you to authoritative rules — it does not define them. Rules live in `.kiro/steering/` files (look for `### RULE:` blocks). If this guide conflicts with a steering file, the steering file wins.

## Start Here

1. **Read `.kiro/steering/INDEX.md`** — it has a task→file table. Find your 2-3 relevant files.
2. Read those files. Don't read all steering files for every task.
3. Follow the `### RULE:` blocks in steering files. They are requirements, not suggestions.

## Document Hierarchy (what to trust)

```
Code (actual behavior)
  ↓ overrides
Steering files (.kiro/steering/*.md)
  ↓ overrides
Docs (docs/*.md)
  ↓ overrides
README.md / AGENTS.md
```

If code contradicts steering, the code is likely correct and the doc is stale — flag it to the user.

## Content Types in This Codebase

| Prefix | Meaning | When to read | When to audit |
|--------|---------|--------------|---------------|
| `### RULE:` | Enforceable constraint (Trigger/Check/Instruction) | Planning + Audit | Yes — check each against diff |
| `## Knowledge:` | Context for planning, not checkable | Planning only | No |
| `## Rules` | Section header containing RULE blocks | Both | Yes |

**Audit pass:** After implementation, scan `### RULE:` blocks in relevant steering files. Each rule's Check question must have a verifiable answer in your diff.

## File Types

### Steering (`.kiro/steering/`) — Rules you MUST follow

Active rules governing how code is written. Treat `### RULE:` blocks as requirements. Don't propose alternatives to documented decisions unless the user explicitly asks.

### Docs (`docs/`) — Context you should understand

- `ROADMAP.md` — What to build next. Items removed after shipping.
- `crossover-courses.md` — Deep domain knowledge article.
- `caching.md` — Cache architecture reference.
- `health-endpoints.md` — Health & monitoring endpoint reference.
- `LearnUpon_API_Documentation.md` — External API reference.

### Root files

- `AGENTS.md` — Next.js 16 has breaking changes. Check `node_modules/next/dist/docs/` for current APIs.
- `README.md` — Human quick-start only.

## Common Task Workflows

### Adding a new API route

1. Read `api-conventions.md` (all `### RULE:` blocks) → `code-structure.md`
2. Follow rules: `withAuthAndRole()` first, LMS wrappers, `maxDuration`, shared utilities, error handling, audit logging
3. **Verify:** `npm run build` + `npm run lint` pass
4. **After:** Run post-feature rule from `feature-planning.md`

### Adding a new UI component

1. Read `code-structure.md` (all `### RULE:` blocks) → `tech.md` (Button rule)
2. Follow rules: `apiFetch()`, shared hooks, shared types, extraction threshold, Button with variant
3. **Verify:** `npm run build` + `npm run lint` pass
4. **After:** Run post-feature rule from `feature-planning.md`

### Fixing a bug

1. Check `INDEX.md` task→file table for relevant steering
2. Check `decisions.md` for design rationale before changing behavior
3. Check `docs/ROADMAP.md` tech debt section — your bug may already be documented
4. **After:** Remove from ROADMAP if it was listed there

### Planning a new feature

1. Read `feature-planning.md` for the meta-rules
2. Check `docs/ROADMAP.md` for existing plans
3. Plan must end with steering/doc update steps (enforced by `feature-planning.md` RULE: "Post-feature")

### Responding to a schema break alert

1. Load `schema-break-runbook.md` (manual inclusion — read explicitly)
2. Follow numbered steps
3. Key files: `src/lib/lu-internal-paths.ts` (paths), `src/lib/lu-response-guards.ts` (validators)

### Risky changes (mutations, pipeline, auth, cache)

1. Read `safety.md` RULE: "Explain before mutating"
2. Present plan with files, data impact, failure mode, blast radius
3. Wait for user confirmation before implementing

## Key Patterns (quick reference)

These are summarized from steering rules. The authoritative source is the `### RULE:` block in the referenced file.

| Pattern | Source | Exception |
|---------|--------|-----------|
| No raw `fetch()` for LMS calls | `api-conventions.md` | Non-LMS services, `/api/health` |
| No raw `fetch()` in client code | `api-conventions.md` (ESLint enforced) | None |
| Never read `process.env.LMS_*` | `api-conventions.md` | None |
| `withAuthAndRole()` first in routes | `api-conventions.md` | `/api/auth/[...nextauth]`, `/api/health`, `/api/csp-report` |
| Types from `@/types/` not inline | `code-structure.md` | Truly local types used in one file |
| `Button` with `variant` prop | `tech.md` | None |
| Internal paths from `lu-internal-paths.ts` | `api-conventions.md` | None |
| Explain before mutating | `safety.md` | Read-only changes, pure refactors |
| Scope discipline | `safety.md` | Explicitly requested changes |

## Verifying Your Work

Every steering file with rules has a "Verifying your work" section at the bottom. Follow it. Minimum for any change:

1. `npm run build` passes
2. `npm run lint` passes
3. If you modified shared utilities, confirm consumers still compile
4. **If you modified docs or steering:** Run the anti-drift checks from `feature-planning.md` (no duplicated facts, no restated rules, no deliverable text in ROADMAP)

---
inclusion: always
---
# Steering & Documentation Governance

## Scope

This file governs **all** changes to `.kiro/steering/` and `docs/` — whether triggered by a new feature, a standalone documentation task, a bug fix, or any other work. The rules below are not feature-specific.

---

## Rule Format Specification

All rules in steering files MUST follow this format:

```markdown
### RULE: [Short imperative name]
- **Trigger:** [When does this rule fire?]
- **Check:** [What specific question must be answered?]
- **Instruction:** [What to do based on the answer?]
```

If a rule lacks any of these three components, it is a philosophical statement — not an enforceable constraint. Philosophical statements do not reliably prevent violations. They may serve as tiebreakers when facing ambiguity, but they should never be the sole defense against a behavior you need to prevent.

**Test:** If you cannot describe *the moment* the agent should stop and check this rule, the rule will not function as a gate.

---

## Rules

### RULE: Post-feature — update steering and docs after every feature
- **Trigger:** A feature, bug fix, or significant change has been completed (plan/spec final steps)
- **Check:** Do any steering files or docs need to be updated to reflect the new work? Specifically: does the change introduce new patterns, dependencies, API routes, auth flows, caching logic, architectural decisions, or user-facing capabilities?
- **Instruction:** Analyze all files in `.kiro/steering/` and `docs/` using the routing table below. Update relevant files so future sessions have accurate context. When adding new content: if it's an enforceable constraint, format as `### RULE:` with Trigger/Check/Instruction. If it's context/architecture/rationale, use `## Knowledge:` prefix. See Rule Format Specification at the top of this file. If no updates are needed, explicitly state "no doc updates needed" with brief justification. If a ROADMAP item shipped, remove it.

| Steering file | Update when the feature... |
|---|---|
| `INDEX.md` | Creates or deletes a steering file or `docs/` file |
| `product.md` | Adds a new user-facing capability, changes RBAC, or adds external dependencies |
| `tech.md` | Introduces new libraries, API patterns, caching strategies, or architecture changes |
| `api-conventions.md` | Adds new API routes or changes request/response patterns |
| `code-structure.md` | Adds new shared types, utilities, hooks, or changes structural conventions |
| `decisions.md` | Requires a non-obvious design decision worth recording (see graduation rule below) |
| `deployment.md` | Affects deployment, environment variables, or infrastructure |
| `schema-break-runbook.md` | Changes data shapes that could break existing cached data |

### RULE: Anti-drift — don't restate facts that already have a home
- **Trigger:** Writing content into any steering or docs file
- **Check:** Does this fact already have an authoritative home in another file?
- **Instruction:** If yes, write a reference (e.g., "See `steering/safety.md` RULE: ...") — do not restate. One fact, one location, references everywhere else.

### RULE: Anti-drift — don't write deliverables into ROADMAP
- **Trigger:** Adding or editing content in `docs/ROADMAP.md`
- **Check:** Am I writing the actual implementation content (code patterns, architecture details, rule text) instead of describing *what to do and why*?
- **Instruction:** ROADMAP entries describe planned work. The actual content belongs in the target file when the work is done. Keep ROADMAP entries to: what, why, and any constraints.

### RULE: Anti-drift — don't write implementation history
- **Trigger:** Writing content into steering or docs after completing a change
- **Check:** Am I recording "files changed", "API response shapes observed", or other ephemeral implementation details?
- **Instruction:** Stop. That's git history. Steering captures patterns and constraints that apply going forward, not a log of what was done.

### RULE: Pre-commit — scan for restated content
- **Trigger:** About to commit changes to any file in `.kiro/steering/` or `docs/`
- **Check:** Does the same rule or fact now exist in two places?
- **Instruction:** Collapse to one authoritative location + references from other locations. Same as verifying `npm run build` passes for code.

### RULE: Pre-commit — check file size
- **Trigger:** About to commit changes to a steering file
- **Check:** Does the file exceed ~250 lines?
- **Instruction:** If yes, either historical content crept in (remove it) or scope is too broad (split the file). Exception: `decisions.md` may grow larger — but only with entries that actively constrain future work.

### RULE: Decision graduation — remove decisions that are now structural facts
- **Trigger:** Reviewing `decisions.md` or considering whether a decision entry is still needed
- **Check:** Is this decision (1) fully expressed by another steering file, (2) enforced by tooling (e.g., lint rule), or (3) so obvious no agent would propose the alternative?
- **Instruction:** If any of these are true, the decision can be removed from `decisions.md`. **Keep** decisions where an agent might reasonably propose the alternative (e.g., "we use in-memory cache, not Redis" stays because agents will suggest Redis). The "Abandoned/Rejected approaches" section should never be pruned — it's the highest-value content for preventing wasted agent cycles.

---

## Knowledge: What a doc update looks like (concrete triggers)

- **Feature introduced a new pattern:** Add the pattern to the relevant steering file (`tech.md` or `code-structure.md`)
- **Feature required a non-obvious design choice:** Add to `decisions.md` (brief: decision + rationale + what was rejected)
- **Feature created a new shared file/module:** Add to the appropriate table in `code-structure.md`
- **Feature is planned but not yet built:** Add to `docs/ROADMAP.md`
- **None of the above applies:** Explicitly state "no doc updates needed" in the plan with brief justification

## Knowledge: How to apply

- Do NOT skip the post-feature rule even if the feature seems small — small features can introduce patterns that compound.
- Prefer updating existing files over creating new ones, unless the feature is large enough to warrant its own steering doc (like `admin-completion-detection.md`) or knowledge article (like `docs/crossover-courses.md`).

## Knowledge: Content type distinction

Every piece of content in steering is one of two types:

1. **RULE** — An enforceable constraint with a trigger condition. Something that can be *violated*. Has a binary answer: "did we follow this or not?" Formatted with `### RULE:` prefix and Trigger/Check/Instruction structure.

2. **KNOWLEDGE** — Context that informs judgment but can't be "violated." Architecture overviews, rationale, domain behavior, tables of shared utilities. Labeled with `## Knowledge:` prefix. Read during planning; skipped during audit.

The audit pass scans `### RULE:` blocks only. The planning pass reads everything.

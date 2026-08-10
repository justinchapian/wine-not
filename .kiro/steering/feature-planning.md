---
inclusion: always
---
# Steering & Documentation Governance

## Scope

This file governs **all** changes to `.kiro/steering/` and `docs/` — whether triggered by a new feature, a standalone documentation task, a bug fix, or any other work.

---

## Rule Format Specification

All rules in steering files MUST follow this format:

```markdown
### RULE: [Short imperative name]
- **Trigger:** [When does this rule fire?]
- **Check:** [What specific question must be answered?]
- **Instruction:** [What to do based on the answer?]
```

If a rule lacks any of these three components, it is a philosophical statement — not an enforceable constraint.

---

## Rules

### RULE: Post-feature — update steering and docs after every feature
- **Trigger:** A feature, bug fix, or significant change has been completed
- **Check:** Do any steering files or docs need to be updated to reflect the new work?
- **Instruction:** Analyze all files in `.kiro/steering/` and `docs/`. Update relevant files so future sessions have accurate context. When adding new content: if it's an enforceable constraint, format as `### RULE:` with Trigger/Check/Instruction. If it's context/architecture/rationale, use `## Knowledge:` prefix. If no updates are needed, explicitly state "no doc updates needed" with brief justification. If a ROADMAP item shipped, remove it.

### RULE: Anti-drift — don't restate facts that already have a home
- **Trigger:** Writing content into any steering or docs file
- **Check:** Does this fact already have an authoritative home in another file?
- **Instruction:** If yes, write a reference — do not restate. One fact, one location, references everywhere else.

### RULE: Anti-drift — don't write deliverables into ROADMAP
- **Trigger:** Adding or editing content in `docs/ROADMAP.md`
- **Check:** Am I writing the actual implementation content instead of describing *what to do and why*?
- **Instruction:** ROADMAP entries describe planned work. The actual content belongs in the target file when the work is done.

### RULE: Anti-drift — don't write implementation history
- **Trigger:** Writing content into steering or docs after completing a change
- **Check:** Am I recording "files changed" or other ephemeral implementation details?
- **Instruction:** Stop. That's git history. Steering captures patterns and constraints that apply going forward, not a log of what was done.

### RULE: Pre-commit — scan for restated content
- **Trigger:** About to commit changes to any file in `.kiro/steering/` or `docs/`
- **Check:** Does the same rule or fact now exist in two places?
- **Instruction:** Collapse to one authoritative location + references from other locations.

### RULE: Pre-commit — check file size
- **Trigger:** About to commit changes to a steering file
- **Check:** Does the file exceed ~250 lines?
- **Instruction:** If yes, either historical content crept in (remove it) or scope is too broad (split the file). Exception: `decisions.md` may grow larger.

### RULE: Decision graduation — remove decisions that are now structural facts
- **Trigger:** Reviewing `decisions.md` or considering whether a decision entry is still needed
- **Check:** Is this decision (1) fully expressed by another steering file, (2) enforced by tooling, or (3) so obvious no agent would propose the alternative?
- **Instruction:** If any of these are true, the decision can be removed from `decisions.md`. The "Abandoned/Rejected approaches" section should never be pruned — it prevents wasted agent cycles.

---

## Knowledge: Content type distinction

1. **RULE** — An enforceable constraint with a trigger condition. Formatted with `### RULE:` prefix and Trigger/Check/Instruction structure.
2. **KNOWLEDGE** — Context that informs judgment but can't be "violated." Labeled with `## Knowledge:` prefix.

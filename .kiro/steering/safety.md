---
inclusion: always
---
# Operational Safety — Plan Before Implement

## Context

This application manages **live LMS data for real users** through rate-limited external APIs (LearnUpon). Mistakes are not easily reversible — deleted enrollments, incorrect completions, and broken group memberships affect real learner records.

---

## Rules

### RULE: Explain before mutating — present a plan and wait for approval
- **Trigger:** Implementing any change that involves: creating/updating/deleting enrollments, switching portals or groups, bulk operations (any loop calling LMS API multiple times), changes to the request pipeline/circuit breaker/retry logic, changes to authentication or authorization flows, changes to cached data structures or invalidation logic, or changes to response guards or schema validation.
- **Check:** Have you presented a plan to the user that includes: (1) files to modify, (2) data impact (creates, mutates, deletes, or read-only), (3) failure mode (what happens if operation partially fails), (4) blast radius (what existing behavior could break)?
- **Instruction:** Present the plan and wait for explicit user confirmation ("go ahead", "looks good") before implementing. If the user raises concerns, revise the plan. Do NOT proceed without approval. This rule does NOT apply to: read-only changes (UI, display, queries), pure refactors that don't change behavior, adding tests, documentation/steering updates, or changes where the user has already described the exact implementation they want.

### RULE: Scope discipline — only modify what the task requires
- **Trigger:** Working on any task where you notice adjacent code that could be "improved"
- **Check:** Is the file/function you're about to modify necessary to satisfy the requested task? Or is it an unrelated refactor, dependency upgrade, architectural change, or cleanup?
- **Instruction:** Only modify files and functionality necessary for the current task. Do not perform unrelated refactoring, dependency upgrades, architectural changes, or cleanup unless explicitly requested. If you discover an improvement outside the current task, mention it to the user. If worth tracking, suggest adding it to `docs/ROADMAP.md`. Do not "improve" adjacent code while you're in the neighborhood.

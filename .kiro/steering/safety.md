---
inclusion: always
---
# Operational Safety — Plan Before Implement

---

## Rules

### RULE: Explain before mutating — present a plan and wait for approval
- **Trigger:** Implementing any change that involves: changes to authentication or authorization, changes to data structures or database schemas, integration with paid/metered external APIs, or any destructive/irreversible operation.
- **Check:** Have you presented a plan to the user that includes: (1) files to modify, (2) data impact, (3) failure mode, (4) what existing behavior could break?
- **Instruction:** Present the plan and wait for explicit user confirmation before implementing. This rule does NOT apply to: read-only UI changes, pure refactors, adding tests, or documentation updates.

### RULE: Scope discipline — only modify what the task requires
- **Trigger:** Working on any task where you notice adjacent code that could be "improved"
- **Check:** Is the file/function you're about to modify necessary to satisfy the requested task?
- **Instruction:** Only modify files and functionality necessary for the current task. Do not perform unrelated refactoring, dependency upgrades, or cleanup unless explicitly requested. If you discover an improvement outside the current task, mention it to the user.

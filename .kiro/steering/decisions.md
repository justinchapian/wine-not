---
inclusion: always
---
# Key Decisions & Rejected Approaches

> **Content type: Knowledge reference.** This file records active design decisions and rejected approaches. Rules derived from these decisions live in their respective steering files. This file explains *why* — and prevents agents from proposing approaches that have already been evaluated and rejected. See `feature-planning.md` RULE: "Decision graduation" for when entries can be removed.

## Open Architectural Questions

These are identified tensions from the product vision that require conscious decisions. They are NOT yet decided — preserve flexibility until real implementation forces a choice.

### Palate model: structured vs. freeform

**Tension:** The vision says "do not assume the palate can be represented by a fixed list of numerical axes" (§7), but recommendations require *some* structure to compare beverages against preferences.

**Options being considered:**
- Hybrid: structured dimensions for queryable comparisons + freeform qualitative observations for nuance
- Fully freeform with AI doing comparison at query time (more flexible, higher AI cost per recommendation)
- Evolving schema that adds dimensions as they emerge from data

**Not yet decided.** Will be informed by MVP implementation of the palate model.

---

### AI abstraction granularity

**Tension:** §25 says "create a provider abstraction" — but the AI performs many distinct tasks (review interpretation, OCR, recommendation reasoning, conversational Q&A, palate-model updates) with different prompt shapes, latency profiles, and cost characteristics.

**Options being considered:**
- One broad `AIProvider` interface with many methods
- Multiple domain-specific abstractions (e.g., `ExperienceInterpreter`, `RecommendationEngine`, `ImageAnalyzer`)
- Capability-based routing: one interface, multiple implementations selected by task type

**Not yet decided.** Will be informed by which AI capabilities are needed first in MVP.

---

### Canonical beverage identity

**Tension:** §20 says "one canonical record" — but wine identity is notoriously hard. "2024 Producer X Pinot Noir" typed differently is still the same wine. At what level do you canonicalize?

**Options being considered:**
- Lazy creation: canonical records created only when a user logs an experience, with fuzzy matching to detect duplicates
- External ID anchoring: use an external wine database ID as the canonical key where available
- AI-assisted matching: let the AI determine if two logged beverages are the same thing

**Not yet decided.** Directly affects whether canonical records are eagerly or lazily created, and how multi-user sharing works.

---

### Multi-user privacy vs. shared recommendations

**Tension:** §21 says privacy should be granular; §22 says two people should jointly query a wine list. The system needs to evaluate preference overlap *without merging models*.

**Options being considered:**
- Query-time projection: each user's model stays private, overlap computed ephemerally at recommendation time
- Shared-context sessions: users temporarily grant access to a subset of their profile for a specific recommendation
- Pre-computed compatibility scores between users who opt in

**Not yet decided.** This is a Phase 10 concern but the data model should not accidentally couple users early.

---

### Evidence chain storage

**Tension:** §36 says preserve the full chain (experience → interpretation → observations → evidence → preference → recommendation → new experience). This is valuable for explainability but shapes the data model significantly — every AI interpretation becomes an immutable artifact.

**Options being considered:**
- Full event-sourcing: every AI output is an immutable versioned record
- Snapshot + changelog: current state stored, with a log of changes and their triggers
- Lightweight: store original experience + current AI conclusions, with ability to re-derive

**Not yet decided.** Will be informed by storage cost vs. explainability needs in practice.

---

### Backend architecture: integrated vs. separate API

**Tension:** §30 says PWA (implying a frontend app), §35 says modular monolith. The vision says "backend/API should remain independent enough that a native client could be added later."

**Options being considered:**
- Full-stack framework (e.g., Next.js) with API routes alongside frontend — simpler deployment, one codebase
- Separate API server + PWA frontend — clearer separation, easier to add native client later
- Full-stack for MVP, extract API later if native client becomes needed

**Not yet decided.** Deployment simplicity vs. future flexibility.

---

## Design Decisions (user-defined, don't change without asking)

_None yet — will be recorded as implementation decisions are made._

## Abandoned/Rejected approaches

_None yet — will be recorded as alternatives are evaluated and rejected._

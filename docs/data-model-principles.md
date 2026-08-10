# Data Model Principles

> This document establishes the principles that constrain future schema design. It is NOT a schema. Implementation details (tables, collections, field names) will be decided during development. These principles ensure the data model serves the product vision regardless of the specific database technology chosen.

## The Six Data Categories

The application must maintain clear boundaries between these distinct concepts (Vision §35):

### 1. Canonical Data — "What the beverage is"

The objective identity of a wine or cocktail. Shared across all users.

- Producer, name, vintage, region, grapes (wine)
- Name, base spirit, ingredients, style (cocktails)
- External enrichment data (from wine databases, if available)
- NOT owned by any single user — if an external data source disappears, the canonical record persists with whatever data was captured

**Key principle:** One canonical record per identifiable beverage (Vision §20). Multiple users' experiences reference the same canonical record.

**Open question:** How to determine identity — see `decisions.md` "Canonical beverage identity."

### 2. User Experience — "What the user experienced"

The user's personal, subjective encounter with a beverage. Always owned by a single user.

- Rating
- Natural-language notes (preserved verbatim — never overwritten by AI)
- Optional context: date, location, food, occasion, people, price, photo
- Behavioral signals: would drink again, would buy, etc.
- Link to canonical beverage record

**Key principle:** The original user input is sacred. It is never modified by the AI. AI interpretation is stored separately.

### 3. AI Observation — "What the system inferred from the experience"

The AI's interpretation of a user experience. Distinct from the experience itself.

- Extracted flavor/aroma descriptors
- Inferred structured characteristics (acidity level, body, etc.)
- Identified sentiment patterns
- Follow-up questions asked and answered
- Confidence levels
- Timestamp and AI provider/model used

**Key principle:** AI observations are immutable once created. If the AI re-interprets an experience (due to model change or new context), a new observation is created — the old one is preserved for the evidence chain.

### 4. User Preference — "What the system currently believes about the user"

The aggregated, current-state model of the user's taste. Derived from evidence.

- Structured characteristics (e.g., "prefers moderate-to-high acidity")
- Qualitative preferences (e.g., "gravitates toward earthy/savory")
- Behavioral patterns (e.g., "only enjoys full-bodied reds with food")
- Contextual preferences (e.g., "lighter wines in summer")
- Each preference links to its supporting evidence

**Key principle:** Preferences are mutable (they update as new evidence arrives) but changes should be traceable. A preference without evidence is suspect.

### 5. Hypothesis — "Something the system believes may be true but hasn't established"

A prediction about the user's taste that needs more data to confirm or reject.

- The claim (e.g., "You would enjoy dry Riesling")
- The reasoning (e.g., "You consistently enjoy high-acid, mineral whites")
- Supporting evidence (which experiences led to this hypothesis)
- Discovery state: 🟡 Strong Hypothesis, 🔵 Wildcard
- Confidence level
- What would confirm it, what would reject it

**Key principle:** Hypotheses are explicitly tentative. They are not facts. They should be testable — the system should know what outcome would confirm or reject them.

### 6. Recommendation — "A temporary decision based on current evidence"

A specific suggestion made at a specific time in a specific context. Ephemeral.

- What was recommended
- Why (linked to preferences, hypotheses, context)
- The mode (Safe/Explore/Surprise/Challenge)
- Whether the user acted on it
- The outcome (if they tried it — links to a new experience)

**Key principle:** Recommendations are snapshots in time. They may become irrelevant as the palate model evolves. They should not be confused with preferences.

---

## The Evidence Chain (Vision §36)

Do not store only the final AI conclusion. Preserve the full chain:

```
Original experience (user input — immutable)
        ↓
AI interpretation (observations — immutable once created)
        ↓
Observations aggregate into evidence
        ↓
Evidence supports/contradicts preferences and hypotheses
        ↓
Preferences and hypotheses inform recommendations
        ↓
User acts on recommendation → new experience
        ↓
Loop continues
```

This enables:
- **Explainability** — "Why does Wine Not? think I like this?" can be answered with specific experiences
- **Reinterpretation** — If the AI model improves, old experiences can be re-analyzed without losing the original interpretation
- **User corrections** — An explicit correction overrides weak inferred evidence, and that correction itself becomes evidence
- **Debugging** — When the system gets something wrong, you can trace exactly where the reasoning went astray

---

## User Corrections (Vision §24)

User corrections are first-class data. They are NOT deletions or overwrites.

When a user says "That's wrong — I don't actually dislike high alcohol, that wine was just poorly balanced":

1. The correction is stored as a new evidence artifact
2. It has higher weight than weak AI inferences
3. The affected preference/hypothesis is re-evaluated
4. The original AI interpretation that led to the wrong conclusion is preserved (for debugging and reinterpretation)

---

## Multi-User Data Boundaries (Vision §21–22)

### What is shared
- Canonical beverage records (the objective "what is this wine" data)

### What is private by default
- User experiences
- AI observations about a user's experiences
- User preferences / palate model
- Hypotheses

### What can be selectively shared
- Individual experiences (user chooses to share a specific review/rating)
- Recommendation context (user opts into a shared recommendation session)

### What is NEVER merged
- Palate models — shared recommendations evaluate overlap at query time, they do not combine two users' models into one

---

## Data Ownership & Portability (Vision §27)

The user's data is the most valuable asset in the system.

- Export must be possible in standard formats (JSON, CSV at minimum)
- All user-generated content (notes, ratings, corrections, photos) must be exportable
- AI-derived data (observations, preferences, hypotheses) should also be exportable
- If an external wine database API disappears, the user retains everything that was captured
- Avoid lock-in to any single external data provider

---

## Privacy Constraints (Vision §28)

- Personal experiences are never public by default
- AI providers receive only the information necessary for the requested operation (not the entire palate history if only a single experience is being interpreted)
- The architecture should make it clear what data flows where

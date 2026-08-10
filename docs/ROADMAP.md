# Roadmap

> Items are removed from this file after they ship. See `git log` for history. See Vision §34 for the original phased progression.

## Development Phases

The exact sequencing may change as development progresses. Each phase should result in a usable application.

### Phase 1 — Personal wine/cocktail journal

The foundation. A user can log beverages, write notes, rate them, add optional context. No AI yet — just a fast, pleasant logging experience on mobile.

**Includes:**
- Mobile-first PWA scaffold with Wine Not? branding
- Wine mode + Cocktail mode (separate but within one app)
- User identity (simple auth for a small private group)
- Canonical beverage records
- Personal experience logging (rating, natural-language notes, optional context)
- Basic search/browse of logged beverages

### Phase 2 — AI interpretation

The system starts learning from what the user logs. AI processes experiences after they're recorded.

**Includes:**
- AI provider abstraction (first implementation TBD)
- Experience interpretation: extract observations from natural-language notes
- Store AI observations as distinct artifacts (not merged into user data)
- Basic follow-up questions when input is ambiguous

### Phase 3 — Evolving palate model

The system forms and updates a model of the user's preferences.

**Includes:**
- Palate model data structure (hybrid structured + qualitative — see open question in `decisions.md`)
- Preference tracking with evidence links
- Palate profile view ("Here's what we know about your taste")
- Evidence traceability ("Why does Wine Not? think I like this?")

### Phase 4 — Discovery categories and hypotheses

The four-category system becomes active.

**Includes:**
- 🟢 Known Hits, 🟡 Strong Hypotheses, 🔵 Wildcards, 🔴 Known Misses / Caution
- Hypothesis formation from accumulated evidence
- Discovery state tracking (each preference/hypothesis gets a category)
- User corrections ("That's wrong" → updates model, preserves correction as evidence)

### Phase 5 — Personal recommendations

The system actively recommends what to try next.

**Includes:**
- Recommendation modes (Safe, Explore, Surprise Me, Challenge Me)
- Natural-language recommendation requests
- "Would I like this?" queries
- Explanation of recommendations (why this, what you could learn)

### Phase 6 — Challenge Me & discovery-oriented recommendations

The AI becomes a curious tasting companion rather than just a predictor.

**Includes:**
- Proactive challenges ("Plot twist: you may not dislike Chardonnay")
- Information-value-driven recommendations
- Cross-domain hypotheses (wine ↔ cocktail connections)
- "We should investigate that" moments

### Phase 7 — Image scanning & beverage identification

Camera becomes a primary input.

**Includes:**
- Wine label scanning → identify and log
- Cocktail menu photo → extract items
- Image recognition provider abstraction
- Quick-log from scan

### Phase 8 — Restaurant/store menu analysis

The "pocket sommelier at dinner" experience.

**Includes:**
- Wine list photo → ranked recommendations based on palate + context
- Store shelf scan → "one safe, one hypothesis, one wildcard"
- Price as a recommendation factor
- Food pairing context

### Phase 9 — Inventory

Lightweight personal collection tracking.

**Includes:**
- "I have this at home" recording (scan or manual)
- "What should I drink tonight?" queries against inventory
- "Which of my wines goes best with X?" pairing from owned bottles

### Phase 10 — Small-group sharing & collaborative recommendations

Multi-user features for family and close friends.

**Includes:**
- Granular privacy (private / shared with selected people / shared recommendation)
- Shared recommendation queries ("Find us something we'll both enjoy")
- Overlap evaluation without merging palate models
- Multi-user canonical beverage records

---

## Tech Debt

_Nothing yet — clean slate._

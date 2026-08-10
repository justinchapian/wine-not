# AI Integration Principles

> This document establishes the principles that govern how AI is integrated into Wine Not?. It does NOT specify providers, models, or prompt engineering details — those will be decided during implementation. These principles ensure AI integration serves the product vision regardless of which providers or models are used.

## Provider Independence (Vision §25)

### The core constraint

Do not hard-code the application around a single AI provider. The architecture must allow providers to be changed later without rewriting domain logic.

### What this means in practice

- Domain logic (palate model updates, recommendation scoring, discovery categorization) should NOT contain provider-specific code
- AI calls should flow through an abstraction that accepts structured inputs and returns structured outputs
- Prompt templates, model names, API keys, and response parsing are implementation details of the provider layer — not visible to domain logic
- If a provider's API changes or pricing becomes unacceptable, switching should be a provider-layer change, not a domain rewrite

### Abstraction granularity (open question)

See `decisions.md` — the exact shape of the abstraction (one broad interface vs. multiple domain-specific ones) is not yet decided. The principles below apply regardless of that decision.

---

## AI Capabilities Map

The AI performs distinct types of work with different characteristics:

| Capability | Input | Output | Latency tolerance | Cost sensitivity |
|------------|-------|--------|-------------------|------------------|
| Experience interpretation | Natural-language notes + optional context | Structured observations (flavor descriptors, sentiment, characteristics) | Seconds OK (async after logging) | High — runs on every experience |
| Follow-up question generation | Ambiguous user input | 1-2 clarifying questions | Sub-second (interactive) | Medium |
| Beverage identification | Image (label/menu) + optional text | Canonical beverage match or new record | Seconds OK | Medium |
| Palate model update | New observations + existing model | Updated preferences/hypotheses | Seconds OK (async) | High — runs on every experience |
| Recommendation reasoning | User intent + palate model + available options | Ranked recommendations with explanations | 1-3 seconds (interactive) | Low — runs on explicit request |
| Natural-language conversation | User question + relevant context | Conversational answer | 1-3 seconds (interactive) | Low — runs on explicit request |
| Image/OCR analysis | Photo of menu/shelf/label | Extracted text + identified items | Seconds OK | Medium |
| Cross-domain hypothesis | Wine profile + cocktail profile | Potential connections with evidence | Seconds OK (async/batch) | Low — runs infrequently |

### Implications for architecture

- **High-frequency, high-cost-sensitivity tasks** (experience interpretation, palate updates) should use smaller/cheaper models where quality is sufficient
- **Low-frequency, quality-critical tasks** (recommendation reasoning, challenge generation) can justify more capable/expensive models
- **Interactive tasks** (conversation, follow-ups) need low latency — model selection should factor in response time
- **Async tasks** (interpretation, palate updates) can tolerate higher latency and could be queued

---

## When AI Fires (Vision §29)

AI does not run continuously. It fires on events:

- **New experience logged** → interpretation + palate model update
- **Beverage scanned** → identification + enrichment
- **Menu analyzed** → OCR + identification + ranking
- **Recommendation requested** → reasoning against palate model
- **Profile question asked** → conversational response with evidence
- **User correction made** → re-evaluation of affected preferences/hypotheses

Between events, the system is passive. No background processing, no scheduled AI jobs (at least initially).

---

## Cost Philosophy (Vision §29)

### Principles

- Prefer smaller models for routine extraction (experience interpretation is high-volume)
- Reserve capable models for complex reasoning (recommendations, challenges, cross-domain hypotheses)
- AI calls should be event-driven, not continuous
- The system should be functional (as a journal) even if AI is temporarily unavailable or rate-limited
- Monitor cost per user-action to detect unexpected spikes

### What "inexpensive to operate" means

For a personal app with <10 users:
- AI costs should be on the order of dollars per month, not tens or hundreds
- If a single experience log triggers an AI call, that call should cost fractions of a cent
- Recommendation queries (less frequent, more complex) can cost more per call but should still be negligible at this usage scale

---

## Explainability & Evidence (Vision §23)

### The explainability requirement

Every AI-derived preference or hypothesis must be traceable to the experiences that produced it.

If the system says "You tend to dislike high perceived alcohol," the user must be able to ask "Why?" and see the supporting experiences.

### What must be preserved

- **Original user input** — The verbatim text the user wrote (never modified by AI)
- **AI interpretation** — What the AI extracted/inferred (stored as a separate artifact — see `data-model-principles.md`)
- **Confidence** — How certain the AI is about each observation
- **Evidence links** — Which experiences support which preferences/hypotheses
- **Profile changes** — When and why the palate model changed
- **User corrections** — Explicit overrides and their impact

### What this means for prompts

AI prompts should request structured output that includes confidence levels and reasoning, not just conclusions. The system needs the AI's "thinking" in a parseable form, not just its final answer.

---

## User Corrections Override Inference (Vision §24)

When the AI's inference conflicts with an explicit user correction:

1. The user correction wins
2. The original AI inference is preserved (for the evidence chain) but marked as overridden
3. The correction itself becomes a high-weight evidence artifact
4. Affected hypotheses and preferences are re-evaluated

The system should be designed so that corrections are easy to make and immediately visible in their effect.

---

## Privacy & Data Minimization (Vision §28)

### What goes to AI providers

- Only the information necessary for the specific operation
- NOT the user's entire palate history when only interpreting a single new experience
- Context should be provided selectively (e.g., "here are the user's known preferences relevant to this recommendation" rather than "here is everything")

### What stays local

- The full palate model
- The complete evidence chain
- User corrections
- Cross-user comparisons (for shared recommendations — computed locally, not sent to AI)

### Auditability

The architecture should make it possible to understand what information was sent to which AI provider for which operation. This doesn't require a complex audit log initially — clear code boundaries and structured AI call patterns are sufficient.

---

## The AI's Personality (Vision §10, §39)

The AI should behave as a curious tasting companion, not a search engine or generic chatbot.

### It should be:
- Curious ("I wonder why you liked this")
- Evidence-based (never claims more than the data supports)
- Willing to challenge ("Plot twist: you may not dislike Chardonnay")
- Honest about uncertainty ("We don't have enough data to know yet")
- Helpful without being sycophantic

### It should NOT be:
- Agreeable when evidence contradicts the user's stated belief
- Confident when evidence is thin
- Preachy or condescending about wine knowledge
- Generic or template-y in its responses

### How personality is implemented

Personality is a prompt-layer concern, not a domain-logic concern. The domain logic determines *what* to say (e.g., "we have contradictory evidence about Chardonnay"), and the prompt layer determines *how* to say it (e.g., "Plot twist: you may not dislike Chardonnay — let's investigate").

This separation means personality can be tuned without changing business logic.

---

## Graceful Degradation

If AI is unavailable (provider outage, rate limit, API key issue):

- The journal should still work (logging, browsing, searching)
- Existing palate model data should still be viewable
- Recommendations based on already-computed preferences should still be possible (even if less dynamic)
- The system should clearly indicate when AI features are degraded rather than silently failing

AI is an enhancement to a functional journal, not the foundation that everything else depends on.

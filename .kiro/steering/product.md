---
inclusion: always
---
# Product — Wine Not?

> **Content type: Knowledge reference.** This file describes what the app is, who uses it, and its guiding principles. It contains no auditable rules. For the full product vision, see `docs/VISION.md`.

## What it is

**Wine Not? — Your Pocket Sommelier**

An AI-powered personal palate and beverage discovery application. It helps users discover, understand, and refine their wine and cocktail preferences over time through a continuous learning loop.

It is NOT primarily a wine database, rating site, cellar manager, social network, or generic chatbot. Those capabilities may exist, but they serve the core purpose:

> A personal palate that learns from experience and helps its owner discover what they like — including things they don't yet know they like.

## Who uses it

- Justin (developer/primary user) and a small circle of family and friends
- Not a commercial product — personal project, simple auth, no billing, no multi-tenancy complexity
- Users are normal people, not wine experts — the app should never require wine/cocktail vocabulary as a prerequisite

## Core philosophy

**Optimize for discovery, not merely prediction.**

The system should help users learn their own palate, not just repeat what they already know they like. Contradictions are learning opportunities. The AI should challenge assumptions when evidence supports it.

## The core loop

```
Log → Learn → Hypothesize → Explore → Challenge → Learn Again
```

Every feature, architectural decision, and product behavior should serve this loop.

## Two modes: Wine and Cocktails

Separate but connected experiences with shared underlying intelligence. The AI has access to both profiles and can form cross-domain hypotheses (conservative, explainable, correctable).

## Discovery categories (foundational concept)

- 🟢 **Known Hits** — Actually tried and clearly enjoyed
- 🟡 **Strong Hypotheses** — Predicted to work, insufficient direct evidence
- 🔵 **Wildcards** — Don't obviously fit the model but could reveal something valuable
- 🔴 **Known Misses / Caution** — Negative evidence preserved with appropriate specificity

## Key principles

- **Don't pigeonhole the user** — Taste is contextual, contradictory, evolving. Preserve exceptions.
- **AI challenges assumptions** — Respectfully, evidence-based, explicit about uncertainty.
- **Natural language is first-class** — Users speak normally; the AI translates to structure.
- **Evidence is traceable** — Every inference links back to experiences.
- **User corrections override weak inference** — Explicit statements trump AI guesses.
- **Privacy-first** — Personal data is never public by default.
- **Data ownership** — The user's data is the most valuable asset. Export in standard formats. No lock-in.
- **Cost-conscious** — Inexpensive to operate. AI fires on events, not continuously.

## Platform

Mobile-first PWA. Should feel app-like, support camera input, be installable. Backend independent enough for a native client later if needed.

## Visual identity

Playful, curious, warm, sophisticated without pretense. Slightly irreverent. Not enterprise, not clinical, not generic AI chatbot.

## What success looks like

Wine Not? helps the user discover preferences they didn't previously know they had. The "Plot twist: you may not dislike Chardonnay" moments are the heart of the product.

## The guiding question

> Does this help Wine Not? understand the user's palate, preserve that understanding, or help the user discover something valuable?

If not, it probably doesn't belong in the core product.

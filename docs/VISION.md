# Wine Not? — Product Vision & Project Foundation

> **This is the original product vision. It is the source of truth for the project's philosophy, principles, and intent. Steering files summarize and operationalize it; this document preserves the full thinking. The vision may evolve — features and details will adapt — but the core loop and philosophy are foundational.**

---

## 1. What Is Wine Not?

Wine Not? is a private, playful, AI-powered personal palate and beverage discovery application.

Its primary purpose is to help a person discover, understand, and refine their preferences for wine and cocktails over time.

It is not primarily:

- A wine database
- A wine-rating website
- A social network
- A cellar-management application
- A generic AI chatbot
- A recommendation engine that simply finds things similar to previous favorites

Those capabilities may exist, but they serve a larger purpose.

The core product is:

> A personal palate that learns from experience and helps its owner discover what they like — including things they don't yet know they like.

The application should eventually feel like having a knowledgeable, curious pocket sommelier who knows the user's history, understands their preferences, remembers what they have tried, challenges their assumptions, and helps them decide what to explore next.

---

## 2. The Central Product Philosophy

Wine Not? should optimize for discovery, not merely prediction.

A conventional recommendation system might learn:

> "You liked Gamay, therefore you should drink more Gamay."

Wine Not? should instead think:

> "You loved Gamay. What does that tell us about your palate, and what else might you enjoy because of those characteristics?"

The distinction is fundamental.

The system should continuously balance:

- Familiarity
- Predicted enjoyment
- Novelty
- Curiosity
- Information value
- Confidence
- Context
- User intent

The goal is not to perfectly predict every rating.

The goal is to help the user learn their own palate.

---

## 3. The Core Loop

The entire application should revolve around this cycle:

```
       EXPERIENCE
           │
           ▼
         LOG IT
           │
           ▼
       AI ANALYSIS
           │
           ▼
     UPDATE PALATE MODEL
           │
           ▼
      FORM HYPOTHESES
           │
           ▼
       RECOMMEND
           │
           ▼
        EXPLORE
           │
           ▼
     HAVE ANOTHER EXPERIENCE
           │
           └──────────────► LEARN AGAIN
```

This should influence the architecture, data model, AI design, and UX.

---

## 4. Wine and Cocktails Are Two Modes

Wine and cocktails should be separate but connected experiences.

The application should have two primary modes:

### Wine

Wine-specific functionality should include:

- Wines
- Producers
- Regions
- Grapes
- Vintages
- Wine experiences
- Wine tasting notes
- Wine recommendations
- Wine discovery
- Restaurant wine lists
- Wine-food pairing
- Wine inventory

### Cocktails

Cocktail-specific functionality should include:

- Cocktails
- Spirits
- Ingredients
- Styles
- Cocktail experiences
- Cocktail tasting notes
- Cocktail recommendations
- Cocktail discovery
- Bar menus
- Cocktail-food pairing where useful

The two modes should have appropriate terminology and UX.

The application should not force wine and cocktails into a single generic "beverage" experience if doing so makes the user experience worse.

However, they should share the same underlying user and AI ecosystem.

---

## 5. Shared Intelligence Across Wine and Cocktails

Wine and cocktail profiles should be distinct, but the AI should have access to both when useful.

For example, the system might discover:

**Wine** — The user repeatedly enjoys: savory, earthy, spicy, complex, dry

**Cocktails** — The user repeatedly enjoys: herbal, bitter, spirit-forward, complex, restrained sweetness

The system could form a cross-domain hypothesis:

> "You appear to enjoy complex, savory/herbal flavors more than straightforward sweetness. That may be why you respond well to both earthy reds and bitter/herbal cocktails."

This should be a hypothesis supported by evidence, not an assumed fact.

Cross-domain learning should be conservative and explainable.

The user should be able to correct it.

---

## 6. The App Is for Normal People, Not Wine Geeks

Wine Not? should not require users to understand:

- tannins
- acidity
- terroir
- varietals
- appellations
- oxidative aging
- tasting vocabulary
- cocktail taxonomy

A user should be able to write:

> "I liked it but it was kind of sour."

or:

> "I don't know why, but something about this bothered me."

or:

> "This was really good. I could drink this all night."

The AI should extract whatever useful information it can.

Where appropriate, it can ask simple follow-up questions to clarify.

For example:

> "When you say sour, did it feel more like fresh citrus/tartness, or did it feel unpleasantly sharp?"

The user should gradually become better at describing their palate if they want to, but wine expertise should never be a prerequisite for using the application.

---

## 7. The Palate Model

The user's palate should be represented as an evolving model rather than a static profile.

The model should contain multiple types of information.

### Structured characteristics

Potential examples:

- Acidity
- Tannin
- Body
- Sweetness
- Fruit intensity
- Earthiness
- Spice
- Herbal character
- Floral character
- Oak
- Alcohol/perceived heat
- Bitterness
- Aromatic intensity

The exact dimensions should remain flexible and should be refined through development.

Do not assume the user's palate can ultimately be represented by a fixed list of numerical axes.

### Qualitative preferences

Examples: Savory, Earthy, Mineral, Floral, Herbal, Smoky, Nutty, Oxidative, Jammy, Crisp, Rich, Funky

### Behavioral preferences

Examples: Would drink again, Would order again, Would buy, Would seek out, Would recommend, Only enjoyed with food, Enjoyed in a particular context

### Contextual preferences

Examples: Restaurant, Home, Date night, Casual evening, With steak, With pizza, Summer, Winter, Budget, Special occasion

### Discovery state

Each preference/hypothesis can have a discovery state such as:

- 🟢 Known Hit
- 🟡 Strong Hypothesis
- 🔵 Wildcard
- 🔴 Known Miss / Caution

### Evidence

Every significant inference should be traceable to experiences.

The system should be able to answer:

> "Why does Wine Not? think I like this?"

and provide the underlying evidence.

---

## 8. Discovery Categories

The four discovery categories are a foundational product concept.

### 🟢 Known Hits

Things the user has actually tried and clearly enjoyed.

Known Hits are established through actual user experiences. They should serve as anchors for finding adjacent discoveries.

The application should distinguish between:

> "The user liked this specific wine."

and:

> "The user has repeatedly enjoyed this style."

### 🟡 Strong Hypotheses

Things the system predicts are likely to work but which lack sufficient direct evidence.

These are explicitly hypotheses, not facts. When the user tries one, the result should update the hypothesis. A confirmed hypothesis may become a Known Hit. A failed hypothesis should provide negative evidence and potentially reveal something unexpected.

### 🔵 Wildcards

Things that do not obviously fit the current palate model but could reveal something valuable.

Wildcards are not random recommendations. They should have a reason for being selected.

A wildcard recommendation should communicate:

- Why it is unusual
- Why it might work
- What the user could learn
- How risky the recommendation is

### 🔴 Known Misses / Caution

Negative evidence should be preserved.

The system must distinguish:

- **Known Miss** — Strong evidence of dislike.
- **Caution** — Some negative evidence exists, but it may be too contextual or narrow to generalize.

The system should avoid turning "I hated this Chardonnay" into "The user hates Chardonnay" unless sufficient evidence exists.

---

## 9. Do Not Pigeonhole the User

This is one of the most important principles.

The application should not reduce a person to: "Justin likes high-acid, low-tannin wines."

That may be directionally useful, but it is not the user's identity.

Taste is contextual, contradictory, evolving, and sometimes surprising.

The system should preserve exceptions. Contradictions should be treated as opportunities for learning.

---

## 10. AI Should Be Allowed to Challenge the User

This is a defining feature of Wine Not?.

The AI should not merely agree with the user.

If the user says "I don't like Chardonnay" but the application sees that every negative Chardonnay experience involved heavy oak, it should eventually say:

> "I think we may be blaming Chardonnay for something else."

The AI should be respectful, evidence-based, and explicit about uncertainty.

It should never pretend to know more than the evidence supports.

But challenging assumptions is a feature, not a failure.

---

## 11. Recommendation Modes

Recommendations should adapt to the user's intent.

Possible modes include:

- **Safe** — "Give me something I'm very likely to enjoy." Optimize for confidence and familiarity.
- **Explore** — "Give me something new that you're reasonably confident I'll enjoy." Balance predicted enjoyment and novelty.
- **Surprise Me** — "Give me something unexpected." Prioritize novelty and interesting possibilities.
- **Challenge Me** — "Find something that tests one of our assumptions about my palate." Prioritize information value.

These modes should be available as natural language prompts as well as potentially explicit UI controls.

---

## 12. Natural Language Is a First-Class Interface

The application should allow the user to speak naturally.

Examples:

- "I want something easy to drink tonight."
- "Give me a red under $30."
- "I'm making steak."
- "I want something weird."
- "I want something I know I'll like."
- "I've had too many big reds lately. Give me something different."
- "Which of these would I like?"
- "Why do I keep liking these wines?"
- "Do I actually dislike Chardonnay?"

The AI should translate these requests into structured recommendation constraints when necessary.

---

## 13. Restaurant Wine-List Scanning

One of the major eventual features is restaurant menu analysis.

The user should be able to photograph a wine menu. Wine Not? should:

1. Extract wine names from the image.
2. Identify wines where possible.
3. Enrich them with relevant information.
4. Compare them to the user's palate.
5. Consider price.
6. Consider food/context if provided.
7. Rank the options.
8. Explain recommendations.

---

## 14. Store/Shelf Scanning

The same concept should work in a wine shop.

The user could photograph a shelf and say:

> "I have $25–35. Give me one safe choice, one hypothesis, and one wildcard."

Wine Not? should identify the visible bottles where possible and provide recommendations using the discovery categories.

---

## 15. Cocktail Menu Scanning

Cocktail mode should eventually support the same interaction.

The user can photograph a cocktail menu and ask for recommendations filtered through their cocktail palate.

---

## 16. Food Pairing

Food should be a contextual variable, not a separate isolated system.

The recommendation should consider: user preferences, beverage characteristics, food characteristics, context.

---

## 17. Personal Wine Inventory

The application does not need to become a full cellar-management system.

A lightweight inventory feature is valuable — the user can record "I have this bottle at home" and ask questions about what to drink, what pairs with food, etc.

Inventory should remain intentionally lightweight. The purpose is to help the user use their collection intelligently, not manage a commercial wine cellar.

---

## 18. Price Is a Recommendation Variable

Price should be treated as one contextual factor among many. The system should not assume that expensive means better. It should recognize value.

---

## 19. Optional Context

Experience logging should support optional context without making logging tedious.

Potential fields: Date, Restaurant/bar, Food, Occasion, People, Price, Location, Photo, Whether purchased, Whether consumed at home.

These should be optional. The primary logging flow should remain fast.

---

## 20. One Canonical Beverage Record

There should be one canonical record for a wine or cocktail where appropriate.

The beverage itself is shared. The experiences are personal.

This allows multiple people to interact with the same beverage without duplicating canonical data.

---

## 21. Multi-User Design

Wine Not? should initially be useful to one person but should support a small private group.

Privacy should be granular. A user should be able to choose whether an experience is private, shared with selected people, or used for a shared recommendation.

Sharing should not automatically mean exposing the user's entire palate history.

---

## 22. Shared Recommendations

One of the most valuable multi-user scenarios is collaborative discovery.

Two people at a restaurant can photograph the wine list and ask: "Find us something under $90 that we'll both enjoy."

Wine Not? evaluates the overlap between their preferences. This should be possible without merging the two people's palate models.

---

## 23. AI Evidence and Explainability

AI-derived preferences should be traceable.

The application should preserve: original user input, AI interpretation, confidence, evidence, profile changes, user corrections.

---

## 24. User Corrections Are Important Data

If the AI gets something wrong, the user should be able to correct it. That correction should become part of the model. The application should prioritize explicit user statements over weak inferred evidence.

---

## 25. AI Provider Independence

Do not hard-code the application around a single AI provider.

Create a provider abstraction capable of supporting different AI models/services.

Potential capabilities: review analysis, image/OCR analysis, beverage identification, structured observation extraction, taste-profile reasoning, recommendation reasoning, natural-language conversation.

The architecture should allow providers to be changed later.

---

## 26. External Wine Data

Do not attempt to build a giant wine database.

The application's canonical data should focus on beverages the user actually encounters.

External data providers may be used for enrichment and identification. However, external services must remain replaceable.

If a wine API disappears, Wine Not? should retain all user data, experiences, ratings, notes, observations, and taste history.

External data should enrich the user's record rather than own it.

---

## 27. Data Ownership

The user's data is the most valuable asset in the system.

The application should make it possible to export personal data in standard formats (JSON, CSV, photos, reviews, ratings, beverage records, taste observations, profile, hypotheses, evidence).

Avoid creating unnecessary lock-in.

---

## 28. Privacy

This is a private-first application.

Personal tasting experiences should not be public by default.

AI providers should only receive the information necessary to perform the requested operation.

Privacy should be a design principle rather than an afterthought.

---

## 29. Cost Philosophy

The application should remain inexpensive to operate.

Prefer: open-source software, low-cost hosting, simple infrastructure, event-driven AI calls, smaller models for routine extraction, more capable models only when justified, replaceable external APIs.

The AI does not need to run continuously. AI calls should happen when useful: new experience logged, beverage scanned, menu analyzed, recommendation requested, profile question asked.

---

## 30. Platform

The initial application should be a mobile-first PWA rather than a native Android application.

The PWA should: work well on mobile, support camera/photo input, be installable on the home screen, feel app-like, have fast touch-friendly interactions.

Native Android development should be deferred unless actual usage demonstrates that native capabilities are necessary.

The backend/API should remain independent enough that a native client could be added later.

---

## 31. Visual Identity

Wine Not? should have a playful, approachable visual identity.

Tagline: **Wine Not? — Your Pocket Sommelier**

The application should feel: curious, fun, personal, slightly irreverent, warm, sophisticated without being pretentious.

Avoid: enterprise dashboards, clinical interfaces, generic AI chatbot aesthetics, overly serious wine-app styling.

Potential UI language: "Known Hit", "Strong Hypothesis", "Wildcard", "Proceed With Caution", "Let's investigate.", "We learned something.", "Are we sure?", "Plot twist.", "You might be onto something."

The playful language should enhance the experience without making the application feel gimmicky.

---

## 32. Suggested Initial Navigation

A possible starting structure:

```
┌───────────────────────────────┐
│          WINE NOT?             │
│       Your Pocket Sommelier    │
├───────────────────────────────┤
│                               │
│     🍷 WINE   │   🍸 COCKTAIL │
│                               │
├───────────────────────────────┤
│                               │
│   What are we exploring?      │
│                               │
│   🟢 Known Hits               │
│   🟡 Strong Hypotheses        │
│   🔵 Wildcards                │
│   🔴 Caution                  │
│                               │
├───────────────────────────────┤
│                               │
│       + LOG SOMETHING         │
│                               │
├───────────────────────────────┤
│ Home | Log | Scan | Ask | Me │
└───────────────────────────────┘
```

This is illustrative, not a mandated final design.

---

## 33. Initial MVP

The MVP should establish the learning loop before attempting advanced computer vision or external integrations.

### Core

- Mobile-first PWA
- Wine Not? branding
- Wine mode
- Cocktail mode
- User identity
- Canonical beverage records
- Personal experiences
- Ratings
- Natural-language notes
- Optional context
- Basic AI interpretation
- Basic evolving palate model
- Discovery categories
- Evidence
- User corrections

### Initial AI capabilities

- Interpret reviews
- Extract useful observations
- Identify potential preferences
- Identify potential dislikes
- Maintain hypotheses
- Explain why something is recommended
- Answer basic "Would I like this?" questions

### Defer initially

- Restaurant menu OCR
- Wine-label scanning
- Advanced wine APIs
- Complex food pairing
- Advanced collaborative recommendations
- Native Android
- Public social features
- Full cellar management

---

## 34. Development Philosophy

Do not attempt to implement the entire vision at once. Develop in small, testable increments. Each increment should result in a usable application.

A sensible progression:

1. Personal wine/cocktail journal
2. AI interpretation
3. Evolving palate model
4. Discovery categories and hypotheses
5. Personal recommendations
6. "Challenge Me" and discovery-oriented recommendations
7. Image scanning and beverage identification
8. Restaurant/store menu analysis
9. Inventory
10. Small-group sharing and collaborative recommendations

The exact sequencing may change as development progresses.

---

## 35. Architectural Guidance

The project should deliberately avoid premature complexity.

Start with a modular monolith rather than microservices.

Keep domain logic, AI integration, external API integrations, database, authentication, and frontend cleanly separated without unnecessarily distributing them across multiple services.

Use clear interfaces around external dependencies (e.g., AIProvider, WineDataProvider, ImageRecognitionProvider). These should be replaceable.

The application should have a clear distinction between:

- **Canonical data** — What the beverage is.
- **User experience** — What the user experienced.
- **AI observation** — What the system inferred from the experience.
- **User preference** — What the system currently believes about the user.
- **Hypothesis** — Something the system believes may be true but has not established.
- **Recommendation** — A temporary decision based on current evidence.

This separation is extremely important.

---

## 36. Important Data Principle

Do not store only the final AI conclusion.

Preserve the chain:

```
Original experience
        ↓
AI interpretation
        ↓
Observations
        ↓
Evidence
        ↓
Preference/hypothesis
        ↓
Recommendation
        ↓
New experience
```

This allows the system to reconsider old assumptions when new evidence appears. It also makes the AI behavior explainable and debuggable.

---

## 37. Recommendation Philosophy

The recommendation engine should eventually consider at least:

- Predicted enjoyment
- Novelty
- Information value
- Confidence
- User intent
- Context
- Price
- Food pairing
- Available inventory

Do not implement this as a giant arbitrary scoring formula immediately. The architecture should make these factors available so that the recommendation strategy can evolve.

---

## 38. Success Criteria

Wine Not? succeeds if, after using it over time, the user can encounter something unfamiliar and ask "Would I like this?" and receive an answer that feels genuinely informed by their personal history.

But the stronger success condition is:

> Wine Not? helps the user discover preferences they didn't previously know they had.

These moments are the heart of the product.

---

## 39. The Product Should Be Curious

Wine Not? should embody curiosity.

It should sometimes say: "You liked this. I wonder why.", "This one doesn't fit your usual pattern.", "We should investigate that.", "I think our hypothesis might be wrong.", "Plot twist: you may not dislike Chardonnay."

The AI should behave less like a search engine and more like a curious tasting companion who keeps careful notes.

---

## 40. The Ultimate Vision

Over time, Wine Not? should become a deeply personal model of the user's taste.

The application should continually move from: simple preference → nuanced hypothesis → experiment → evidence → better hypothesis.

---

## 41. The Guiding Question

Whenever considering a feature, architectural decision, or product behavior, ask:

> Does this help Wine Not? understand the user's palate, preserve that understanding, or help the user discover something valuable?

If not, it probably does not belong in the core product.

The central loop remains:

**Log → Learn → Hypothesize → Explore → Challenge → Learn Again**

Wine Not? should not simply tell people what they like. It should help them find out.

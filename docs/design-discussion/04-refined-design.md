# SympleHost Review Intelligence Agents — Product Roadmap

> Working design document. This document turns the original broad idea into a practical phased roadmap for building LLM-powered agents that analyse property listings, guest reviews, ratings, and stay context, then convert them into ranked, evidence-backed actions for property managers.

---

## 0. Executive summary

SympleHost should not build a generic “review sentiment analysis” tool.

The product should be:

> **An AI decision copilot for property managers that turns guest reviews and listing data into ranked, evidence-backed actions across operations, maintenance, listing accuracy, guest communication, pricing readiness, and owner reporting.**

The core idea is simple:

```text
Every review becomes a structured signal.
Every signal becomes a possible action.
Every action is backed by evidence.
Every manager decision is captured.
Every completed action is tracked.
The system learns what works.
```

The first version should stay small and useful. It should not overpromise causal ROI, autonomous decision-making, or complex forecasting. The first job is to prove that the system can read reviews, understand the operational meaning, and produce recommendations that a real property manager would accept.

---

## 1. Original problem statement

We have property listings and customer reviews.

The business objects are:

```text
Property Manager
    ↓
Property / Listing
    ↓
Reservation / Stay
    ↓
Guest / Traveller
    ↓
Review
    ↓
Review Aspects
    ↓
Insights / Actions / Outcomes
```

The system needs to analyse reviews and determine the key aspects in each review. But the deeper goal is not only extraction. The real goal is to help property managers decide:

* What should be fixed?
* What should be rewritten in the listing?
* What should be escalated to maintenance?
* What should be added to the house guide?
* What should be changed in pre-arrival communication?
* What is hurting ratings?
* What is only a minor annoyance?
* Which properties are safe to push on price?
* Which properties need operational fixes before revenue optimization?
* Which issues keep repeating across the portfolio?

A useful system should shorten the path from guest feedback to property-manager action.

---

## 2. First-principles framing

A guest review is not just text. It is operational telemetry.

When a guest writes:

> “The apartment was beautiful and close to everything, but parking was confusing and the bathroom stayed damp after showers.”

A shallow system says:

```text
Sentiment: mixed
```

A useful system says:

```text
Positive signals:
- Design and location are strong.

Negative signals:
- Parking instructions are unclear.
- Bathroom ventilation may be weak.

Recommended actions:
- Add photo-based parking instructions to the pre-arrival guide.
- Add vehicle-size guidance if parking is tight.
- Ask maintenance to inspect bathroom ventilation.
- If bathroom dampness repeats, update the cleaning/inspection checklist.
```

This is the fundamental transformation:

```text
Review text → aspect signals → diagnosis → ranked action → manager decision → outcome learning
```

The product rule:

> **If an insight does not help a property manager make a decision, it is noise.**

---

## 3. What the current dataset gives us

The current dataset is enough to build a meaningful prototype. It contains:

* Properties/listings
* Property descriptions
* Property types
* Cities, regions, neighborhoods, latitude/longitude
* Amenities
* Bedrooms, bathrooms, max guests
* Average nightly rate
* Reviews
* Review text split into liked, disliked, and additional comments
* Overall and subcategory ratings
* Source platform
* Check-in and check-out dates
* Number of nights
* Guest count
* Response status

This is enough for the first version of:

* Review aspect extraction
* Property-level issue summaries
* Property-level strength summaries
* Amenity–aspect promise gap detection
* Rating-text discrepancy detection
* Evidence-backed recommendation cards
* Basic platform/property/rating analysis

However, the dataset should be treated as a **starting point**, not the final production universe.

In production, we should also request:

* Property manager ID
* Owner ID
* Cleaner/vendor ID
* Cleaning logs
* Maintenance logs
* Guest messages during the stay
* Actual price paid
* Cleaning fee
* Discounts
* Occupancy history
* Cancellation/refund history
* Listing photo versions
* Listing description versions
* Whether recommended actions were completed
* Cost of completed actions
* Post-action review outcomes

These extra fields unlock the later learning loop.

---

## 4. Product goal

The goal is to build a system that helps property managers answer six questions.

### 1. What do guests love?

Examples:

* Location
* Cleanliness
* Host responsiveness
* Family suitability
* Kitchen/laundry usefulness
* Wi-Fi/work readiness
* Outdoor space
* Listing accuracy

These strengths should feed listing copy, marketing, owner reports, and pricing confidence.

### 2. What do guests complain about?

Examples:

* Parking confusion
* Bathroom ventilation
* Noise
* Weak Wi-Fi
* Insufficient lighting
* Missing supplies
* Check-in confusion
* Listing mismatch

These issues should feed maintenance tickets, operating checklists, house manual updates, listing rewrites, and guest messaging.

### 3. What is operationally fixable?

Some issues can be fixed quickly:

* Add parking photos
* Add check-in screenshots
* Add more coffee pods
* Improve lighting
* Replace router
* Add towels
* Update house manual

Some issues cannot be fully fixed but can be managed:

* Street noise
* Small room size
* Distance from city center
* Tight parking
* Lack of elevator

For these, the system should recommend expectation-setting, listing copy changes, or pricing adjustments.

### 4. Which issues are high priority?

Priority should depend on:

```text
severity × frequency × recency × rating impact × actionability
```

A rare but serious safety issue should outrank a frequent minor complaint.

A recurring medium-severity issue with an easy fix should also rank highly.

### 5. What should the property manager do next?

The system should output actions, not just analysis.

Examples:

* “Add photo-based parking instructions to the pre-arrival guide.”
* “Inspect bathroom ventilation and update the cleaning checklist.”
* “Rewrite listing copy to mention possible morning street noise.”
* “Do not increase price until recurring Wi-Fi complaints are resolved.”
* “Use guest praise about family suitability in the listing headline.”

### 6. Did the action work?

This is the long-term differentiator.

The system should eventually learn:

* Which recommendations managers accept
* Which recommendations managers reject
* Which interventions reduce future complaints
* Which issues keep returning despite action
* Which actions improve ratings or review sentiment
* Which recommendation types are high-impact for each property type

---

## 5. The three-layer system

The system has three layers. These should be explicit in the architecture and roadmap.

```text
Layer 1: Quantitative layer
Layer 2: Interpretative LLM layer
Layer 3: Feedback-based learning loop
```

Each layer has a different job.

---

## 6. Layer 1 — Quantitative layer

The quantitative layer uses structured data and statistics.

It should not try to “understand” language directly. It should analyse structured fields and structured signals produced by the LLM layer.

### Inputs

* Property attributes
* Amenities
* City, region, neighborhood
* Property type
* Bedrooms, bathrooms, max guests
* Average nightly rate
* Platform
* Stay length
* Guest count
* Overall rating
* Subratings: cleanliness, accuracy, communication, location, check-in, value
* Extracted review aspects
* Manager decisions from the feedback loop

### Questions it should answer

* Which properties have the lowest ratings?
* Which properties have the most recurring issues?
* Which platforms produce lower ratings?
* Do longer stays surface more complaints?
* Are high-occupancy stays more likely to mention comfort or cleanliness issues?
* Are certain amenities associated with positive or negative sentiment?
* Which issue categories are associated with lower overall ratings?
* Which properties have strong ratings but hidden negative text?
* Which properties appear ready for pricing upside?
* Which properties need operational fixes before price increases?

### Early v1 outputs

In v1, the quantitative layer should stay simple:

* Counts
* Averages
* Frequencies
* Cross-tabs
* Property-level summaries
* Platform-level summaries
* Rating-text discrepancy
* Issue frequency by property
* Issue frequency by platform
* Issue severity by property
* Amenity–aspect promise gap table

### Later outputs

In later phases, the quantitative layer can support:

* Rating driver models
* Negative review risk prediction
* Property health score
* Pricing-readiness score
* Recommendation ranking model
* Before/after intervention tracking
* Matched comparison groups
* Outcome-based learning

### Important language discipline

Early versions should use cautious language.

Allowed:

* “associated with”
* “appears linked to”
* “suggests”
* “correlated with”
* “is mentioned in N reviews”
* “this is a hypothesis”

Avoid in v1:

* “causes”
* “proves”
* “will increase revenue by”
* “guarantees rating uplift”
* “ROI is $X”

Causality and ROI should only come after intervention and revenue data exist.

---

## 7. Layer 2 — Interpretative LLM layer

The LLM layer understands the review text.

Its job is to convert unstructured review language into structured operational meaning.

### Inputs

* `whatGuestLiked`
* `whatGuestDidntLike`
* `additionalComments`
* `fullReview`
* Property description
* Amenities
* Property type
* Guest count and stay length
* Ratings

### Core task: aspect extraction

Each review can contain multiple aspects.

Example:

> “The location was perfect and the host replied quickly, but the bathroom fan was loud and the Wi-Fi slowed down at night.”

The LLM should output:

| Aspect               | Polarity | Severity | Action                     |
| -------------------- | -------- | -------: | -------------------------- |
| Location             | Positive |      Low | Use in listing positioning |
| Host communication   | Positive |      Low | Reinforce as strength      |
| Bathroom ventilation | Negative |   Medium | Maintenance inspection     |
| Wi-Fi performance    | Negative |   Medium | Test router under load     |

### Required fields per aspect

```json
{
  "review_id": "rev_001",
  "property_id": "prop_abc",
  "aspect_category": "parking",
  "aspect_subcategory": "parking_instructions",
  "polarity": "negative",
  "severity": "medium",
  "actionability": "high",
  "evidence_snippet": "Parking was tighter than expected",
  "evidence_field": "whatGuestDidntLike",
  "action_category": "guest_communication",
  "suggested_action": "Add parking photos and vehicle-size guidance to the pre-arrival message",
  "confidence": "high",
  "extractor_version": "v0.1"
}
```

### What the LLM must do well

The LLM must distinguish between similar but operationally different complaints.

For example, parking complaints can mean:

* Parking does not exist
* Parking exists but is hard to find
* Parking is tight for large vehicles
* Parking was not explained clearly
* Parking costs extra
* Parking feels unsafe

Each requires a different fix.

Similarly, Wi-Fi complaints can mean:

* No Wi-Fi
* Slow Wi-Fi
* Wi-Fi drops out
* Wi-Fi works but not with many devices
* Wi-Fi is fine for leisure but not remote work

The LLM should extract this nuance.

---

## 8. Layer 3 — Feedback-based learning loop

The feedback loop is what turns the system from a one-time analysis tool into a learning product.

The system should not simply produce recommendations. It should observe what happens after recommendations.

### Basic loop

```text
New review arrives
→ Review understanding agent extracts aspects
→ Quantitative layer updates property signals
→ Recommendation synthesizer creates action cards
→ Manager accepts / rejects / defers
→ Action is completed or ignored
→ Future reviews are monitored
→ System learns from manager decisions and outcomes
```

### v1 feedback loop

The first version can be simple:

* Capture Accept / Reject / Defer
* Capture optional free-text reason
* Suppress repeated rejected recommendations for the same property
* Track whether accepted actions were marked complete
* Track whether the same complaint appears again later

### v2 feedback loop

Later, the system can learn:

* Which recommendation types are accepted most often
* Which recommendations are rejected by each property manager
* Which action categories tend to reduce repeat complaints
* Which issues are hard to resolve
* Which property types respond well to which interventions

### v3 feedback loop

Eventually, the system can support stronger outcome learning:

* Before/after review comparison
* Matched control properties
* Rating lift estimates
* Complaint reduction estimates
* Recommendation re-ranking based on observed outcomes

This is the moat.

Most systems can summarize reviews. Fewer can tell a property manager what to do. Even fewer can learn which actions actually work.

---

## 9. Core aspect taxonomy

Start with a practical taxonomy. It should be simple enough for v1 but flexible enough to expand.

### 1. Cleanliness

Sub-aspects:

* Bathroom cleanliness
* Kitchen cleanliness
* Linen and towels
* Dust
* Smell or odor
* Mold or dampness
* Pool cleanliness
* General hygiene

Typical actions:

* Update cleaning checklist
* Retrain cleaner
* Add inspection step
* Escalate recurring issue

### 2. Comfort and sleep quality

Sub-aspects:

* Bed comfort
* Mattress/pillows
* Temperature
* Noise
* Curtains/light
* Air circulation
* Privacy

Typical actions:

* Replace pillows/mattress
* Add blackout curtains
* Add fan/heater
* Disclose noise
* Provide earplugs

### 3. Check-in and check-out

Sub-aspects:

* Instructions clarity
* Lockbox/smart lock
* Key handover
* Timing
* Entry confusion
* Checkout instructions

Typical actions:

* Improve check-in guide
* Add photos/screenshots
* Send instructions earlier
* Add FAQ to house manual

### 4. Host communication

Sub-aspects:

* Responsiveness
* Helpfulness
* Clarity
* Tone
* Issue resolution
* Language mismatch

Typical actions:

* Improve response templates
* Add AI concierge support
* Add escalation rules
* Improve house guide

### 5. Listing accuracy

Sub-aspects:

* Photos vs reality
* Amenity mismatch
* Size mismatch
* Location expectation mismatch
* Quietness mismatch
* View/privacy mismatch

Typical actions:

* Rewrite listing copy
* Add caveat
* Update photos
* Remove overclaim
* Add expectation-setting message

### 6. Amenities

Sub-aspects:

* Wi-Fi
* AC/heating
* Kitchen
* Laundry
* Parking
* Pool
* Workspace
* Lighting
* Coffee/toiletries/supplies

Typical actions:

* Repair or upgrade amenity
* Add missing item
* Clarify amenity limitations
* Improve photos and listing placement

### 7. Location and access

Sub-aspects:

* Walkability
* Transit
* Restaurants/cafes
* Beach/accessibility
* Street noise
* Safety
* Parking/access route

Typical actions:

* Reposition listing
* Add local guide
* Add transport instructions
* Disclose unavoidable downsides

### 8. Maintenance

Sub-aspects:

* Broken appliances
* Plumbing
* Bathroom fan
* AC
* Hot water
* Locks
* Lighting
* Wi-Fi equipment

Typical actions:

* Create maintenance ticket
* Escalate recurring failure
* Add inspection before arrival

### 9. Value for money

Sub-aspects:

* Price too high
* Cleaning fee concerns
* Better/worse than expected
* Value relative to location
* Value relative to amenities

Typical actions:

* Review pricing
* Fix issues before price increase
* Improve listing positioning
* Add low-cost value boosters

### 10. Safety and risk

Sub-aspects:

* Locks
* Unsafe neighborhood
* Smoke detectors
* Electrical issues
* Slippery stairs
* Pool safety
* Privacy concerns

Typical actions:

* Immediate escalation
* Safety inspection
* Owner notification
* Human review before public response

---

## 10. The central data object: review-aspect records

The review-aspect table is the core product object.

It is the bridge between:

* Raw review text
* LLM understanding
* Quantitative analysis
* Recommendation generation
* Feedback learning

### Why this table matters

Without it, we only have raw text and ratings.

With it, we can ask:

* Which properties have recurring parking complaints?
* Which amenities are underperforming?
* Which issues are associated with lower ratings?
* Which issue types are high severity but low frequency?
* Which complaints remain after an action was taken?
* Which properties have hidden dissatisfaction despite high ratings?

### Suggested schema

| Field                | Purpose                                       |
| -------------------- | --------------------------------------------- |
| `review_id`          | Link back to source review                    |
| `property_id`        | Link to listing/property                      |
| `source_platform`    | Analyse channel differences                   |
| `review_date`        | Track trends over time                        |
| `aspect_category`    | Main issue/strength bucket                    |
| `aspect_subcategory` | Specific issue                                |
| `polarity`           | Positive / neutral / negative                 |
| `severity`           | Low / medium / high / urgent                  |
| `actionability`      | Low / medium / high                           |
| `specificity`        | Vague vs specific                             |
| `evidence_snippet`   | Quote from review                             |
| `evidence_field`     | liked / disliked / comments / fullReview      |
| `action_category`    | Ops / maintenance / listing / pricing / comms |
| `suggested_action`   | Initial action idea                           |
| `confidence`         | Low / medium / high                           |
| `extractor_version`  | Track prompt/model version                    |

---

## 11. Amenity–Aspect Promise Gap

This is one of the strongest original ideas and should remain central.

The question is:

> **Does the property deliver what the listing promises?**

Compare:

```text
Listing description + amenities
vs
Review aspects + complaints
```

### Why this matters

Bad reviews often come from expectation mismatch, not just poor property quality.

A property may be acceptable, but if the listing overpromises, guests feel disappointed.

### Edge types

| Type                 | Meaning                                         | Example                                           | Action                              |
| -------------------- | ----------------------------------------------- | ------------------------------------------------- | ----------------------------------- |
| Promise fulfilled    | Amenity listed and praised                      | “Wi-Fi was stable” for a listing with Fast Wi-Fi  | Use in marketing                    |
| Promise gap          | Amenity listed but criticized                   | “Wi-Fi slowed down” for a listing with Fast Wi-Fi | Inspect/repair or soften claim      |
| Instruction gap      | Amenity exists but guest struggled to use it    | “Parking was confusing”                           | Improve guide                       |
| Latent demand        | Amenity not listed but repeatedly requested     | Guests ask for workspace                          | Consider capex or repositioning     |
| Silent amenity       | Amenity listed but never mentioned              | BBQ listed but no praise or usage                 | Improve photos/copy or deprioritize |
| Expectation mismatch | Listing suggests one thing, reviews say another | “Quiet retreat” vs street noise                   | Rewrite listing copy                |

### Examples

| Listing promise | Review signal                 | Interpretation                 | Recommendation                            |
| --------------- | ----------------------------- | ------------------------------ | ----------------------------------------- |
| Fast Wi-Fi      | Wi-Fi slows with many devices | Promise gap                    | Test router, add mesh, adjust claim       |
| Free parking    | Parking is tight/confusing    | Instruction or expectation gap | Add parking photos and vehicle-size notes |
| Quiet stay      | Morning street noise          | Expectation mismatch           | Add caveat, provide earplugs              |
| Work-friendly   | Poor task lighting            | Amenity quality gap            | Add desk lamp/floor lamp                  |
| Full kitchen    | Missing utensils              | Amenity completeness gap       | Add kitchen inventory checklist           |

In v1, this can be built with simple joins and rules. It does not require a graph database.

---

## 12. Rating-text discrepancy

This is another high-value original idea.

Numeric ratings and review text often disagree.

A guest may leave 5 stars but include a real complaint. Another guest may leave a low rating while the written text sounds mild.

The system should compute:

```text
numeric_rating_score
text_sentiment_score
rating_text_gap
```

### Four cases

| Rating | Text           | Interpretation                           | Action                            |
| ------ | -------------- | ---------------------------------------- | --------------------------------- |
| High   | Positive       | True positive                            | Use as strength                   |
| High   | Negative issue | Courtesy rating / hidden dissatisfaction | Surface issue before rating drops |
| Low    | Positive/mild  | Single issue or unusual expectation      | Investigate dominant issue        |
| Low    | Negative       | True negative                            | Prioritize action and response    |

The hidden dissatisfaction case is especially important.

A property manager looking only at ratings may miss early warning signs. The LLM should surface recurring complaints even when ratings remain high.

---

## 13. Recommendation cards: the trust deliverable

The property manager should not first see raw aspect tables. They should see ranked recommendation cards.

Each recommendation should be:

* Specific
* Actionable
* Evidence-backed
* Low on jargon
* Clear about confidence
* Clear about caveats
* Easy to accept, reject, or defer

### Card structure

```text
Action
Why this matters
Evidence from reviews
Affected property/properties
Aspect category
Number of supporting reviews
Severity
Cost level: Low / Medium / High
Impact level: Low / Medium / High
Confidence: Low / Medium / High
Suggested owner: Ops / Maintenance / Listing / Comms / Pricing / Owner
Counterargument or caveat
Decision: Accept / Reject / Defer
```

### Example card

```text
Action:
Add a photo-based parking guide to the pre-arrival message.

Why this matters:
Guests repeatedly mention that parking is tight or unclear. This appears to be an instruction and expectation-setting issue rather than a complete lack of parking.

Evidence:
- “Parking was tighter than expected.”
- “Clearer guidance on the best spot to use would help future guests.”

Affected:
Adelaide Garden Cottage
Aspect: Parking and access
Supporting reviews: 5

Cost:
Low

Impact:
Medium

Confidence:
High

Suggested owner:
Guest communication / operations

Caveat:
This will mostly help guests arriving by car. If most guests do not drive, the portfolio-level impact may be limited.

Decision:
Accept / Reject / Defer
```

This card format is the manager-facing product surface.

---

## 14. Action categories and examples

Every recommendation should map to a real action owner.

### Operations

* Update cleaning checklist
* Add restocking checklist
* Improve turnover inspection
* Add parking instructions
* Add check-in screenshots
* Add house manual FAQ

### Maintenance

* Inspect bathroom ventilation
* Replace router
* Fix AC/heating
* Check plumbing
* Repair smart lock
* Replace broken lighting

### Listing copy and accuracy

* Add noise disclosure
* Clarify parking limitations
* Update Wi-Fi claim
* Remove misleading wording
* Add accurate amenity details
* Reposition property for the right guest segment

### Guest communication

* Send pre-arrival parking guide
* Send check-in instructions earlier
* Add Wi-Fi troubleshooting note
* Add long-stay supply expectations
* Add local guide recommendations

### Pricing and revenue readiness

* Hold price increases until recurring issues are fixed
* Increase price only after value sentiment improves
* Use strong positive aspects to support premium positioning
* Compare value complaints against actual price paid when available

### Owner approval / capex

* Add task lighting
* Install better curtains
* Upgrade Wi-Fi hardware
* Replace mattress
* Improve soundproofing
* Add workspace
* Upgrade bathroom ventilation

### Risk / escalation

* Safety issue detected
* Lock/security complaint
* Electrical or fire risk
* Privacy concern
* Severe cleanliness/pest complaint
* Legal or council/noise complaint

These should bypass normal ranking and go to immediate human review.

---

## 15. Agent topology

Do not start with a large agent zoo. Start simple, but keep the architecture expandable.

### v1 agents/components

#### 1. Ingestion and validation component

Deterministic. No LLM required.

Responsibilities:

* Load JSON/CSV/database records
* Validate schema
* Check duplicate IDs
* Check every review maps to a valid property
* Normalize dates and platforms
* Normalize amenities
* Hash guest emails
* Remove unnecessary personal data before LLM calls
* Produce data-quality report

Output:

* Clean property table
* Clean review table
* Validation report

#### 2. Review understanding agent

LLM-based.

Responsibilities:

* Read review text fields
* Extract aspect records
* Extract evidence snippets
* Assign polarity, severity, actionability, specificity, confidence
* Suggest initial action category
* Avoid unsupported claims

Output:

* Review-aspect table

#### 3. Quantitative analytics component

Deterministic/statistical.

Responsibilities:

* Aggregate aspects by property
* Aggregate aspects by platform
* Compare aspects with ratings
* Compute rating-text discrepancy
* Build amenity–aspect promise gap table
* Detect recurring issues
* Detect property strengths

Output:

* Property-aspect summary
* Platform summary
* Promise-gap table
* Rating-text discrepancy table

#### 4. Recommendation synthesizer agent

LLM-based.

Responsibilities:

* Convert structured insights into ranked recommendation cards
* Include evidence quotes
* Include caveats
* Use careful non-causal language
* Assign action owner
* Assign cost/impact/confidence levels

Output:

* Recommendation cards

#### 5. Feedback and learning component

Deterministic at first.

Responsibilities:

* Capture Accept / Reject / Defer
* Capture rejection reason
* Track completed actions
* Suppress repeated rejected recommendations
* Track whether issue appears again later

Output:

* Decision log
* Intervention log
* Early learning signals

#### 6. Review response drafter

Optional in v1.

Responsibilities:

* Draft public replies for reviews needing response
* Use property voice
* Acknowledge issue
* Mention action carefully
* Keep human approval required

Output:

* Draft replies, not auto-posted replies

---

## 16. Implementation commitments and operating rules

The previous design left several things too vague for an engineer to implement. v1 should make explicit choices.

---

## 16.1 Model provider and validation stack for v1

For v1, keep the stack simple and direct.

### Recommended v1 stack

```text
LLM calls: Direct Anthropic SDK
Structured validation: Pydantic v2
Storage: Postgres or DuckDB for prototype
Batch processing: Python scripts / lightweight job runner
Prototype UI: Streamlit or simple internal web app
Analytics: pandas + scikit-learn / statsmodels
Evaluation: pytest + saved gold-set fixtures
```

### Why not start with LangChain or LangGraph?

Do not start with a heavy orchestration framework unless the workflow clearly needs it.

The first version is mostly:

```text
Load data
→ call LLM for structured extraction
→ validate JSON
→ aggregate results
→ synthesize recommendation cards
→ capture manager decision
```

This does not require a complex graph framework. Direct SDK calls plus typed schemas will be easier to debug, cheaper to run, and simpler for a small engineering team to control.

LangGraph, Pydantic AI, or a custom agent harness can be introduced later when there are multiple long-running workflows, branching tool calls, retries, human approval queues, and recurring scheduled tasks.

### Structured output rule

Every LLM output must pass schema validation.

For aspect extraction:

```text
LLM response
→ parse JSON
→ validate with Pydantic
→ check evidence snippet is substring of source review
→ reject or retry invalid output
→ write validated aspect record
```

Retry policy:

```text
Attempt 1: normal extraction
Attempt 2: retry with validation error message
Attempt 3: final retry with stricter instruction
If still invalid: drop the extraction, log failure, send to manual review queue
```

### v1 model-routing policy

Use two model classes:

| Task                     | Model class                      | Reason                              |
| ------------------------ | -------------------------------- | ----------------------------------- |
| Aspect extraction        | cheaper Sonnet/Haiku-class model | high-volume, structured, repetitive |
| Recommendation synthesis | stronger Sonnet-class model      | needs better judgment and wording   |
| Review response drafting | Sonnet-class model               | tone-sensitive                      |
| Manager Q&A, later       | strongest available model        | reasoning-heavy and interactive     |

Do not fine-tune in v1. First collect enough labeled examples and failure cases.

---

## 16.2 Evaluation plan: buildable test specification

Evaluation cannot be vague. v1 needs a concrete test plan.

### Gold set size

Create a hand-labeled gold set of **80 reviews**.

Stratify it so it includes:

| Slice                                        |                            Minimum examples |
| -------------------------------------------- | ------------------------------------------: |
| Positive reviews                             |                                          25 |
| Neutral reviews                              |                                          20 |
| Negative reviews                             |                                          20 |
| Long reviews / multi-aspect reviews          |                                          10 |
| Low-rating but mild-text reviews             |                                           5 |
| High-rating but complaint-containing reviews |                                           5 |
| Reviews from each major platform             |         At least 8 per platform if possible |
| Reviews across property types                | At least 2 per property type where possible |

If 80 is too expensive at the start, label 50 for the first pass and expand to 80 before v1 launch.

### Labeling schema

Each labeled review should include:

* All aspect categories present
* Aspect subcategory
* Polarity
* Severity
* Actionability
* Evidence quote
* Suggested action category

Each review can have multiple labeled aspects.

### Human labeling process

Use two labelers for at least the first 30 reviews.

Measure agreement on:

* Aspect category
* Polarity
* Severity

If labelers disagree heavily, fix the taxonomy before tuning the model.

### Evaluation metrics

Track these metrics separately:

| Metric                                  |     Target for v1 |
| --------------------------------------- | ----------------: |
| Aspect category micro-F1                |            ≥ 0.75 |
| Aspect category macro-F1                |            ≥ 0.65 |
| Polarity accuracy                       |            ≥ 0.85 |
| Severity agreement within one level     |            ≥ 0.85 |
| Evidence snippet validity               |           ≥ 0.995 |
| Pydantic schema pass rate after retries |           ≥ 0.995 |
| Hallucinated aspect rate                | ≤ 0.02 per review |
| Recommendation card evidence coverage   |              100% |
| Statistical-claim lint pass rate        |              100% |

Macro-F1 matters because rare but important categories like safety should not be hidden by common categories like location or cleanliness.

### Recommendation quality evaluation

For the first property-manager pilot, evaluate the top recommendation cards manually.

Target:

| Metric                                                |           Target |
| ----------------------------------------------------- | ---------------: |
| Manager says recommendation is understandable         |            ≥ 90% |
| Manager says recommendation is relevant               |            ≥ 70% |
| Manager accepts or seriously considers recommendation |            ≥ 40% |
| Manager flags recommendation as hallucinated          | 0 critical cases |
| Manager says evidence is convincing                   |            ≥ 80% |

### Regression tests

Keep a fixed set of 20 reviews as regression fixtures.

Every prompt/model change must be tested against these fixtures before release. A change should not ship if it causes:

* New hallucinated aspects
* Evidence snippets that do not appear in source text
* Major category drift without explanation
* More than 10% drop in aspect F1 on the gold set

---

## 16.3 Cold-start and minimum-evidence rules

A property with 3 reviews should not be treated like a property with 60 reviews.

The system needs evidence thresholds before surfacing recommendations.

### Property evidence tiers

| Tier                    |  Review count | System behavior                                             |
| ----------------------- | ------------: | ----------------------------------------------------------- |
| Tier 0: New property    |     0 reviews | Use listing audit + portfolio priors only                   |
| Tier 1: Sparse          |   1–4 reviews | Surface only high-severity or obvious single-review actions |
| Tier 2: Emerging signal |  5–14 reviews | Surface recommendations with caution language               |
| Tier 3: Stable signal   | 15–39 reviews | Normal recommendation flow                                  |
| Tier 4: Strong signal   |   40+ reviews | Support stronger property-specific conclusions              |

### Minimum evidence threshold

Default rule:

```text
A normal recommendation needs at least 3 supporting aspect mentions.
```

Exceptions:

| Case                                     |                              Threshold |
| ---------------------------------------- | -------------------------------------: |
| Safety/security issue                    |                              1 mention |
| Severe hygiene/pest issue                |                              1 mention |
| Legal/compliance risk                    |                              1 mention |
| Listing promise gap with clear evidence  |                             2 mentions |
| Low-cost guest communication improvement |                             2 mentions |
| Marketing use of positive strength       |                    3 positive mentions |
| Pricing-readiness signal                 | 10+ reviews or portfolio-level support |

### Sparse-property behavior

For properties with fewer than 5 reviews, use language like:

```text
Early signal — based on limited review evidence.
```

Do not rank sparse-property recommendations above stable-property recommendations unless severity is high.

### Portfolio priors for cold start

When a property has too few reviews, use portfolio-level patterns carefully.

Example:

```text
This new city apartment has parking listed. Across similar urban properties, parking confusion often becomes a guest-friction point. Consider adding parking photos before the first stay.
```

This is not a review-backed claim for that property. It is a preventive recommendation based on similar-property patterns.

The card should clearly label it:

```text
Evidence type: portfolio prior, not property-specific review evidence.
```

---

## 16.4 Time-decay and recency weighting

Old complaints should not count the same as recent complaints.

A complaint from yesterday is more informative than a complaint from eight months ago, especially if the issue may have been fixed.

### Recency weight

Use a simple exponential decay for aspect aggregation:

```text
recency_weight = exp(-age_days / half_life_days)
```

Recommended default:

```text
half_life_days = 90
```

Meaning:

* A complaint today has weight 1.00
* A complaint 90 days old has weight about 0.50
* A complaint 180 days old has weight about 0.25
* A complaint 270 days old has weight about 0.125

### Severity override

High-severity issues decay more slowly.

Suggested half-lives:

| Severity      |                              Half-life |
| ------------- | -------------------------------------: |
| Low           |                                60 days |
| Medium        |                                90 days |
| High          |                               180 days |
| Urgent/safety | Does not decay until manually resolved |

### Action-resolution override

If an accepted action is marked complete, pre-action complaints should be down-weighted more aggressively for future recommendations.

Example:

```text
Bathroom fan complaints appeared before repair.
Repair marked complete on March 1.
Future aggregation should treat pre-March complaints as historical context, not active evidence.
```

### Active vs historical issue status

Each recurring issue should have a status:

| Status     | Meaning                                                          |
| ---------- | ---------------------------------------------------------------- |
| Active     | Recent evidence or unresolved action                             |
| Monitoring | Action completed; watching future reviews                        |
| Historical | Old issue with no recent recurrence                              |
| Resolved   | Action completed and no recurrence after sufficient reviews/time |

Recommendation cards should distinguish:

```text
Active issue: needs action now.
Historical issue: monitor, do not re-recommend unless it recurs.
```

---

## 16.5 New managers and new properties

The learning loop has a cold-start problem. New managers and new properties have no decision history and no outcome data.

The system should bootstrap intelligently.

### New manager bootstrap

For a new manager, start with:

* Global default ranking rules
* Portfolio-level issue patterns
* Conservative recommendation thresholds
* Low-cost, high-actionability recommendations first
* Human feedback capture from day one

The system should avoid pretending it already knows the manager's preferences.

Recommendation cards should include:

```text
Why this is shown: default ranking, no manager-specific preference history yet.
```

As the manager accepts/rejects recommendations, build a simple preference profile:

* Preferred action types
* Rejected action types
* Cost sensitivity
* Owner-approval bottlenecks
* Properties where certain actions are not feasible
* Tone preference for review replies

### New property bootstrap

For a new property with no reviews, use non-review signals:

* Listing description
* Amenities
* Property type
* Location
* Similar properties in the portfolio
* Known issue patterns for that property type

Useful first-day outputs:

* Listing accuracy audit
* Amenity completeness check
* Pre-arrival guide checklist
* Photo/content checklist
* Risk checklist based on similar properties

Example:

```text
New property checklist:
- If parking is listed, add exact parking instructions and photos.
- If Wi-Fi/workspace is listed, verify speed and add a workspace photo.
- If property is described as quiet, confirm whether reviews/local context support that claim.
- If family-friendly is claimed, verify cot, kitchen basics, laundry, and safety notes.
```

### New property evidence labeling

Cards for new properties should use explicit evidence labels:

| Evidence label            | Meaning                                |
| ------------------------- | -------------------------------------- |
| Property review evidence  | Based on this property's reviews       |
| Portfolio prior           | Based on similar properties            |
| Listing audit             | Based on listing/amenity mismatch risk |
| Operational best practice | General STR operating recommendation   |

Do not mix these labels. A manager should know why the recommendation exists.

### Bootstrap-to-personalization transition

As data accumulates:

```text
0 reviews: listing audit + portfolio priors
1–4 reviews: early signals + urgent issues only
5–14 reviews: cautious property-level recommendations
15+ reviews: normal property-level recommendations
40+ reviews: stronger trend and segmentation analysis
```

---

## 17. Phased roadmap

The roadmap should build capability in layers, not all at once.

---

# Phase 0 — Data foundation and evaluation setup

## Goal

Make the data reliable and create an evaluation baseline before building agents.

## Why this phase matters

If ingestion is wrong, every insight downstream is wrong. The earlier roadmap had inconsistent property/review counts, which shows why validation must come first.

## Build

### Data validation

* Load the dataset
* Count properties and reviews
* Validate property IDs
* Validate review IDs
* Check duplicate review IDs
* Check missing fields
* Check review-to-property foreign keys
* Normalize platform names
* Normalize amenities
* Normalize dates
* Identify PII fields

### PII handling

* Hash guest emails
* Avoid sending guest emails to LLMs
* Use first name only for response drafting if needed
* Keep aspect extraction independent of guest identity

### Gold set creation

Hand-label **80 reviews** for the v1 gold set, with a minimum fallback of 50 for the first internal pass.

Stratify the gold set across:

* Sentiment classes
* Platforms
* Property types
* Short and long reviews
* High-rating reviews with complaint text
* Low-rating reviews with mild or positive text
* Multi-aspect reviews

Each label should include:

* Aspect category
* Aspect subcategory
* Polarity
* Severity
* Actionability
* Evidence snippet
* Suggested action category

## Outputs

* Data-quality report
* Clean property table
* Clean review table
* Initial labeled review set
* Aspect taxonomy v0

## Success criteria

* Data loads consistently
* Every review maps to a property
* PII handling is clear
* Gold set is usable for checking extraction quality
* Gold set has at least 80 labeled reviews before customer-facing v1
* Aspect taxonomy has acceptable human agreement before model evaluation begins

---

# Phase 1 — Review understanding MVP

## Goal

Convert review text into structured aspect records.

## Build

### Aspect extraction prompt/schema

The LLM should read:

* What guest liked
* What guest did not like
* Additional comments
* Full review
* Optional property context

It should output structured JSON matching the review-aspect schema.

### Evidence validation

Every extracted aspect must have an evidence snippet that appears in the original review.

No evidence means no claim.

### Severity and actionability

The extractor should classify:

* Low severity: minor improvement, small annoyance
* Medium severity: meaningful guest-experience issue
* High severity: repeated or serious issue affecting satisfaction
* Urgent: safety, security, legal, or severe hygiene issue

Actionability:

* High: easy to fix operationally
* Medium: fixable with some cost or owner approval
* Low: structural issue that can mostly be managed through expectations or pricing

## Outputs

* Review-aspect table
* Extraction telemetry
* Failed extraction log
* Evidence-validation failures

## Success criteria

* Aspect extraction is understandable to a property manager
* Aspect category micro-F1 ≥ 0.75 on the gold set
* Aspect category macro-F1 ≥ 0.65 on the gold set
* Polarity accuracy ≥ 0.85
* Severity agreement within one level ≥ 0.85
* Evidence snippet validity ≥ 99.5%
* Schema validation pass rate after retries ≥ 99.5%
* Hallucinated aspect rate ≤ 0.02 per review
* No hallucinated issues reach the manager surface
* The system can extract multiple aspects from one review

---

# Phase 2 — Property diagnosis

## Goal

Turn individual review aspects into property-level understanding.

## Build

### Per-property aggregation

For each property, compute:

* Top positive aspects
* Top negative aspects
* Most frequent issues
* Most severe issues
* Most actionable issues
* Recent issue trend
* Rating distribution
* Rating-text discrepancy flags

### Property health summary

For each property, produce:

* What guests love
* What guests complain about
* What seems operationally fixable
* What seems structural
* What needs immediate attention
* What could be used in marketing

### Issue prioritization

Initial ranking formula:

```text
priority = severity × recency_weighted_frequency × actionability × evidence_strength × promise_gap_boost
```

Use simple scoring at first. Avoid overfitting.

Apply evidence-tier rules:

* Normal property-level recommendation: at least 3 supporting aspect mentions
* Safety/security/severe hygiene: 1 mention is enough
* Listing promise gap: 2 mentions is enough
* Sparse property with 1–4 reviews: only show high-severity or low-cost obvious fixes
* New property with 0 reviews: show listing-audit and portfolio-prior recommendations only

## Outputs

* Property diagnosis table
* Property health summary
* Ranked issue list per property
* Strength list per property

## Success criteria

* A manager can recognize the property from the summary
* The top issues match what a human would notice after reading the reviews
* The system separates strengths from issues clearly

---

# Phase 3 — Amenity–Aspect Promise Gap

## Goal

Compare what the listing promises against what guests experience.

## Build

### Amenity normalization

Normalize amenities into canonical names.

Examples:

* `Fast WiFi`, `Wi-Fi`, `Wifi` → `wifi`
* `Dedicated workspace`, `Workspace` → `workspace`
* `Secure parking`, `Free parking`, `Parking` → `parking`

### Promise-gap rules

Detect:

* Amenity listed and praised
* Amenity listed and criticized
* Amenity missing but repeatedly requested
* Amenity listed but never mentioned
* Listing claim contradicted by review text

### Listing accuracy checks

Compare property description language against review signals.

Examples:

* Description says “quiet” but reviews mention noise
* Description says “work-friendly” but reviews mention poor lighting or weak Wi-Fi
* Description says “family-friendly” but reviews mention missing family supplies
* Description says “parking” but reviews mention tight or confusing parking

## Outputs

* Promise-gap table
* Listing accuracy alerts
* Amenity improvement suggestions
* Listing rewrite candidates

## Success criteria

* The system identifies mismatches that a manager would consider real
* Recommendations are framed as hypotheses unless strongly evidenced
* The output directly supports listing changes or operational fixes

---

# Phase 4 — Recommendation cards

## Goal

Turn analysis into ranked, manager-facing decisions.

## Build

### Recommendation synthesis

The synthesizer should create top recommendations per property.

Each card must include:

* Action
* Why it matters
* Evidence
* Affected property/properties
* Number of supporting reviews
* Cost level
* Impact level
* Confidence
* Suggested owner
* Counterargument/caveat
* Decision field

### Ranking logic

Rank by:

* Severity
* Frequency
* Recency
* Actionability
* Rating association
* Evidence specificity
* Whether the issue is a promise gap
* Whether the action is low-cost

### Claim discipline

The synthesizer must not invent:

* Dollar ROI
* Guaranteed rating lift
* Causal claims
* Unsupported urgency

## Outputs

* Recommendation card table
* Manager-facing card stack
* Top 3 recommendations per property
* Portfolio-level top recommendations

## Success criteria

* Managers understand the recommendation quickly
* Managers trust the evidence
* Managers can accept/reject/defer without needing to read every review
* The recommendations are specific enough to act on

---

# Phase 5 — Review response drafting

## Goal

Help managers respond to reviews, but keep the focus on operational learning.

## Build

### Response prioritization

Prioritize responses for:

* Negative reviews
* Neutral reviews with clear issues
* Public/high-visibility platforms
* Safety or trust issues
* Reviews marked as needing response

### Draft response generation

The response should:

* Thank the guest
* Acknowledge positives
* Acknowledge issue without being defensive
* Mention practical follow-up if appropriate
* Avoid overpromising
* Keep tone professional and human

### Human approval

No auto-posting in v1.

## Outputs

* Draft review replies
* Response priority queue

## Success criteria

* Manager edits are minimal
* Responses sound human
* Responses do not create legal or reputational risk

---

# Phase 6 — Feedback capture

## Goal

Start the learning loop.

## Build

### Decision capture

For each recommendation:

* Accept
* Reject
* Defer
* Reason
* Owner
* Target date
* Completion status

### Suppression rules

If a recommendation is rejected repeatedly for the same property, do not keep showing it.

Example:

```text
Manager rejects “add coffee pods” twice for the same property.
System suppresses similar supply-upgrade recommendations unless the issue becomes high severity.
```

### Basic outcome tracking

Track whether the same aspect appears in future reviews after an action is accepted.

Example:

```text
Action accepted: Add parking guide.
Monitor: Do parking complaints decrease in future reviews?
```

## Outputs

* Decision log
* Intervention log
* Suppression rules
* Early outcome tracking

## Success criteria

* The system remembers manager preferences
* The system avoids repeating rejected advice
* The team can begin measuring whether actions reduce complaints

---

# Phase 7 — Stronger quantitative analytics

## Goal

Use the structured aspect table to connect review themes with ratings and property attributes.

## Build

### Rating driver analysis

Analyse which aspects are associated with lower or higher ratings.

Examples:

* Cleanliness issues vs overall rating
* Parking issues vs check-in/value rating
* Wi-Fi issues vs long-stay reviews
* Communication praise vs overall rating
* Listing accuracy issues vs value rating

### Platform analysis

Compare platforms carefully.

Questions:

* Are some platforms more negative?
* Are platform ratings calibrated differently?
* Are certain issue types more common on certain platforms?

### Stay-length analysis

Questions:

* Do short stays complain more about check-in?
* Do long stays complain more about Wi-Fi, kitchen, laundry, and supplies?
* Do larger groups surface comfort and space issues?

### Property health score

Create a simple property health score from:

* Overall rating
* Recent rating trend
* Negative aspect frequency
* Severity-weighted issue count
* Listing accuracy risk
* Response backlog
* Repeated unresolved issues

## Outputs

* Rating driver report
* Platform comparison
* Stay-length analysis
* Property health score
* Risk flags

## Success criteria

* Quantitative findings support, not replace, manager judgment
* Claims remain descriptive unless intervention data exists
* Property health score matches manager intuition reasonably well

---

# Phase 8 — Closed-loop learning

## Goal

Move from recommendations to evidence of what works.

## Build

### Intervention tracking

For each accepted recommendation, track:

* Action type
* Property
* Date accepted
* Date completed
* Owner
* Cost level or actual cost if known
* Related aspect category
* Future review outcomes

### Before/after monitoring

Example:

```text
Before action:
Parking complaints appeared in 6 reviews over 90 days.

Action:
Added photo-based parking guide.

After action:
Parking complaints appeared in 1 review over next 90 days.
```

This is not yet full causality, but it is useful.

### Recommendation re-ranking

Use manager decisions and observed outcomes to improve future ranking.

Signals:

* Accepted recommendations
* Rejected recommendations
* Deferred recommendations
* Completed actions
* Repeated complaints after action
* Complaint reduction after action

## Outputs

* Intervention outcome report
* Improved recommendation ranking
* Manager preference model
* Property playbooks

## Success criteria

* The system can show which recommendations appear to reduce complaints
* Managers feel recommendations improve over time
* The system becomes more personalized to each property manager

---

# Phase 9 — Advanced modules after the core loop works

Only build these after the basic review → recommendation → feedback loop is trusted.

## 1. Manager Q&A agent

Let managers ask:

* Which properties have recurring parking issues?
* Which properties are not ready for price increases?
* Which listings have accuracy problems?
* Which issues keep recurring after maintenance?
* Which reviews mention family suitability?

## 2. Owner report generator

Generate monthly owner reports:

* Rating trend
* Guest praise
* Guest complaints
* Completed actions
* Recommended owner approvals
* Revenue-readiness notes

## 3. Maintenance and cleaning integration

Connect recommendations to:

* Maintenance tickets
* Cleaning checklists
* Vendor performance
* Turnover quality

## 4. Listing optimization agent

Generate:

* Revised listing copy
* Better amenity descriptions
* Expectation-setting caveats
* Photo brief suggestions
* Segment-specific positioning

## 5. Pre-arrival guest communication

Use review intelligence to prevent future issues.

Examples:

* Send parking guide before arrival
* Send Wi-Fi instructions for long stays
* Send noise/earplug guidance where appropriate
* Send check-in screenshots

## 6. Pricing-readiness module

Use satisfaction data to inform pricing.

Examples:

* Property has strong cleanliness, accuracy, and value sentiment → consider price test
* Property has recurring Wi-Fi complaints → fix before price increase
* Property has location praise but value complaints → review price positioning

## 7. Multimodal listing accuracy

Later, compare:

* Listing photos
* Guest-uploaded photos
* Review complaints
* Listing claims

This can detect misleading photos, missing amenity photos, or outdated listing visuals.

---

## 18. Evaluation and guardrails

The system must be trusted. Trust comes from evidence, evaluation, and restraint.

### Evaluation

Create a gold set of hand-labeled reviews and measure:

* Aspect category accuracy
* Polarity accuracy
* Severity agreement
* Evidence quote validity
* Hallucination rate
* Recommendation usefulness

### Guardrails

* Every manager-facing claim needs an evidence quote.
* Review text must be treated as data, not instructions.
* Guest emails should not be sent to the LLM.
* Avoid using personal guest attributes for service differentiation.
* Public review replies require human approval.
* No causal claims without intervention evidence.
* No dollar ROI without real cost/revenue data.
* Safety issues should escalate immediately.

### Versioning

Track:

* Prompt version
* Model version
* Extractor version
* Extraction timestamp

This matters because if the extraction logic changes, old and new outputs should not be mixed without awareness.

---

## 19. What not to build first

Avoid the following in v1:

* Fully autonomous review replies
* Full causal inference
* Complex knowledge graph infrastructure
* Dollar ROI estimates
* Dynamic pricing automation
* Divestment simulation
* Guest-facing matchmaker
* Large multi-agent debate loops
* Weather and local-event enrichment
* Multimodal photo analysis
* Cross-customer benchmarking

These are not bad ideas. They are just not needed to prove the first version.

The first question is:

> **Can we reliably turn reviews into useful, trusted, evidence-backed actions?**

If yes, build the next layers.

---

## 20. Data enrichment wishlist

To improve later phases, request these data sources.

### Reservation data

* Booking date
* Cancellation status
* Price paid
* Cleaning fee
* Discount
* Guest segment if available
* Repeat guest flag

Unlocks:

* Better value analysis
* Booking-risk analysis
* Repeat-guest insight

### Operations data

* Cleaning logs
* Maintenance tickets
* Issue completion dates
* Cleaner/vendor IDs
* Turnover time

Unlocks:

* Root-cause analysis
* Vendor performance
* Intervention outcome tracking

### Guest communication data

* In-stay messages
* Pre-arrival questions
* Complaint threads
* Resolution time

Unlocks:

* Early issue detection
* Better house manual
* Better AI concierge

### Listing/version data

* Listing description history
* Amenity history
* Photo history
* Platform-specific listing variants

Unlocks:

* Listing accuracy analysis
* Before/after copy testing
* Photo brief generation

### Revenue data

* ADR
* Occupancy
* RevPAR
* Platform commissions
* Direct booking conversion

Unlocks:

* Pricing readiness
* Revenue impact analysis
* Stronger owner reporting

---

## 21. Phased build summary

| Phase   | Focus                    | Main output                               |
| ------- | ------------------------ | ----------------------------------------- |
| Phase 0 | Data foundation          | Clean data + gold set                     |
| Phase 1 | Review understanding     | Review-aspect table                       |
| Phase 2 | Property diagnosis       | Strengths/issues per property             |
| Phase 3 | Promise gap              | Listing/amenity mismatch table            |
| Phase 4 | Recommendation cards     | Ranked manager actions                    |
| Phase 5 | Review response drafting | Human-approved draft replies              |
| Phase 6 | Feedback capture         | Accept/reject/defer learning data         |
| Phase 7 | Quantitative analytics   | Rating drivers + property health          |
| Phase 8 | Closed-loop learning     | Intervention outcome tracking             |
| Phase 9 | Advanced modules         | Q&A, owner reports, pricing, integrations |

---

## 22. Final stakeholder narrative

The clean narrative is:

> SympleHost already helps property managers run their listings. This system adds a review intelligence layer that reads guest feedback, connects it to listing attributes and ratings, identifies what matters, and turns it into concrete actions.

The product does not simply say what guests said.

It answers:

```text
What happened?
Why does it matter?
What should we do?
How confident are we?
What evidence supports it?
Did the action work?
```

That is the roadmap.

Start with a trusted review-to-action loop. Then add stronger analytics, feedback learning, owner reporting, pricing readiness, and operational integrations.

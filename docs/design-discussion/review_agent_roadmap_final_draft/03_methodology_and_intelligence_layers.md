# 03 — Methodology and Intelligence Layers

## 1. Purpose

This file defines the methodology that powers the system.

The system has three layers:

```text
Layer 1 — Quantitative layer
Layer 2 — Interpretative LLM layer
Layer 3 — Feedback-based learning loop
```

All three layers are needed.

- The quantitative layer identifies patterns across structured data.
- The LLM layer understands text and creates structured signals.
- The feedback loop learns from manager decisions and future outcomes.

## 2. Layer 1 — Quantitative layer

### 2.1 Role

The quantitative layer answers:

```text
What patterns are visible across properties, reviews, conversations, ratings, and outcomes?
```

It does not replace the LLM layer. It consumes the structured outputs produced by the LLM layer.

### 2.2 Inputs

- Property attributes
- Amenities
- Listing claims
- Reservation metadata
- Review ratings
- Review aspects
- Conversation issue episodes
- Review–conversation alignments
- Manager decisions
- Operational task completion
- Customer recovery outcomes
- Repeat bookings

### 2.3 Core methods in v1

Use simple, transparent analytics first:

- counts
- averages
- medians
- cross-tabs
- recency-weighted frequencies
- rating distributions
- issue frequency by property
- issue frequency by platform
- time-to-response distributions
- resolution-status distributions
- recommendation acceptance rates

### 2.4 Recency weighting

Old complaints should matter less than recent complaints.

Use exponential decay:

```text
recency_weight = exp(-age_days / half_life_days)
```

Default half-life:

```text
90 days
```

Severity-specific half-life:

| Severity | Half-life |
|---|---:|
| Low | 60 days |
| Medium | 90 days |
| High | 180 days |
| Urgent / safety | Does not decay until manually resolved |

### 2.5 Minimum evidence thresholds

A recommendation should not surface just because one low-quality signal exists.

| Recommendation type | Minimum evidence |
|---|---:|
| Normal property-level action | 3 supporting mentions |
| Listing promise gap | 2 supporting mentions |
| Low-cost guest communication fix | 2 supporting mentions |
| Safety/security/severe hygiene | 1 mention |
| Pricing-readiness signal | 10+ reviews or strong portfolio support |
| Marketing strength claim | 3 positive mentions |
| New property recommendation | listing audit / portfolio prior, clearly labeled |

### 2.6 Evidence tiers

| Tier | Review count | Behavior |
|---|---:|---|
| Tier 0 | 0 reviews | Listing audit + portfolio priors only |
| Tier 1 | 1–4 reviews | Urgent or obvious low-cost fixes only |
| Tier 2 | 5–14 reviews | Cautious recommendations |
| Tier 3 | 15–39 reviews | Normal recommendation flow |
| Tier 4 | 40+ reviews | Stronger trend analysis |

### 2.7 Property health score

A property health score should be simple and interpretable in early versions.

Candidate formula:

```text
property_health_score =
  0.25 * normalized_overall_rating
+ 0.20 * recent_review_sentiment
+ 0.15 * cleanliness_health
+ 0.15 * listing_accuracy_health
+ 0.10 * conversation_resolution_health
+ 0.10 * active_issue_penalty_inverse
+ 0.05 * response_status_health
```

Do not overfit. Treat this as a prioritization aid, not truth.

### 2.8 Conversation quality score

Candidate formula:

```text
conversation_quality_score =
  0.30 * first_response_sla_score
+ 0.25 * resolution_rate
+ 0.20 * sentiment_recovery_score
+ 0.15 * escalation_success_score
+ 0.10 * low_repeat_contact_score
```

### 2.9 Pricing-readiness signal

A property is more price-ready when:

- overall rating is strong
- value rating is strong
- cleanliness and accuracy issues are low
- active complaints are low
- sentiment is stable or improving
- review–conversation alignment does not show unresolved issues
- positive aspects support premium positioning

A property is not price-ready when:

- value complaints are recurring
- issue recurrence is high
- unresolved chat complaints leak into reviews
- listing accuracy gaps exist
- cleanliness or safety complaints exist

### 2.10 Statistical language policy

Allowed in v1:

- “appears”
- “is associated with”
- “is correlated with”
- “is mentioned in”
- “suggests”
- “hypothesis”

Forbidden in v1:

- “causes”
- “proves”
- “will increase revenue”
- “guarantees rating lift”
- “ROI is X dollars”

## 3. Layer 2 — Interpretative LLM layer

### 3.1 Role

The LLM layer answers:

```text
What does the guest mean, operationally?
```

It should translate messy human language into structured records.

### 3.2 Review interpretation

The LLM extracts:

- aspect category
- subcategory
- polarity
- severity
- actionability
- evidence quote
- suggested action category
- confidence

Examples of nuance:

| Text | Correct interpretation |
|---|---|
| “Parking was tight” | Parking may exist but vehicle-size expectation is unclear |
| “Wi-Fi slowed when everyone connected” | Router capacity issue, not total outage |
| “Quiet except morning traffic” | Location strength with noise expectation risk |
| “Nothing major, just more coffee pods” | Low-severity supply improvement |
| “Photos looked better than reality” | Listing accuracy risk |

### 3.3 Conversation interpretation

The LLM extracts from messages:

- complaint
- request
- question
- praise
- urgency
- issue category
- emotional tone
- whether host response addressed the issue
- whether guest confirmed resolution
- whether compensation was mentioned

### 3.4 Alignment interpretation

The LLM and deterministic matching together classify:

- complaint in chat and repeated in review
- complaint in chat but absent from review
- complaint absent in chat but present in review
- unresolved issue that became public
- resolved issue that did not harm review
- hidden dissatisfaction
- potential service recovery success

### 3.5 Prompting discipline

Every extraction prompt should require:

- structured JSON output
- allowed enum values
- evidence snippet
- no speculation beyond evidence
- confidence level
- “unknown” when uncertain
- no PII in output unless required

### 3.6 Validation discipline

Every LLM output must pass:

1. JSON parse
2. Pydantic validation
3. enum validation
4. evidence snippet substring check
5. policy lint
6. retry on failure
7. quarantine if still invalid

## 4. Layer 3 — Feedback-based learning loop

### 4.1 Role

The feedback loop answers:

```text
What happens after the system recommends something?
```

It captures:

- manager decisions
- rejection reasons
- action completion
- guest recovery decisions
- outreach approval
- message sent
- guest reply
- repeat booking
- complaint recurrence

### 4.2 Learning stages

#### v1 — Rule-based learning

- suppress repeatedly rejected recommendation types
- avoid re-suggesting completed historical issues
- track active / monitoring / resolved status
- capture reason tags

#### v2 — Re-ranking

Train a basic ranking model on:

```text
recommendation features → manager decision
```

Features:

- action category
- cost level
- impact level
- evidence count
- severity
- property type
- manager history
- owner constraints
- prior acceptance

#### v3 — Outcome-aware learning

Measure whether accepted recommendations are associated with:

- fewer future complaints
- better rating trend
- shorter resolution time
- higher repeat booking
- lower recovery cost
- better guest response

Use cautious language unless controlled comparisons exist.

## 5. The methodology loop

```text
Raw data
→ extraction
→ validation
→ aggregation
→ diagnosis
→ recommendation
→ human decision
→ action/recovery
→ outcome tracking
→ learning
```

## 6. Cold-start methodology

### New manager

Use:

- global defaults
- portfolio priors
- conservative thresholds
- low-cost actions first
- human feedback capture from day one

### New property

Use:

- listing audit
- amenity completeness check
- pre-arrival guide checklist
- portfolio priors from similar properties
- operational best practices

Label evidence type clearly:

- property review evidence
- property conversation evidence
- portfolio prior
- listing audit
- operational best practice

## 7. Active / monitoring / resolved status

Every issue should have a lifecycle.

| Status | Meaning |
|---|---|
| Active | Current evidence or unresolved action |
| Monitoring | Action completed, waiting for future data |
| Historical | Old issue without recent recurrence |
| Resolved | Action completed and no recurrence after enough time/reviews |

## 8. Recommendation ranking methodology

Initial score:

```text
priority_score =
  severity_weight
* recency_weighted_frequency
* actionability_weight
* evidence_strength
* promise_gap_boost
* conversation_alignment_boost
* customer_recovery_risk_boost
```

Boosts:

| Condition | Effect |
|---|---|
| Chat complaint repeated in review | Increase priority |
| Safety/security issue | Escalate |
| Promise gap | Increase priority |
| Low-cost high-actionability fix | Increase priority |
| Issue resolved and absent from review | Lower priority |
| Old issue marked fixed | Lower priority |
| Sparse property | Add caution / lower confidence |

## 9. Methodology for customer recovery

Customer recovery should be based on:

- review severity
- conversation alignment
- resolution status
- preventability
- guest sentiment
- guest retention potential
- policy constraints
- platform restrictions
- approval rules

Never offer anything in exchange for review modification.

## 10. Methodology for evidence

Every manager-facing claim must include evidence.

Evidence types:

| Evidence type | Example |
|---|---|
| Review quote | “Parking was tighter than expected” |
| Conversation quote | “Where exactly do we park?” |
| Structured metric | “5 supporting reviews in last 90 days” |
| Listing claim | “Secure parking listed as amenity” |
| Manager decision history | “Similar recommendation previously rejected” |
| Outcome data | “No recurrence in 90 days after task completed” |

## 11. Methodology maturity ladder

| Level | Capability |
|---|---|
| Level 0 | Static summaries |
| Level 1 | Aspect extraction |
| Level 2 | Evidence-backed recommendation cards |
| Level 3 | Conversation-review alignment |
| Level 4 | Recovery workflow |
| Level 5 | Feedback capture |
| Level 6 | Outcome learning |
| Level 7 | Predictive risk and pricing readiness |
| Level 8 | Semi-automated operational prevention |

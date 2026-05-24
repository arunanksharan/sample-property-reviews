# 09 — Implementation Backlog and Delivery Plan

## 1. Purpose

This file converts the roadmap into a practical build plan.

The goal is to help engineering, product, and operations teams know what to build, in what order, and what counts as done.

## 2. Recommended team

Minimum pilot team:

- 1 product manager
- 1 backend/data engineer
- 1 full-stack/product engineer
- 1 ML/LLM engineer
- 1 operations/domain expert
- 1 part-time QA/evaluator

## 3. Technical baseline

Recommended v1 choices:

```text
LLM provider: Anthropic SDK directly
Validation: Pydantic v2
Language: Python
Prototype DB: DuckDB
Pilot DB: Postgres
Analytics: pandas, scikit-learn, statsmodels
UI: Streamlit or lightweight React app
Evaluation: pytest fixtures + labeled gold sets
```

Avoid framework complexity until the workflow proves itself.

## 4. Milestone M0 — Data foundation

Duration: 1–2 weeks

Build:

- repository scaffold
- schema definitions
- ingestion scripts
- PII hashing
- review loader
- property loader
- reservation linker
- data-quality report
- basic test suite

Deliverables:

- `properties_clean`
- `reviews_clean`
- `reservations_clean`
- data validation report
- schema validation tests

Definition of done:

- all rows load consistently
- duplicate checks pass
- every review maps to property
- PII fields identified and protected
- data quality report generated automatically

## 5. Milestone M1 — Review extraction

Duration: 2–3 weeks

Build:

- review aspect Pydantic model
- review extraction prompt
- direct SDK call wrapper
- retry logic
- evidence substring validation
- extraction logging
- 80-review gold set
- evaluation script

Deliverables:

- `review_aspects`
- extraction telemetry
- gold-set evaluation report

Definition of done:

- micro-F1 ≥ 0.75
- macro-F1 ≥ 0.65
- evidence validity ≥ 99.5%
- hallucinated aspect rate ≤ 0.02/review
- output schema pass ≥ 99.5% after retries

## 6. Milestone M2 — Review analytics and property diagnosis

Duration: 2 weeks

Build:

- aspect aggregation
- recency weighting
- evidence tiers
- property health summary
- rating-text discrepancy
- promise-gap v0
- active/monitoring/historical/resolved statuses

Deliverables:

- `property_aspect_summary`
- `rating_text_discrepancy`
- `promise_gap_table`
- `property_diagnosis`

Definition of done:

- top issues per property generated
- sparse properties labeled correctly
- old complaints down-weighted
- promise gaps surfaced with evidence
- manager can review property summary

## 7. Milestone M3 — Recommendation cards

Duration: 2–3 weeks

Build:

- recommendation ranking function
- minimum evidence thresholds
- recommendation Pydantic model
- card synthesis prompt
- evidence/citation table
- caveat generation
- manager UI card stack

Deliverables:

- `recommendations`
- recommendation card UI
- decision capture form

Definition of done:

- cards include action, why, evidence, cost, impact, confidence, caveat
- every card has evidence or explicit evidence type
- no causal/ROI overclaiming
- manager can accept/reject/defer

## 8. Milestone M4 — Conversation intelligence

Duration: 3–5 weeks

Build:

- message ingestion format
- conversation data quality report
- message issue extractor
- issue episode builder
- response time calculation
- resolution status classifier
- conversation gold set

Deliverables:

- `messages_clean`
- `message_issues`
- `issue_episodes`
- conversation metrics dashboard

Definition of done:

- message issue F1 ≥ 0.75
- episode grouping F1 ≥ 0.70
- resolution accuracy ≥ 0.75
- first-response/time-to-resolution metrics generated

## 9. Milestone M5 — Review–conversation alignment

Duration: 2–3 weeks

Build:

- aspect-to-episode matching
- alignment classifier
- preventability scoring
- service recovery state
- enhanced recommendation cards

Deliverables:

- `review_conversation_alignment`
- alignment report
- enhanced card UI

Definition of done:

- alignment precision ≥ 0.80
- alignment recall ≥ 0.65
- failed recovery/silent complaint states surfaced
- conversation evidence appears in cards

## 10. Milestone M6 — Customer recovery workflow

Duration: 3–4 weeks

Build:

- offer policy table
- approval matrix
- recovery case schema
- recovery-needed classifier
- strategy selector
- private message drafter
- policy lint
- approval queue

Deliverables:

- `customer_recovery_cases`
- `outreach_messages`
- recovery approval UI
- draft message queue

Definition of done:

- recovery-needed precision ≥ 0.85
- escalation recall ≥ 0.95
- policy violations = 0
- no compensation without approval
- no review-change incentives generated

## 11. Milestone M7 — Outcome tracking

Duration: 2–4 weeks

Build:

- operational task link
- completion status
- issue lifecycle state
- complaint recurrence tracking
- outreach reply tracking
- offer redemption tracking
- repeat booking linkage

Deliverables:

- `operational_tasks`
- `manager_decisions`
- `outcome_records`
- outcome dashboard

Definition of done:

- accepted recommendations create/attach tasks
- completed tasks move issues to monitoring
- future recurrence tracked
- recovery outcomes tracked

## 12. Milestone M8 — Advanced analytics

Duration: 4–8 weeks

Build:

- property health score
- conversation quality score
- rating-driver analysis
- pricing-readiness score
- recommendation acceptance model
- recovery strategy effectiveness report

Deliverables:

- analytics dashboard
- risk score table
- pricing-readiness report
- learning report

Definition of done:

- scores are interpretable
- sparse data caveats shown
- no unsupported causal claims
- managers find outputs useful

## 13. Milestone M9 — Integrations and scale

Duration: ongoing

Build:

- PMS integration
- OTA message integration
- WhatsApp Business integration
- tasking integration
- CRM integration
- owner reports
- manager Q&A
- audit logs
- role permissions

Deliverables:

- production system
- scheduled ingestion
- role-based UI
- integration monitoring

## 14. Sprint-level backlog

### Sprint 1

- repo setup
- schemas
- data loaders
- validation report
- first gold-set labeling instructions

### Sprint 2

- review extractor
- Pydantic validation
- evidence check
- evaluation script

### Sprint 3

- property aggregation
- promise gap
- discrepancy score
- basic property report

### Sprint 4

- recommendation ranking
- card synthesis
- manager card UI
- decision capture

### Sprint 5

- conversation ingestion
- message issue extraction
- message gold set

### Sprint 6

- issue episode builder
- response time metrics
- resolution status

### Sprint 7

- review–conversation alignment
- enhanced cards
- recovery trigger

### Sprint 8

- recovery policy
- recovery case
- draft generation
- approval queue

### Sprint 9

- task linkage
- outcome tracking
- recurrence monitoring

### Sprint 10

- analytics dashboard
- pilot QA
- product polish

## 15. Risk register

| Risk | Mitigation |
|---|---|
| LLM hallucination | evidence validation + schema checks |
| Bad customer outreach | human approval + policy lint |
| Overclaiming causality | language lint + policy |
| Sparse data | evidence tiers and cold-start labels |
| PII leakage | hashing + payload checks |
| Manager distrust | evidence-backed cards |
| Recommending impossible actions | manager feedback and suppression rules |
| Bad extraction drift | regression fixtures and versioning |
| Compensation misuse | approval matrix |
| Too much complexity | phased build, direct SDK first |

## 16. Pilot success definition

A pilot succeeds when:

- property managers trust the evidence
- recommendations are specific enough to act on
- at least 40% of top recommendations are accepted or seriously considered
- conversation alignment changes at least some diagnoses
- customer recovery drafts pass policy and tone review
- accepted actions can be tracked to completion
- future complaints are monitored

## 17. What to demo

Demo flow:

1. Select property.
2. Show top review themes.
3. Show in-stay conversation issue timeline.
4. Show review–conversation alignment.
5. Show recommendation card.
6. Accept recommendation.
7. Create operational task.
8. Open recovery case.
9. Generate private outreach draft.
10. Approve/hold draft.
11. Show outcome tracking placeholder.

## 18. Final build principle

The system should start narrow but complete.

Do not build half of many things.

Build one end-to-end loop:

```text
review + conversation
→ diagnosis
→ recommendation
→ human decision
→ task/recovery
→ outcome tracking
```

Then improve each component.

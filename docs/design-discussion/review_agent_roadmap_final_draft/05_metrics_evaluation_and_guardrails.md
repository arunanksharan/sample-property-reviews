# 05 — Metrics, Evaluation, and Guardrails

## 1. Purpose

This file defines the measurable quality gates for the entire system.

The system has three classes of evaluation:

1. **Extraction quality** — does the AI correctly understand reviews and conversations?
2. **Recommendation quality** — are the actions useful, trusted, and evidence-backed?
3. **Outcome quality** — do actions improve guest experience, operations, recovery, and retention?

## 2. Evaluation datasets

### 2.1 Review gold set

Size:

```text
80 reviews for v1 launch
50 reviews minimum for first internal pass
```

Stratification:

| Slice | Minimum |
|---|---:|
| Positive reviews | 25 |
| Neutral reviews | 20 |
| Negative reviews | 20 |
| High-rating with complaint text | 5 |
| Low-rating with mild text | 5 |
| Multi-aspect reviews | 10 |
| Each major platform | 8 if possible |
| Each major property type | 2 if possible |

Labels:

- aspect category
- subcategory
- polarity
- severity
- actionability
- evidence quote
- suggested action category

### 2.2 Conversation gold set

Size:

```text
50 conversation threads
50 review–conversation pairs
```

Labels:

- message issue category
- message intent
- urgency
- severity
- evidence quote
- episode grouping
- response time
- resolution status
- alignment type
- preventability

### 2.3 Recovery-case gold set

Size:

```text
50 post-review recovery cases
```

Labels:

- recovery needed / not needed
- strategy type
- offer appropriate / not appropriate
- approval level
- escalation needed
- draft tone quality
- policy risk

## 3. Extraction metrics

### 3.1 Review extraction

| Metric | v1 target |
|---|---:|
| Aspect category micro-F1 | ≥ 0.75 |
| Aspect category macro-F1 | ≥ 0.65 |
| Polarity accuracy | ≥ 0.85 |
| Severity within-one-level agreement | ≥ 0.85 |
| Evidence snippet validity | ≥ 99.5% |
| Pydantic schema pass after retries | ≥ 99.5% |
| Hallucinated aspect rate | ≤ 0.02 per review |
| Unsupported manager-facing claim | 0 |

### 3.2 Conversation extraction

| Metric | v1 target |
|---|---:|
| Message issue micro-F1 | ≥ 0.75 |
| Message intent accuracy | ≥ 0.80 |
| Urgency classification accuracy | ≥ 0.80 |
| Evidence snippet validity | ≥ 99.5% |
| Episode grouping pairwise F1 | ≥ 0.70 |
| Resolution-status accuracy | ≥ 0.75 |
| Escalation detection recall | ≥ 0.90 |

### 3.3 Review–conversation alignment

| Metric | v1 target |
|---|---:|
| Alignment precision | ≥ 0.80 |
| Alignment recall | ≥ 0.65 |
| Preventability human agreement | ≥ 0.70 |
| Failed-recovery detection precision | ≥ 0.80 |
| Silent-complaint detection precision | ≥ 0.80 |

## 4. Recommendation quality metrics

| Metric | Target |
|---|---:|
| Manager says card is understandable | ≥ 90% |
| Manager says card is relevant | ≥ 70% |
| Manager accepts or seriously considers | ≥ 40% in pilot |
| Evidence judged convincing | ≥ 80% |
| Manager flags hallucination | 0 critical cases |
| Recommendation duplicated after rejection | < 5% |
| Recommendation missing owner/action | 0 |

## 5. Customer recovery metrics

### 5.1 Classification

| Task | Metric | Target |
|---|---|---:|
| Recovery-needed classification | Precision | ≥ 0.85 |
| Recovery-needed classification | Recall | ≥ 0.70 |
| Escalation detection | Recall | ≥ 0.95 |
| Offer appropriateness | Human agreement | ≥ 0.80 |
| Policy compliance | Critical failures | 0 |

### 5.2 Draft quality

| Metric | Target |
|---|---:|
| Draft factuality | ≥ 0.95 evidence-grounded |
| Draft tone acceptable | ≥ 0.85 |
| Human edit distance | trend below 30% |
| Message asks for review change | 0 |
| Unsupported compensation promise | 0 |
| Legal/safety auto-send | 0 |

### 5.3 Retention outcomes

Track, but do not overclaim causality early:

- guest reply rate
- positive reply rate
- repeat booking rate
- offer redemption rate
- direct booking conversion
- complaint recurrence
- future review sentiment
- unsubscribes / negative replies

## 6. Operational metrics

| Metric | Meaning |
|---|---|
| Time to issue detection | Time from review/message ingestion to aspect/episode |
| Time to recommendation | Time to manager-facing card |
| Time to decision | How quickly manager acts |
| Time to task creation | Operational speed |
| Time to resolution | Fix execution |
| Response SLA breach rate | Guest communication health |
| Review response backlog | Public response workload |
| Recovery case backlog | Private outreach workload |

## 7. Business metrics

### Property performance

- average rating
- rating trend
- subrating trend
- active issues per property
- unresolved issue count
- promise-gap count
- complaint recurrence rate
- property health score

### Manager performance

- recommendation acceptance rate
- task completion rate
- average time to decision
- response SLA
- recovery approval rate
- repeat rejection patterns

### Guest/retention performance

- repeat booking rate
- direct booking rate
- outreach response rate
- offer redemption rate
- service recovery success
- complaint recurrence after recovery

## 8. Guardrails

### 8.1 Evidence guardrails

- Every aspect needs a source quote.
- Every alignment needs review and/or conversation evidence.
- Every recommendation needs evidence or explicit evidence-type label.
- Every recovery message must match documented complaint.
- No source snippet means no manager-facing claim.

### 8.2 Policy guardrails

Blocked content:

- asking guest to change review
- offering compensation for review change
- blaming guest
- blaming cleaner/vendor publicly
- promising refunds without approval
- making legal admissions in sensitive cases
- sending discounts without approval
- auto-sending safety/legal messages

### 8.3 PII guardrails

- hash guest emails
- do not send unnecessary guest identifiers to LLM
- first name only for drafted message, if needed
- no demographic inference
- no sensitive attribute use
- deletion requests must remove derived individual records where required

### 8.4 Prompt-injection guardrails

Guest messages and reviews are untrusted data.

Rules:

- wrap guest text in explicit data delimiters
- instruct model to treat text as content, not instruction
- strip obvious prompt-injection phrases for secondary processing
- validate outputs strictly
- flag outputs referencing hidden/system instructions

### 8.5 Causal/ROI guardrails

Do not say:

- “caused”
- “proved”
- “guaranteed”
- “will increase revenue by X”
- “ROI is X”

Until there is intervention and revenue data.

Use:

- “associated with”
- “appears linked to”
- “suggests”
- “hypothesis”
- “after action, recurrence decreased; not causal without controls”

## 9. Regression testing

Maintain fixed test sets:

| Set | Size | Purpose |
|---|---:|---|
| Review fixtures | 20 | prompt/model regression |
| Conversation fixtures | 20 | issue extraction regression |
| Alignment fixtures | 20 | review-chat matching regression |
| Recovery fixtures | 20 | policy/draft regression |

A prompt/model change must not ship if it:

- increases hallucinations
- reduces F1 by more than 10%
- breaks evidence validation
- causes policy violations
- changes recommendation semantics without explanation

## 10. Dashboards

### Evaluation dashboard

- extraction F1
- schema pass rate
- evidence validity
- hallucination rate
- policy lint pass rate
- drift by category

### Product dashboard

- recommendations created
- acceptance rate
- task completion
- active issues
- resolved issues
- recovery cases
- outreach sent
- repeat bookings

### Risk dashboard

- safety issues
- legal/privacy issues
- severe hygiene
- policy violations
- compensation approvals
- unresolved negative reviews

## 11. Case review process

Weekly review during pilot:

1. sample 10 recommendation cards
2. sample 10 extraction records
3. sample 5 recovery drafts
4. review false positives/negatives
5. update taxonomy/prompt/rules
6. rerun regression tests
7. document changes in model/prompt version log

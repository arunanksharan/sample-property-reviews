# 04 — Agent Workflows and Operating System

## 1. Purpose

This file describes the practical agent/component architecture.

The system should be agentic in behavior, but not overcomplicated in implementation.

Start with deterministic pipelines plus a few well-scoped LLM agents. Do not build an “agent zoo” before the workflow proves value.

## 2. v1 implementation stack

Recommended stack:

```text
LLM calls: Direct Anthropic SDK
Structured validation: Pydantic v2
Storage: Postgres for production / DuckDB for prototype
Batch jobs: Python scripts or lightweight workflow runner
Analytics: pandas + scikit-learn + statsmodels
UI: Streamlit or simple internal web app
Evaluation: pytest + gold-set fixtures
```

Do not begin with LangGraph or LangChain unless the team already has strong internal standards around them. Direct SDK calls plus strict schemas are easier to debug.

## 3. Component map

```text
[Data ingestion]
      ↓
[Review understanding agent]
      ↓
[Conversation issue extractor]
      ↓
[Issue episode builder]
      ↓
[Review–conversation alignment agent]
      ↓
[Quantitative analytics component]
      ↓
[Recommendation synthesizer]
      ↓
[Customer recovery recommender]
      ↓
[Drafting agents]
      ↓
[Human approval + decision capture]
      ↓
[Outcome tracker]
```

## 4. Component 1 — Data ingestion and validation

Type: deterministic.

Responsibilities:

- load property data
- load review data
- load reservation data
- load conversation messages
- normalize IDs
- normalize timestamps
- normalize platforms
- normalize amenities
- hash guest identifiers
- validate foreign keys
- detect duplicates
- produce data-quality report

Hard blocks:

- duplicate primary keys
- review without property
- message without timestamp
- PII in LLM payload
- unknown enum values
- missing reservation linkage for conversation alignment

## 5. Component 2 — Review understanding agent

Type: LLM.

Input:

- review fields
- ratings
- property context
- listing context

Output:

- review aspect records

Rules:

- one review can produce many aspects
- every aspect needs evidence snippet
- no evidence means no claim
- assign severity and actionability
- extract both positive and negative signals
- distinguish operationally different complaints

## 6. Component 3 — Conversation issue extractor

Type: LLM.

Input:

- individual messages or short windows of messages
- sender type
- timestamp
- property/reservation context

Output:

- message issue records

Must identify:

- complaint
- question
- request
- praise
- urgency
- sentiment
- issue category
- whether action is required

## 7. Component 4 — Issue episode builder

Type: mostly deterministic, optional LLM support.

Input:

- message issue records
- chronological thread messages

Output:

- issue episodes

Logic:

1. group messages by reservation, thread, category, and temporal closeness
2. identify first guest complaint
3. identify first host response
4. calculate response time
5. detect escalation
6. detect resolution confirmation
7. assign resolution status
8. capture evidence snippets

Episode grouping example:

```text
18:42 Guest: Wi-Fi not working in bedroom.
18:55 Host: Please restart router; we are checking.
19:30 Guest: Still not working.
20:05 Host: We reset from our side.
20:10 Guest: Works now, thanks.
```

Issue episode:

```text
category = Wi-Fi
response time = 13 min
resolution time = 88 min
status = resolved
sentiment recovery = negative → positive
```

## 8. Component 5 — Review–conversation alignment agent

Type: LLM + deterministic matching.

Input:

- review aspect records
- issue episode records
- reservation/property context

Output:

- alignment records

Alignment types:

- chat complaint repeated in review
- chat complaint absent from review
- review complaint not in chat
- chat resolved positive review
- chat unresolved negative review
- positive chat negative review
- negative chat positive review

The output should answer:

```text
Was this review complaint already visible during the stay?
Did the host respond?
Was it resolved?
Did it still damage the review?
Was it preventable?
```

## 9. Component 6 — Quantitative analytics component

Type: deterministic/statistical.

Outputs:

- property issue summary
- property strength summary
- active issue list
- promise-gap table
- rating-text discrepancy table
- conversation quality score
- property health score
- customer recovery risk table

## 10. Component 7 — Recommendation synthesizer

Type: LLM.

Input:

- property summary
- review aspects
- issue episodes
- alignment records
- quantitative scores
- evidence snippets
- minimum evidence thresholds

Output:

- recommendation cards

Must include:

- action
- why
- evidence
- affected property
- supporting count
- cost level
- impact level
- confidence
- suggested owner
- caveat
- decision options

Claim discipline:

- no causal language in v1
- no fake dollar ROI
- no unsupported urgency
- no recommendation without evidence label

## 11. Component 8 — Customer recovery recommender

Type: rules + LLM.

Input:

- review
- review aspects
- conversation alignment
- severity
- resolution status
- guest retention signals
- offer policy
- approval matrix

Output:

- customer recovery case

Decisions:

- no outreach
- thank-you only
- improvement acknowledgment
- apology
- apology + fix confirmation
- apology + future-stay discount
- escalation to senior manager
- legal/safety review

## 12. Component 9 — Drafting agents

There are two drafting surfaces.

### Public review response drafter

Goal:

- protect brand reputation
- publicly acknowledge feedback
- stay short and careful

### Private post-review outreach drafter

Goal:

- recover guest trust
- repair relationship
- communicate fix
- optionally offer goodwill gesture with approval

Rules:

- never ask for review changes
- never offer compensation for review modification
- no auto-send for sensitive cases
- human approval required for compensation

## 13. Component 10 — Decision and outcome tracker

Type: deterministic.

Tracks:

- recommendation accepted/rejected/deferred
- recovery case approved/rejected/escalated
- operational task created/completed
- message sent
- guest response
- repeat booking
- complaint recurrence
- rating trend

## 14. Main workflows

### Workflow A — New review arrives

```text
Review ingested
→ review aspect extraction
→ rating-text discrepancy
→ check reservation conversations
→ align review with issue episodes
→ update property diagnosis
→ create recommendation cards
→ create recovery case if needed
→ manager reviews cards
→ decisions captured
```

### Workflow B — New conversation message arrives

```text
Message ingested
→ message issue extraction
→ update/open issue episode
→ calculate response SLA
→ escalate if urgent
→ optionally trigger in-stay support workflow
→ later align with review outcome
```

### Workflow C — Manager accepts recommendation

```text
Recommendation accepted
→ create operational task
→ assign owner
→ set target date
→ mark issue as active
→ after task completion, mark issue as monitoring
→ track future recurrence
```

### Workflow D — Customer recovery approved

```text
Recovery case created
→ strategy selected
→ draft generated
→ human approves
→ message sent
→ guest response tracked
→ repeat booking/offer redemption tracked
→ recovery outcome recorded
```

## 15. Human-in-the-loop boundaries

Human approval required for:

- refunds
- discounts
- compensation
- safety/security issues
- legal/privacy complaints
- angry guests
- public review response posting
- listing changes that materially alter promises
- owner-funded capex
- ambiguous high-impact recommendations

## 16. Agent routing

| Task | Model tier |
|---|---|
| Review aspect extraction | cheaper structured model |
| Message issue extraction | cheaper structured model |
| Recommendation synthesis | stronger reasoning/writing model |
| Recovery strategy | stronger model + rules |
| Public/private drafts | stronger tone-sensitive model |
| Manager Q&A | strongest model later |

## 17. Error handling

| Failure | Action |
|---|---|
| LLM output invalid JSON | retry with validation error |
| Pydantic failure | retry once, then quarantine |
| evidence snippet not found | reject extracted claim |
| unsupported recommendation | block from card surface |
| policy violation in draft | block send |
| missing reservation link | do not align review/conversation |
| sparse evidence | label as early signal |

## 18. Deployment path

### Prototype

- local files / DuckDB
- batch processing
- Streamlit UI
- manual uploads

### Pilot

- Postgres
- scheduled batch jobs
- one or two message channels
- manager card UI
- recovery approval queue

### Production

- PMS integration
- OTA/channel integrations
- WhatsApp/email integrations
- tasking integration
- CRM integration
- audit logs
- role-based permissions
- monitoring and alerts

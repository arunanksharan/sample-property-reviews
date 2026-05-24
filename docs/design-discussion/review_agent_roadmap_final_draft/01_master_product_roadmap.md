# 01 — Master Product Roadmap

## 1. Product thesis

SympleHost can build a differentiated AI system by connecting three signals that are usually treated separately:

1. **Listing promise** — what the property description, amenities, photos, and pre-arrival instructions claim.
2. **Guest journey** — what happened before and during the stay through conversations, questions, complaints, host responses, and issue resolution.
3. **Public review outcome** — what the guest later wrote publicly and how they rated the stay.

The opportunity is to build a system that converts those signals into:

- Property actions
- Maintenance tasks
- Cleaning/process changes
- Listing rewrites
- Pre-arrival guide improvements
- Pricing-readiness signals
- Owner reports
- Private customer recovery outreach
- Long-term learning about what interventions work

## 2. Strategic framing

The old framing:

```text
Analyse reviews and extract sentiment.
```

The better framing:

```text
Build an AI decision copilot that helps property managers decide where to spend the next dollar and the next hour.
```

The complete framing:

```text
Build an AI guest-experience operating system that detects issues from reviews and conversations, diagnoses root causes, recommends property and communication actions, helps recover guests, and learns which interventions reduce complaints and improve retention.
```

## 3. The three product tracks

### Track A — Review Intelligence

Reads reviews, ratings, and listing data.

Outputs:

- Review aspects
- Property strengths
- Property complaints
- Rating-text discrepancy
- Amenity–aspect promise gaps
- Property recommendation cards
- Review response drafts

### Track B — Conversation Intelligence

Reads guest conversations from Airbnb, WhatsApp, Booking.com, email, direct chat, support tickets, and internal notes.

Outputs:

- Message-level issues
- Issue episodes
- Response-time metrics
- Resolution-status metrics
- Review–conversation alignment
- Service recovery classification
- Preventability analysis

### Track C — Post-Review Recovery & Retention

Uses review + conversation + operational context to decide whether and how to contact guests privately.

Outputs:

- Customer recovery case
- Outreach/no-outreach decision
- Strategy selection
- Offer recommendation
- Approval requirement
- Draft private message
- Retention outcome tracking

## 4. Unified product loop

```text
1. Understand the property promise
   - listing description
   - amenities
   - pricing
   - photos
   - house manual
   - pre-arrival messages

2. Understand the guest journey
   - reservation details
   - guest questions
   - in-stay complaints
   - host response
   - resolution status
   - escalations

3. Understand the review outcome
   - rating
   - subratings
   - liked text
   - disliked text
   - additional comments
   - public sentiment

4. Align the journey with the review
   - issue raised in chat and repeated in review
   - issue raised in chat but absent from review
   - issue absent in chat but appears in review
   - positive review despite negative chat
   - negative review despite resolved chat

5. Decide actions
   - property fix
   - listing rewrite
   - communication improvement
   - maintenance task
   - pricing caution
   - owner approval
   - customer recovery outreach

6. Capture decisions
   - accept
   - reject
   - defer
   - complete
   - send outreach
   - approve discount
   - deny compensation

7. Learn outcomes
   - complaint recurrence
   - review improvement
   - response-time improvement
   - repeat booking
   - discount redemption
   - manager acceptance
   - recommendation quality
```

## 5. Product modules

### Module 1 — Data Foundation

Goal: create a clean, trusted data base.

Includes:

- Property/listing ingestion
- Review ingestion
- Conversation ingestion
- Reservation linking
- PII handling
- ID normalization
- Timestamp normalization
- Data-quality checks
- Evaluation gold sets

### Module 2 — Review Understanding

Goal: convert review text into structured aspect records.

Includes:

- Aspect extraction
- Polarity detection
- Severity
- Actionability
- Evidence snippets
- Suggested action category
- Rating-text discrepancy

### Module 3 — Conversation Understanding

Goal: convert guest messages into structured issue episodes.

Includes:

- Message issue extraction
- Complaint/request/question/praise detection
- Host response detection
- Resolution status
- Time to first response
- Time to resolution
- Escalation detection
- Guest sentiment recovery

### Module 4 — Review–Conversation Alignment

Goal: compare what happened privately during the stay with what appeared publicly in the review.

Includes:

- Match review aspect to conversation issue episode
- Detect repeated unresolved complaints
- Detect successful service recovery
- Detect silent dissatisfaction
- Detect review-only complaints
- Assign preventability
- Trigger operational and customer-recovery actions

### Module 5 — Property Diagnosis

Goal: understand each property as an operating unit.

Includes:

- Strengths
- Weaknesses
- Active issues
- Historical/resolved issues
- Amenity–aspect promise gaps
- Listing accuracy risks
- Property health score
- Pricing-readiness signal

### Module 6 — Recommendation Cards

Goal: create evidence-backed, manager-facing decisions.

Includes:

- Action
- Why it matters
- Evidence
- Supporting review/conversation snippets
- Cost level
- Impact level
- Confidence
- Caveat/counterargument
- Owner
- Accept/reject/defer

### Module 7 — Customer Recovery

Goal: decide whether to privately contact guests after reviews.

Includes:

- Recovery case creation
- Strategy selection
- Offer recommendation
- Human approval
- Private message drafting
- Send tracking
- Guest response tracking
- Repeat-booking tracking

### Module 8 — Feedback Learning

Goal: make the system improve over time.

Includes:

- Manager decision history
- Completion status
- Complaint recurrence
- Review trend after action
- Guest recovery outcome
- Repeat booking
- Re-ranking of future recommendations

## 6. Phased roadmap

### Phase 0 — Data foundation and baseline evaluation

Build:

- Ingestion for properties, reviews, reservations
- Basic conversation import format
- PII hashing
- Schema validation
- ID linkage checks
- Timestamp normalization
- Gold sets for reviews, conversations, and recovery cases

Deliverables:

- Clean data tables
- Data-quality report
- Evaluation harness
- Gold set v0

Exit criteria:

- Every review maps to a property and reservation where possible.
- Conversation messages can be linked to reservations for at least one channel.
- Gold set is ready for testing.

### Phase 1 — Review Intelligence MVP

Build:

- Review aspect extraction
- Review-aspect schema
- Evidence-snippet validation
- Basic property-level aggregation
- Rating-text discrepancy
- Amenity–aspect promise gap v0

Deliverables:

- Review-aspect table
- Property summary table
- Promise-gap table
- Review intelligence report

Exit criteria:

- Aspect category micro-F1 ≥ 0.75.
- Evidence validity ≥ 99.5%.
- Manager finds top property issues recognizable.

### Phase 2 — Conversation Intelligence MVP

Build:

- Message ingestion for one or two channels
- Message issue extraction
- Issue episode builder
- Response-time and resolution metrics
- Conversation issue table

Deliverables:

- Message table
- Issue episode table
- Conversation quality metrics

Exit criteria:

- Issue extraction micro-F1 ≥ 0.75.
- Episode grouping F1 ≥ 0.70.
- Resolution-status accuracy ≥ 0.75.

### Phase 3 — Review–Conversation Alignment

Build:

- Match review aspects to issue episodes
- Alignment-type classification
- Preventability score
- Service recovery state
- Conversation-enhanced recommendation cards

Deliverables:

- Review–conversation alignment table
- Preventability report
- Service recovery classification
- Enhanced recommendation cards

Exit criteria:

- Alignment precision ≥ 0.80.
- Recovery-trigger precision ≥ 0.85.
- Managers agree alignment improves diagnosis.

### Phase 4 — Manager Recommendation System

Build:

- Ranked recommendation cards
- Evidence from reviews and conversations
- Cost/impact/confidence levels
- Decision capture
- Operational task linking
- Owner approval routing

Deliverables:

- Manager-facing card stack
- Recommendation table
- Decision log
- Operational task log

Exit criteria:

- ≥ 40% of top recommendations accepted or seriously considered.
- Zero unsupported claims in manager-facing cards.
- No safety issue bypasses human review.

### Phase 5 — Post-Review Customer Recovery

Build:

- Customer recovery case object
- Outreach/no-outreach classifier
- Strategy selector
- Offer policy matrix
- Approval workflow
- Private message drafter
- Send tracking

Deliverables:

- Recovery case table
- Draft message queue
- Offer approval queue
- Recovery outcomes dashboard

Exit criteria:

- Recovery-needed precision ≥ 0.85.
- Escalation recall ≥ 0.95.
- No message asks for review change.
- No compensation sent without approval.

### Phase 6 — Operational Feedback Loop

Build:

- Intervention tracking
- Action completion status
- Complaint recurrence detection
- Post-action monitoring
- Manager preference profile
- Suppression rules for rejected recommendations

Deliverables:

- Intervention outcome report
- Manager preference profile
- Recommendation suppression rules
- Active/monitoring/resolved issue states

Exit criteria:

- System avoids repeating rejected recommendations.
- Completed actions move issues into monitoring state.
- Future complaint recurrence is tracked.

### Phase 7 — Advanced Quantitative Layer

Build:

- Property health scoring
- Rating-driver analysis
- Conversation-quality score
- Pricing-readiness signal
- Stay-length and guest-density analysis
- Platform calibration
- Issue-risk scoring

Deliverables:

- Property health dashboard
- Risk score table
- Pricing-readiness report
- Portfolio insights

Exit criteria:

- Quantitative findings remain non-causal unless supported by outcomes.
- Managers find scores directionally useful.
- Sparse/new properties are labeled correctly.

### Phase 8 — Outcome Learning and Re-Ranking

Build:

- Outcome-aware recommendation ranking
- Before/after analysis
- Matched-control selection where possible
- Recovery strategy performance
- Offer effectiveness analysis
- Retention attribution framework

Deliverables:

- Learning dashboard
- Re-ranked recommendation engine
- Strategy-effectiveness report
- Outcome evidence library

Exit criteria:

- System can show which actions appear to reduce complaint recurrence.
- Recommendation acceptance improves over time.
- Recovery strategies can be compared by outcome.

### Phase 9 — Scale, Integrations, and Automation

Build:

- PMS/channel integrations
- WhatsApp/email/OTA integration
- Maintenance/ticketing integration
- Owner reports
- Manager Q&A
- Pre-arrival prevention engine
- Listing optimization engine
- Dynamic pricing integration where appropriate

Deliverables:

- Integrated agent system
- Owner report generator
- Preventive guest messaging engine
- Portfolio intelligence layer

Exit criteria:

- Human workflows remain controllable.
- Automation only expands where evaluation and guardrails are mature.

## 7. Non-negotiable product principles

1. **Evidence over opinion** — every manager-facing claim needs a source snippet or structured metric.
2. **Action over summary** — the output should shorten time-to-decision.
3. **Human approval for sensitive work** — compensation, refund, safety, legal, and public response actions require humans.
4. **No review manipulation** — never offer discounts or benefits in exchange for review changes.
5. **Cautious quantitative language** — no causal or dollar-ROI claims before intervention data exists.
6. **Cold-start honesty** — label whether evidence comes from property reviews, conversation history, portfolio priors, listing audit, or general best practice.
7. **Learning loop discipline** — accepted/rejected actions and outcomes are product data, not UI exhaust.

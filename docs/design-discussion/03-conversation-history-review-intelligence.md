# 03 — Conversation History Integration for Review Intelligence

> Working roadmap document. This extends the review-intelligence system by adding guest conversation history — Airbnb messages, WhatsApp, email, Booking.com messages, direct-booking chat, and support tickets — so the system can connect what guests complained about during the stay with what they later wrote in the review.

---

## 0. Why this document exists

The earlier roadmap focused on property listings, structured ratings, and post-stay reviews.

That is necessary, but incomplete.

A guest review is the final public artifact of a guest experience. The conversation history is the private timeline of what actually happened before that review.

If the system only reads the review, it may miss the most important context:

- Did the guest already complain during the stay?
- Did the host respond quickly?
- Was the issue resolved before checkout?
- Did the same issue reappear in the review despite being “resolved”?
- Did the guest never report the issue until the review?
- Did the host promise something that was not delivered?
- Did the guest ask for help and get ignored?
- Did a good service recovery prevent a bad review?
- Was the review complaint preventable?

The missing product layer is:

> **Connect the private conversation journey to the public review outcome.**

This makes the system much more valuable because it can diagnose not only what went wrong, but where the service process failed.

---

## 1. Updated product goal

The product goal becomes:

> **Build an AI decision copilot that connects listings, reservations, guest conversations, reviews, ratings, and manager actions into a single guest-experience intelligence loop.**

The system should answer:

```text
What did the guest experience?
When did the guest raise the issue?
How did the host respond?
Was the issue resolved?
Did the issue appear again in the review?
What should the property manager fix?
What should the guest communication team change?
What should be learned for future stays?
```

This extends the original review-intelligence flow:

```text
Review text
→ aspect extraction
→ recommendation card
```

into a fuller guest journey flow:

```text
Conversation history
+ review text
+ ratings
+ listing context
+ stay metadata
→ complaint timeline
→ resolution analysis
→ review alignment
→ property action
→ customer recovery action
→ feedback learning
```

---

## 2. The new entity model

The earlier model was:

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

With conversation history, the model becomes:

```text
Property Manager
    ↓
Property / Listing
    ↓
Reservation / Stay
    ↓
Guest / Traveller
    ↓
Conversation Thread(s)
    ↓
Message(s)
    ↓
Issue Episode(s)
    ↓
Review
    ↓
Review–Conversation Alignment
    ↓
Actions / Recovery / Outcomes
```

The key new object is the **Issue Episode**.

A conversation may contain many individual messages, but the system should group them into operational episodes.

Example:

```text
Message 1: “Hi, the Wi-Fi is not working.”
Message 2: “We tried restarting the router.”
Message 3: “Host: Sorry, we’ll send someone.”
Message 4: “Guest: It’s working now, thanks.”

Issue episode:
- Category: Wi-Fi
- Start: guest complaint
- Host response time: 12 minutes
- Resolution status: resolved
- Guest sentiment after resolution: positive
```

That episode can then be compared with the later review.

---

## 3. Why conversation history changes the intelligence

A review says what the guest chose to say publicly after the stay.

The conversation history shows the operational reality during the stay.

The combination creates much stronger diagnosis.

### Case A — Complaint in chat and complaint in review

Example:

```text
Chat: “The bathroom fan is very loud.”
Review: “The bathroom ventilation was noisy at night.”
```

Interpretation:

- Issue was raised during stay.
- Issue reappeared in review.
- Either it was not resolved or the resolution did not satisfy the guest.
- This should be treated as an active operational issue.

Action:

- Create maintenance task.
- Check host response quality.
- Consider customer recovery outreach.
- Monitor future reviews.

---

### Case B — Complaint in chat but absent from review

Example:

```text
Chat: “The Wi-Fi stopped working.”
Host: “We reset it and sent backup mobile hotspot details.”
Review: “Host was very responsive and the stay was great.”
```

Interpretation:

- Issue occurred.
- Host likely recovered the experience.
- Strong service recovery signal.
- The issue may still need technical follow-up, but the communication process worked.

Action:

- Track issue operationally.
- Reinforce the response playbook.
- Use as training example for support team.

---

### Case C — Complaint absent in chat but present in review

Example:

```text
No in-stay message.
Review: “The parking was confusing and frustrating.”
```

Interpretation:

- Guest did not report the issue during stay.
- The issue may be expectation-setting or instruction clarity.
- The system should prevent it proactively for future guests.

Action:

- Improve pre-arrival guide.
- Add parking photos.
- Add house manual FAQ.
- Do not blame support team for slow resolution if the guest never reported it.

---

### Case D — Repeated chat complaints but mild review

Example:

```text
Chat: multiple complaints about AC over two days.
Review: “Nice place overall, AC could be improved.”
Rating: 5 stars.
```

Interpretation:

- Courtesy rating.
- Hidden dissatisfaction.
- Review under-represents operational pain.
- Do not ignore the issue because the rating is high.

Action:

- Treat chat evidence as stronger than rating.
- Create maintenance task.
- Track as hidden risk.

---

### Case E — Positive chat and negative review

Example:

```text
Chat: “Thanks, all good.”
Review: “Host was unhelpful and check-in was confusing.”
```

Interpretation:

- Possible missing conversation channel.
- Guest may have been polite privately but dissatisfied publicly.
- Host may have overestimated resolution.
- Need review of full timeline.

Action:

- Flag for human review.
- Compare timestamps and channels.
- Improve confirmation questions: “Is everything now resolved?”

---

## 4. New data sources to ingest

The system should eventually ingest conversation history from:

- Airbnb messages
- Booking.com messages
- Agoda messages
- Expedia/VRBO messages
- WhatsApp
- Email
- Direct booking website chat
- SMS
- In-app support chat
- Internal support tickets
- Maintenance ticket comments
- Cleaner/vendor notes

For v1, start with one or two channels only.

Recommended v1 scope:

```text
Reservation-linked guest messages
+ review data
+ property/listing data
```

Do not start by integrating every channel. The important thing is to prove the linking logic.

---

## 5. Conversation data schema

Each raw message should be normalized into a common format.

```json
{
  "message_id": "msg_001",
  "thread_id": "thread_001",
  "reservation_id": "res_001",
  "property_id": "prop_abc",
  "guest_id_hash": "guest_hash_123",
  "channel": "airbnb",
  "sender_type": "guest",
  "sent_at": "2025-01-12T18:42:00Z",
  "message_text": "Hi, the Wi-Fi is not working in the bedroom.",
  "language": "en",
  "attachments": [],
  "source_system": "airbnb_messages"
}
```

Important fields:

| Field | Why it matters |
|---|---|
| `reservation_id` | Links conversation to stay |
| `property_id` | Links issue to property |
| `guest_id_hash` | Enables repeat-guest analysis without exposing email |
| `channel` | Helps understand communication source |
| `sender_type` | Separates guest complaint from host response |
| `sent_at` | Enables response-time and timeline analysis |
| `message_text` | Source for LLM issue extraction |
| `thread_id` | Groups messages into a conversation |

---

## 6. New structured object: issue episodes

The system should not treat each message independently.

It should group related messages into **Issue Episodes**.

Example schema:

```json
{
  "issue_episode_id": "issue_001",
  "reservation_id": "res_001",
  "property_id": "prop_abc",
  "guest_id_hash": "guest_hash_123",
  "issue_category": "wifi",
  "issue_subcategory": "bedroom_signal",
  "first_guest_message_id": "msg_001",
  "first_reported_at": "2025-01-12T18:42:00Z",
  "first_host_response_at": "2025-01-12T18:55:00Z",
  "host_response_minutes": 13,
  "resolution_status": "resolved",
  "resolved_at": "2025-01-12T20:10:00Z",
  "guest_sentiment_initial": "negative",
  "guest_sentiment_after_response": "positive",
  "severity": "medium",
  "evidence_snippets": [
    "Hi, the Wi-Fi is not working in the bedroom.",
    "It’s working now, thanks."
  ],
  "confidence": "high"
}
```

Resolution statuses:

| Status | Meaning |
|---|---|
| `unresolved` | Guest raised issue, no clear fix |
| `resolved` | Guest confirmed or evidence strongly suggests resolution |
| `partially_resolved` | Host responded, but guest remained dissatisfied |
| `unknown` | Not enough evidence |
| `escalated` | Sent to manager/vendor/maintenance |
| `compensated` | Refund/discount/credit offered |
| `guest_abandoned` | Guest stopped replying before resolution |

---

## 7. New structured object: review–conversation alignment

For each review aspect, compare it to conversation issue episodes.

Example schema:

```json
{
  "alignment_id": "align_001",
  "review_id": "rev_001",
  "reservation_id": "res_001",
  "property_id": "prop_abc",
  "review_aspect_category": "wifi",
  "matching_issue_episode_id": "issue_001",
  "alignment_type": "chat_complaint_repeated_in_review",
  "review_evidence": "Wi-Fi was unreliable in the bedroom.",
  "conversation_evidence": "Hi, the Wi-Fi is not working in the bedroom.",
  "resolution_status_before_review": "partially_resolved",
  "preventability": "high",
  "recommended_internal_action": "Upgrade router or add mesh node",
  "recommended_customer_action": "Send apology and mention fix before offering repeat-stay incentive",
  "confidence": "high"
}
```

Alignment types:

| Alignment type | Meaning |
|---|---|
| `chat_complaint_repeated_in_review` | Guest raised issue during stay and mentioned it again publicly |
| `chat_complaint_absent_from_review` | Issue occurred but did not affect review |
| `review_complaint_not_in_chat` | Guest only revealed issue after stay |
| `chat_resolved_positive_review` | Service recovery likely succeeded |
| `chat_unresolved_negative_review` | Service recovery failed |
| `positive_chat_negative_review` | Possible hidden dissatisfaction or missing context |
| `negative_chat_positive_review` | Courtesy rating or successful recovery |
| `no_chat_no_issue` | No relevant issue in either source |

---

## 8. Updated three-layer architecture

Conversation history strengthens all three layers.

### Layer 1 — Quantitative layer

Structured metrics from conversations:

- First response time
- Number of guest messages before host response
- Time to resolution
- Number of issue episodes per stay
- Issue category frequency
- Escalation rate
- Resolution rate
- Repeat-contact rate
- Complaint recurrence rate
- Review complaint after prior chat complaint
- Chat complaint absent from review
- Review complaint absent from chat
- Public review damage after unresolved issue
- Rating by response-time bucket
- Rating by resolution status
- Rating by issue category

Example analysis:

```text
Reservations with unresolved in-stay Wi-Fi episodes are associated with lower value ratings and more negative review text.
```

Use cautious language. This is association unless outcome experiments exist.

### Layer 2 — Interpretative LLM layer

The LLM now extracts meaning from:

- Reviews
- Guest messages
- Host replies
- Internal notes
- Maintenance comments

The LLM should identify:

- Guest complaint
- Guest request
- Question
- Emotional tone
- Urgency
- Issue category
- Whether host response addressed the issue
- Whether guest confirmed resolution
- Whether the same issue reappeared in the review
- Whether the host overpromised
- Whether compensation may be appropriate

### Layer 3 — Feedback-based learning loop

Conversation history improves the feedback loop because it tracks the service process before the review.

The system can learn:

- Which in-stay responses prevent negative reviews
- Which issues require faster escalation
- Which host templates work
- Which issues should trigger proactive pre-arrival warnings
- Which complaints deserve post-review recovery outreach
- Which properties generate repeat in-stay complaints
- Which operational fixes reduce both chat complaints and review complaints

---

## 9. New metrics

### Conversation-level metrics

| Metric | Definition |
|---|---|
| First response time | Minutes between guest issue and first host response |
| Time to resolution | Minutes/hours until issue appears resolved |
| Touch count | Number of messages needed to handle issue |
| Escalation count | Number of times issue moved to manager/vendor |
| Sentiment recovery | Guest sentiment before vs after host response |
| Unresolved issue rate | Share of issue episodes not resolved before checkout |
| Repeat complaint rate | Same issue mentioned multiple times in one stay |
| Post-review recurrence | Issue appears in review after conversation complaint |

### Property-level metrics

| Metric | Meaning |
|---|---|
| Conversation issue rate | How often guests raise issues during stays |
| Public review leakage | Issues that appear in reviews after being raised in chat |
| Silent complaint rate | Review complaints that were not raised during stay |
| Service recovery rate | Chat issues resolved without negative review impact |
| Escalation failure rate | Escalated issues that still become negative reviews |
| Communication quality score | Speed + clarity + sentiment recovery |

### Manager-level metrics

| Metric | Meaning |
|---|---|
| Average first response time | Communication responsiveness |
| Resolution rate | Operational effectiveness |
| Repeated unresolved issue categories | Process weakness |
| Compensation usage rate | Guest recovery cost |
| Review damage after complaint | Service recovery quality |

---

## 10. Recommendation logic with conversation history

Conversation history changes recommendation priority.

### Higher priority when:

- Guest raised issue during stay and it appears again in review
- Host did not respond quickly
- Host response did not address the complaint
- Same issue appears across multiple stays
- Issue was marked resolved but guest still complained in review
- Complaint is safety/security/hygiene related
- Guest expresses disappointment after an unresolved issue
- Review rating is low and conversation shows preventable failure

### Lower priority when:

- Issue was raised and resolved successfully
- Guest praised host recovery
- Complaint is old and no longer appears
- Issue was unique and low severity
- Guest did not respond after resolution and later left positive review

### New recommendation examples

```text
Action:
Create Wi-Fi escalation rule for stays longer than 3 nights.

Why:
Wi-Fi complaints appear during in-stay conversations and are repeated in later reviews, suggesting current troubleshooting is not preventing review damage.
```

```text
Action:
Add confirmation question after resolving guest issues.

Why:
Several issues were marked resolved by the host, but later appeared in reviews. Ask guests to confirm whether the issue is fully resolved before closing the case.
```

```text
Action:
Add parking photos to pre-arrival message.

Why:
Parking complaints appear in reviews but not in-stay chat, suggesting guests only realize the confusion on arrival and do not report it during the stay.
```

---

## 11. Service recovery classification

Conversation + review data enables service recovery analysis.

### Recovery states

| State | Meaning | Product action |
|---|---|---|
| Prevented complaint | Issue in chat, absent from review | Reinforce support playbook |
| Failed recovery | Issue in chat, repeated in review | Escalate ops + customer recovery |
| Silent dissatisfaction | No chat, complaint in review | Improve pre-arrival info/listing |
| Courtesy rating | Positive rating, negative conversation | Treat as hidden risk |
| Unreported operational issue | Review complaint not in chat | Add proactive prevention |
| Successful turnaround | Negative chat, positive review mentioning host | Use as training example |
| Escalation failure | Vendor/maintenance involved, review still negative | Review vendor/SLA process |

---

## 12. Customer recovery trigger

This document feeds directly into the post-review customer communication roadmap.

A customer recovery case should be created when:

- Review is negative or neutral
- Review mentions a specific complaint
- Complaint was raised in chat and unresolved or repeated
- Complaint was preventable
- Guest had a poor communication experience
- Guest is high-value or repeatable
- Issue was severe enough to justify apology, fix confirmation, or goodwill gesture

But the system should not automatically offer compensation. It should recommend a recovery strategy for human approval.

---

## 13. Updated agent/components

Add these components to the original roadmap.

### 1. Conversation ingestion component

Responsibilities:

- Load messages from supported channels
- Normalize sender, timestamp, channel, reservation, property
- Hash guest identifiers
- Remove unnecessary PII
- Store raw message references

### 2. Conversation issue extractor

LLM-based.

Responsibilities:

- Extract guest complaints, requests, questions, and praise
- Assign issue category, severity, urgency, and sentiment
- Identify whether message requires action
- Extract evidence snippets

### 3. Issue episode builder

Mostly deterministic, with optional LLM support.

Responsibilities:

- Group related messages into episodes
- Track issue start, host response, escalation, resolution
- Compute response time and resolution time
- Detect repeated complaints in the same stay

### 4. Review–conversation alignment agent

LLM + deterministic matching.

Responsibilities:

- Match review aspects to prior conversation issue episodes
- Classify alignment type
- Detect repeated/unresolved issues
- Detect silent complaints
- Assign preventability

### 5. Recovery recommendation agent

Feeds the post-review communication roadmap.

Responsibilities:

- Determine whether private outreach is appropriate
- Recommend apology/fix/discount/credit/no action
- Flag legal/safety cases for human review
- Draft recovery rationale

---

## 14. MVP integration plan

Do not integrate every channel at once.

### Phase C0 — Conversation data audit

Goal:

Understand what conversation data exists and how it links to reservations.

Build:

- Message schema
- Channel list
- Reservation linkage check
- PII mapping
- Timestamp normalization

Output:

- Conversation data-quality report

### Phase C1 — Issue extraction from messages

Goal:

Extract structured issues from guest messages.

Build:

- Message issue extraction schema
- Evidence validation
- Issue categories aligned with review taxonomy
- Severity and urgency classification

Output:

- Message issue table

### Phase C2 — Issue episode construction

Goal:

Group messages into operational episodes.

Build:

- Episode grouping logic
- Response-time calculation
- Resolution-status detection
- Escalation detection

Output:

- Issue episode table

### Phase C3 — Review–conversation alignment

Goal:

Connect review complaints with in-stay issues.

Build:

- Aspect-to-episode matching
- Alignment type classification
- Preventability score
- Service recovery state

Output:

- Review–conversation alignment table

### Phase C4 — Recommendation upgrade

Goal:

Use conversation evidence to improve recommendation cards.

Build:

- Add conversation evidence to cards
- Add resolution status
- Add preventability
- Add customer recovery trigger

Output:

- Enhanced recommendation cards

---

## 15. Evaluation plan

Conversation intelligence needs its own evaluation.

### Gold set

Label:

- 50 conversation threads
- 50 review–conversation pairs

If the dataset is small, start with 25 each and expand before release.

### Metrics

| Task | Metric | v1 target |
|---|---|---:|
| Message issue extraction | Aspect/issue micro-F1 | ≥ 0.75 |
| Message issue extraction | Polarity accuracy | ≥ 0.85 |
| Urgency classification | Accuracy | ≥ 0.80 |
| Evidence snippet validity | Substring match | ≥ 99.5% |
| Episode grouping | Pairwise grouping F1 | ≥ 0.70 |
| Resolution status | Accuracy | ≥ 0.75 |
| Review–chat matching | Alignment precision | ≥ 0.80 |
| Review–chat matching | Alignment recall | ≥ 0.65 |
| Preventability classification | Human agreement | ≥ 0.70 |

Prefer precision over recall for customer-facing recovery actions. A false recovery message can be worse than a missed one.

---

## 16. Guardrails

Conversation data is more sensitive than reviews.

Rules:

- Hash guest emails before LLM calls.
- Do not send unnecessary guest identifiers to the model.
- Treat guest messages as private data.
- Do not infer protected or sensitive traits from conversation content.
- Do not use guest nationality, name, accent, or language to downgrade service.
- Do not auto-send compensation offers.
- Do not auto-send public replies.
- Safety/legal/health/privacy issues require human review.
- Every extracted issue needs an evidence snippet.
- Every review–conversation alignment needs evidence from both sources when possible.
- Keep audit logs of extraction version and source message IDs.

---

## 17. How this changes the original roadmap

The original review-intelligence roadmap remains valid, but conversation history adds a new diagnostic layer.

The updated system should now support three linked records:

```text
Review Aspect Record
Conversation Issue Episode
Review–Conversation Alignment Record
```

Together, they allow the system to distinguish:

- Property problem
- Communication problem
- Expectation-setting problem
- Resolution failure
- Successful service recovery
- Silent dissatisfaction
- Customer retention opportunity

This makes the product much more operationally useful.

---

## 18. Final narrative

The review tells us what the guest said after the stay.

The conversation history tells us what happened during the stay.

The combination tells us what the property manager should do next.

This is the upgraded intelligence loop:

```text
Guest conversation
→ issue episode
→ host response and resolution analysis
→ guest review
→ review–conversation alignment
→ operational recommendation
→ customer recovery recommendation
→ manager decision
→ outcome learning
```

This should become a core part of the SympleHost AI agent system.

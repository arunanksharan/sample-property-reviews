# 06 — Conversation and Review Alignment Roadmap

## 1. Purpose

The review is the public artifact of the stay. The conversation history is the private timeline of what actually happened.

The goal of this module is to connect them.

Without conversation history, the system knows only what the guest wrote after checkout. With conversation history, the system can determine:

- whether the guest already complained during the stay
- how quickly the host responded
- whether the issue was resolved
- whether the issue still appeared in the review
- whether the complaint was preventable
- whether communication helped or harmed the outcome
- whether customer recovery is appropriate

## 2. Why this matters

### Review-only interpretation

```text
Review: “Wi-Fi was unreliable.”
Action: improve Wi-Fi.
```

### Review + conversation interpretation

```text
Chat day 1: “Wi-Fi not working in bedroom.”
Host response: 4 hours later.
Guest follow-up: “Still not working.”
Review: “Wi-Fi was unreliable and communication was slow.”
Diagnosis:
- Technical issue
- Slow response issue
- Failed recovery
- Higher customer recovery priority
```

The second interpretation is much more useful.

## 3. Conversation lifecycle

```text
Pre-arrival
→ check-in
→ in-stay support
→ issue escalation
→ resolution
→ checkout
→ review
→ post-review recovery
```

The system should understand where the issue happened in the journey.

## 4. Data sources

Start with one or two channels.

Recommended v1:

```text
Airbnb / OTA messages
+ direct booking chat or WhatsApp export
```

Later:

- Booking.com
- Agoda
- Expedia
- VRBO
- email
- SMS
- WhatsApp Business
- support tickets
- maintenance notes
- cleaner/vendor notes
- AI concierge transcripts

## 5. Message normalization

Every channel must be normalized into the same schema:

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
  "source_system": "airbnb_messages"
}
```

## 6. Message issue extraction

The LLM extracts message issues.

Output:

```json
{
  "message_issue_id": "mi_001",
  "message_id": "msg_001",
  "issue_category": "wifi",
  "issue_subcategory": "bedroom_signal",
  "message_intent": "complaint",
  "urgency": "medium",
  "severity": "medium",
  "requires_action": true,
  "evidence_snippet": "Wi-Fi is not working in the bedroom",
  "confidence": "high"
}
```

Message intents:

- complaint
- request
- question
- praise
- cancellation/refund
- check-in logistics
- safety/security
- maintenance
- other

## 7. Issue episode construction

Messages should be grouped into episodes.

An episode is one operational issue from first report to resolution or abandonment.

### Episode fields

- issue category
- first report timestamp
- first host response timestamp
- host response minutes
- resolution status
- resolution timestamp
- time to resolution
- escalation status
- compensation status
- guest sentiment before/after
- evidence snippets

### Episode statuses

| Status | Meaning |
|---|---|
| unresolved | no clear fix |
| resolved | guest confirmed or evidence strongly suggests resolution |
| partially_resolved | host acted but guest remained dissatisfied |
| unknown | unclear |
| escalated | sent to manager/vendor |
| compensated | refund/credit/discount offered |
| guest_abandoned | guest stopped replying |

## 8. Review–conversation alignment

Each review aspect should be compared with issue episodes from the same reservation.

### Alignment types

| Type | Meaning |
|---|---|
| chat_complaint_repeated_in_review | Raised during stay and appeared in review |
| chat_complaint_absent_from_review | Raised during stay but did not appear in review |
| review_complaint_not_in_chat | Appeared in review but not in chat |
| chat_resolved_positive_review | Issue resolved and review positive |
| chat_unresolved_negative_review | Issue unresolved and review negative |
| positive_chat_negative_review | Guest seemed fine in chat but negative in review |
| negative_chat_positive_review | Chat was negative but review positive |
| no_chat_no_issue | No relevant issue |

## 9. Interpretation matrix

| Chat state | Review state | Interpretation | Action |
|---|---|---|---|
| Complaint in chat | Complaint repeated in review | Failed or incomplete recovery | Fix property + customer recovery |
| Complaint in chat | Absent from review | Service recovery may have worked | Track issue + reinforce playbook |
| No complaint in chat | Complaint in review | Silent dissatisfaction / prevention gap | Improve pre-arrival info/listing |
| Negative chat | Positive review | Successful recovery or courtesy rating | Track hidden risk |
| Positive chat | Negative review | Missing context or private politeness | Human review |
| Repeated chat complaints | Mild review | Courtesy rating / hidden operational pain | Treat chat as strong evidence |
| Safety issue in chat | Any review | Escalation required | Human review |

## 10. Preventability scoring

Preventability asks:

```text
Could the business reasonably have prevented or reduced this complaint?
```

Factors:

| Factor | Effect |
|---|---|
| Issue was known before stay | higher preventability |
| Issue was raised during stay and not resolved | higher preventability |
| Listing overpromised | higher preventability |
| Pre-arrival instructions missing | higher preventability |
| Structural issue disclosed clearly | lower preventability |
| Guest never reported issue | medium, depending on preventability |
| Weather/external event | lower preventability |

Values:

- low
- medium
- high
- unknown

## 11. Conversation metrics

### Reservation-level

- issue episodes per stay
- first response time
- time to resolution
- escalation count
- repeat contact count
- sentiment recovery
- unresolved issue count

### Property-level

- issue rate per stay
- unresolved issue rate
- public review leakage
- silent complaint rate
- service recovery rate
- escalation failure rate
- repeat issue categories

### Manager-level

- average response time
- resolution rate
- service recovery success
- repeated unresolved categories
- guest satisfaction after support

## 12. How conversation changes recommendation ranking

Increase priority when:

- issue raised in chat and repeated in review
- host response slow
- issue unresolved
- guest followed up multiple times
- host promised fix that did not happen
- complaint is preventable
- issue is safety/hygiene/security
- repeated across multiple reservations

Decrease priority when:

- issue resolved quickly
- guest confirmed satisfaction
- issue absent from review
- issue old and not recurring
- complaint unique and low severity

## 13. Conversation-enhanced recommendation card

```text
Action:
Add a Wi-Fi escalation rule for stays longer than 3 nights.

Why:
Wi-Fi issues were raised during in-stay conversations and repeated in later reviews. Current troubleshooting appears insufficient for preventing review damage.

Conversation evidence:
- “The Wi-Fi is still not working in the bedroom.”
- Host response time: 4h 12m.

Review evidence:
- “Wi-Fi was unreliable during our stay.”

Alignment:
Chat complaint repeated in review.

Resolution status:
Partially resolved.

Preventability:
High.

Suggested owner:
Maintenance + guest support.

Customer recovery:
Recommended.
```

## 14. Phased implementation

### Phase C0 — Conversation data audit

Build:

- channel inventory
- reservation-linkage check
- PII map
- timestamp normalization
- sample threads

Output:

- conversation data quality report

### Phase C1 — Message issue extraction

Build:

- message issue schema
- extraction prompt
- Pydantic validation
- evidence check

Output:

- message issue table

### Phase C2 — Issue episodes

Build:

- grouping logic
- response time calculation
- resolution detection
- episode status

Output:

- issue episode table

### Phase C3 — Review alignment

Build:

- aspect-to-episode matching
- alignment classification
- preventability scoring
- service recovery state

Output:

- alignment table

### Phase C4 — Recommendation upgrade

Build:

- conversation evidence in recommendation cards
- review leakage detection
- customer recovery trigger
- service process recommendations

Output:

- enhanced recommendations

## 15. Evaluation

Gold set:

- 50 conversation threads
- 50 review–conversation pairs

Metrics:

| Task | Metric | Target |
|---|---|---:|
| Message issue extraction | Micro-F1 | ≥ 0.75 |
| Intent classification | Accuracy | ≥ 0.80 |
| Evidence validity | Substring | ≥ 99.5% |
| Episode grouping | Pairwise F1 | ≥ 0.70 |
| Resolution status | Accuracy | ≥ 0.75 |
| Alignment | Precision | ≥ 0.80 |
| Alignment | Recall | ≥ 0.65 |
| Preventability | Agreement | ≥ 0.70 |

## 16. Guardrails

- Conversation data is private.
- Hash guest identifiers.
- Do not send unnecessary identifiers to LLM.
- Do not infer sensitive traits.
- Every issue needs evidence.
- Every alignment needs evidence where available.
- No customer recovery message should be auto-sent from alignment alone.
- Safety/legal/privacy cases require human review.

## 17. Integration with customer recovery

The alignment module produces the key inputs for post-review recovery:

- complaint category
- severity
- resolution status
- preventability
- guest sentiment after issue
- whether issue was repeated in review
- whether compensation was already offered
- whether outreach is appropriate

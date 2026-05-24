# 07 — Post-Review Recovery and Retention Roadmap

## 1. Purpose

This roadmap defines how the system should decide what to say to a guest after they leave a review.

This is not the same as public review response.

There are two communication surfaces:

```text
Public review response:
- visible to future guests
- protects brand reputation
- short, careful, non-defensive

Private post-review outreach:
- sent directly to the guest
- repairs relationship
- confirms fixes
- may include goodwill gesture
- supports retention and repeat booking
```

This document focuses on private outreach.

## 2. Product goal

Build an AI-assisted recovery system that:

- reads the review
- reads conversation history
- understands complaint and service timeline
- decides whether private outreach is appropriate
- chooses a recovery strategy
- recommends whether a goodwill offer is appropriate
- drafts a message for human approval
- tracks guest response and retention outcome
- learns which recovery strategies work

## 3. Non-negotiable principle: do not buy reviews

Never offer a discount, refund, gift, benefit, or upgrade in exchange for review modification.

Forbidden:

```text
We will give you 20% off if you update your review.
```

Allowed:

```text
We are sorry the stay did not fully meet expectations. We have addressed the issue and would like to offer a goodwill credit toward a future stay, with no obligation attached.
```

The communication is about service recovery, not review manipulation.

## 4. Customer recovery case

Central object:

```json
{
  "recovery_case_id": "case_001",
  "review_id": "rev_001",
  "reservation_id": "res_001",
  "property_id": "prop_abc",
  "guest_id_hash": "guest_hash_123",
  "complaint_category": "parking",
  "complaint_severity": "medium",
  "conversation_alignment": "review_complaint_not_in_chat",
  "resolution_status": "not_applicable",
  "preventability": "high",
  "retention_potential": "medium",
  "recommended_strategy": "apology_plus_future_discount",
  "offer_type": "future_stay_discount",
  "offer_level": "low",
  "approval_required": true,
  "draft_status": "pending_human_review"
}
```

## 5. Recovery strategy types

### Strategy 1 — Thank-you / relationship reinforcement

Use when:

- review positive
- no meaningful complaint
- guest likely to return
- direct booking nurture is allowed

Offer:

- usually none
- optional loyalty code if policy supports it

### Strategy 2 — Minor improvement acknowledgment

Use when:

- review positive overall
- minor issue raised
- issue easy to fix

Examples:

- extra coffee pods
- better lighting
- clearer parking instruction
- more towels

Offer:

- usually none
- optional low goodwill if direct retention strategy exists

### Strategy 3 — Service recovery apology

Use when:

- neutral/negative review
- real complaint
- issue affected stay
- no major compensation required

Offer:

- apology
- fix confirmation
- possible future invitation

### Strategy 4 — Goodwill gesture / future discount

Use when:

- complaint valid
- complaint preventable
- experience materially degraded
- guest may be retained
- manager policy allows it

Offer options:

- future stay discount
- direct booking credit
- late checkout next stay
- complimentary add-on
- partial refund only with approval

### Strategy 5 — Escalated human recovery

Use when:

- safety/security
- severe hygiene
- legal issue
- privacy/discrimination complaint
- angry guest
- refund dispute
- large compensation request

Offer:

- human-decided only

### Strategy 6 — No private outreach

Use when:

- review positive and no campaign exists
- complaint vague
- guest already contacted
- outreach may be excessive
- platform/policy restrictions apply

## 6. Decision matrix

| Review/conversation state | Action |
|---|---|
| Positive review, no complaint | Thank-you / nurture |
| Positive review, minor issue | Thank + acknowledge improvement |
| High rating but complaint text | Hidden dissatisfaction outreach |
| Neutral review, specific complaint | Service recovery message |
| Negative review, complaint not in chat | Apology + ask/detail + fix confirmation |
| Negative review, unresolved chat issue | Apology + escalation + possible goodwill |
| Chat issue resolved, positive review | Thank guest + reinforce relationship |
| Chat issue repeated in review | Recovery + operational escalation |
| Safety/security | Human escalation |
| Refund request | Human escalation |
| Repeat/direct guest | Prioritize retention flow |
| OTA guest | Use channel-compliant messaging path |

## 7. Offer decisioning

### Offer types

| Offer | When |
|---|---|
| No offer | positive/minor |
| Apology only | low severity |
| Fix confirmation | issue addressed |
| Future discount | valid preventable complaint |
| Direct credit | direct booking strategy |
| Complimentary add-on | low-cost retention |
| Partial refund | severe verified failure with approval |
| Escalation only | safety/legal/refund dispute |

### Offer levels

| Level | Meaning |
|---|---|
| None | no financial gesture |
| Low | small future discount/add-on |
| Medium | meaningful credit/service |
| High | refund/large credit; approval |
| Escalated | senior/legal review |

Avoid exact amounts in v1 unless the customer provides an offer policy table.

### Offer policy example

```text
Low severity:
    apology or fix confirmation

Medium severity + high preventability:
    apology + fix confirmation + optional low future discount

High severity + unresolved:
    apology + escalation + medium goodwill gesture, approval required

Safety/legal:
    human escalation only
```

## 8. Retention potential

Use behavioral signals, not demographic assumptions.

Positive signals:

- repeat guest
- direct booking
- long stay
- high booking value
- guest says they would return
- positive comments despite complaint
- complaint is fixable
- location/property praised

Do not use:

- ethnicity
- nationality
- inferred wealth
- protected traits
- sensitive personal attributes

Simple v1 score:

```text
retention_potential =
  repeat_guest_signal
+ direct_booking_signal
+ positive_text_signal
+ complaint_fixability
+ guest_return_intent
- severe_dissatisfaction
```

Use this only for prioritization, not unequal service quality.

## 9. Message generation principles

A good message is:

- specific
- short
- human
- non-defensive
- grounded in evidence
- clear about fix
- careful about offer
- free from pressure to change review

Required parts:

1. greeting
2. thanks
3. specific issue acknowledgment
4. accountability
5. fix or next step
6. goodwill offer if applicable
7. closing / invitation to reply

Avoid:

- “Please change your review”
- “This never happens”
- “You should have told us earlier”
- “We disagree”
- blaming guest/vendor/cleaner
- unapproved compensation
- legal admissions

## 10. Human approval workflow

Approval levels:

| Case | Approval |
|---|---|
| Thank-you only | guest support |
| Minor apology | guest support |
| Low future discount | property manager |
| Medium/high credit | senior manager |
| Refund | senior manager / finance |
| Safety/legal/privacy | senior + legal/policy |
| Angry guest | senior review |

Approval card:

```text
Guest:
Property:
Review rating:
Complaint:
Conversation summary:
Was issue raised during stay?
Resolution status:
Recommended strategy:
Offer recommendation:
Draft message:
Linked operational task:
Risks:
Approve / Edit / Reject / Escalate
```

## 11. Link customer recovery to operational action

Never send empty apologies.

If the message says:

```text
We are fixing the bathroom ventilation.
```

then the system must create or link a task:

```text
Inspect bathroom fan and ventilation.
```

The recovery case should not be marked complete until:

- message sent or intentionally skipped
- task created
- task owner assigned
- follow-up outcome tracked

## 12. Draft templates

### Positive review

```text
Hi {{guest_first_name}},

Thank you for staying with us and for sharing such kind feedback. We’re glad you enjoyed {{positive_aspect}} at {{property_name}}.

We’d be happy to host you again whenever your plans bring you back to {{city}}.

Warmly,
{{team_name}}
```

### Positive review with minor issue

```text
Hi {{guest_first_name}},

Thank you for staying with us and for the thoughtful feedback. We’re glad you enjoyed {{positive_aspect}}.

We also noted your comment about {{minor_issue}}. That is helpful, and we’re reviewing how to improve it for future guests.

We’d be happy to welcome you back again.

Warmly,
{{team_name}}
```

### Neutral review with complaint

```text
Hi {{guest_first_name}},

Thank you for staying with us and for taking the time to share detailed feedback.

I’m sorry that {{specific_issue}} affected your stay. We understand how that would have been frustrating, especially given {{stay_context_if_relevant}}.

We’re taking the following step: {{specific_fix_or_next_action}}.

We appreciate you flagging this and would like the opportunity to host you again under a better experience.

Warmly,
{{team_name}}
```

### Negative review with unresolved issue and goodwill offer

```text
Hi {{guest_first_name}},

Thank you for your feedback. I’m sorry that {{specific_issue}} affected your stay at {{property_name}}.

Looking at the stay history, this should have been handled better. We are now {{specific_fix_or_escalation}} so future guests do not face the same issue.

As a goodwill gesture, we’d like to offer {{offer_description}} toward a future stay, with no obligation attached. If you’re open to it, we’d be glad to host you again and provide a better experience.

Warmly,
{{team_name}}
```

### Complaint not raised during stay

```text
Hi {{guest_first_name}},

Thank you for staying with us and for sharing your feedback after checkout.

I’m sorry to hear that {{specific_issue}} affected your experience. We did not realize this during the stay, so we appreciate you bringing it to our attention now.

We’re using your feedback to improve {{specific_prevention_step}}, so future guests have clearer guidance and a smoother stay.

Warmly,
{{team_name}}
```

## 13. Metrics

Operational:

- draft approval rate
- edit distance
- time to send
- escalation rate
- offer approval rate
- message send rate

Guest outcome:

- guest reply rate
- positive reply rate
- repeat booking rate
- discount redemption rate
- direct booking conversion
- future review sentiment
- complaint recurrence

Guardrail:

- review-change requests = 0
- compensation without approval = 0
- safety/legal auto-send = 0
- unsupported issue claims = 0

## 14. Evaluation

Gold set:

```text
50 post-review recovery cases
```

Metrics:

| Task | Metric | Target |
|---|---|---:|
| Recovery needed | Precision | ≥ 0.85 |
| Recovery needed | Recall | ≥ 0.70 |
| Offer appropriateness | Human agreement | ≥ 0.80 |
| Escalation detection | Recall | ≥ 0.95 |
| Draft factuality | Evidence-grounded | ≥ 0.95 |
| Draft tone | Human acceptable | ≥ 0.85 |
| Policy violation | Critical failures | 0 |

## 15. Phased roadmap

### Phase R0 — Policy design

- offer policy
- approval rules
- escalation rules
- tone guide
- platform restrictions

### Phase R1 — Triage

- decide recovery needed
- classify severity
- detect escalation
- create recovery case

### Phase R2 — Strategy and offer

- choose strategy
- recommend offer level
- check policy
- require approval

### Phase R3 — Draft generation

- create draft
- evidence check
- policy lint
- tone lint

### Phase R4 — Send and track

- approval queue
- send queue
- guest reply tracking
- offer redemption tracking

### Phase R5 — Retention learning

- repeat booking tracking
- strategy effectiveness
- offer effectiveness
- recommendation re-ranking

## 16. Final principle

The goal is not to “reply to reviews.”

The goal is:

```text
Detect issue
→ fix property
→ recover guest trust
→ track outcome
→ learn which recovery actions work
```

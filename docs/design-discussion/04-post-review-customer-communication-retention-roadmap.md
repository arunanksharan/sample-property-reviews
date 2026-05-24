# 04 — Post-Review Customer Communication and Retention Roadmap

> Working roadmap document. This document builds on the review-intelligence and conversation-history systems. Its goal is to help property managers communicate with guests after a review, especially when a guest raised a complaint, with the objective of service recovery, retention, repeat bookings, and brand trust.

---

## 0. Why this roadmap exists

The review-intelligence roadmap identifies what guests liked, what they complained about, and what the property manager should fix.

The conversation-history roadmap adds the timeline of what happened during the stay.

But there is one more important business workflow:

> **What should we say to the customer after the review?**

This is not the same as a public review response.

There are two different communication surfaces:

```text
Public review response:
- Seen by future guests
- Protects brand reputation
- Should be short, polite, and careful

Private post-review outreach:
- Sent directly to the guest
- Focuses on recovery, retention, and relationship repair
- May include apology, fix confirmation, goodwill gesture, future discount, or concierge follow-up
```

This document focuses on the second surface: **private customer communication after the review**.

The goal is not to manipulate reviews. The goal is to recover trust, acknowledge real issues, and increase the chance that a guest returns or recommends the property.

---

## 1. Product goal

Build an AI-assisted post-review communication system that:

- Reads the review
- Reads the conversation history
- Understands the complaint and service timeline
- Classifies whether customer recovery is needed
- Chooses an appropriate outreach strategy
- Drafts a private message for human approval
- Recommends whether to offer a goodwill gesture
- Tracks guest response and repeat-booking outcome
- Learns which recovery strategies work

The system should answer:

```text
Should we contact this guest privately?
What should we say?
Should we apologize, explain, offer a fix, offer a discount, or do nothing?
Who should approve the message?
What should we track after sending?
Did the outreach increase retention?
```

---

## 2. Guiding principles

### 1. Do not buy reviews

Never offer a discount, refund, gift, or benefit in exchange for changing, deleting, or improving a review.

Bad:

```text
We’ll give you 20% off if you update your review.
```

Good:

```text
We’re sorry the stay did not fully meet expectations. We have addressed the issue and would like to offer a goodwill credit toward a future stay, with no obligation attached.
```

The communication must be about service recovery, not review manipulation.

### 2. Human approval for anything sensitive

The system can draft and recommend, but should not auto-send messages involving:

- Refunds
- Discounts
- Safety issues
- Legal issues
- Health/hygiene complaints
- Angry guests
- Discrimination or privacy concerns
- Damage disputes
- Platform policy-sensitive topics

### 3. Match the response to the issue

Do not overcompensate for minor issues. Do not under-respond to serious failures.

### 4. Use evidence, not generic apology

The message should reference the specific issue:

```text
We saw your note about the bathroom ventilation...
```

not:

```text
Sorry for any inconvenience.
```

### 5. Confirm action where possible

Guests trust recovery more when they see the business actually fixed something.

```text
We have asked our maintenance team to inspect the fan and improve ventilation before the next stay.
```

### 6. Separate operational action from customer recovery

Fixing the property and messaging the guest are separate workflows.

The system should create both when needed:

```text
Operational action:
Inspect bathroom ventilation.

Customer action:
Send apology and recovery message to guest.
```

---

## 3. Inputs

The post-review communication engine should use:

### Review data

- Review text
- Review title
- What guest liked
- What guest did not like
- Additional comments
- Overall rating
- Subratings
- Sentiment
- Review platform
- Response status

### Conversation history

- Guest messages before/during/after stay
- Host replies
- Issue episodes
- Response times
- Resolution status
- Escalation history
- Whether compensation was already offered
- Guest sentiment after resolution

### Reservation data

- Stay dates
- Number of nights
- Guest count
- Booking value
- Channel
- Repeat guest flag
- Direct vs OTA booking
- Future booking eligibility

### Property data

- Property type
- Amenities
- Known recurring issues
- Whether issue has been fixed
- Owner/manager constraints

### Policy data

- Allowed discount levels
- Refund authority
- Approval rules
- Platform restrictions
- Brand tone
- Manager preferences
- Owner restrictions

---

## 4. Core output: customer recovery case

The central object is a **Customer Recovery Case**.

Example schema:

```json
{
  "recovery_case_id": "case_001",
  "review_id": "rev_001",
  "reservation_id": "res_001",
  "property_id": "prop_abc",
  "guest_id_hash": "guest_hash_123",
  "case_type": "private_post_review_outreach",
  "complaint_category": "parking",
  "complaint_severity": "medium",
  "review_rating": 3,
  "conversation_alignment": "review_complaint_not_in_chat",
  "resolution_status": "not_applicable",
  "preventability": "high",
  "retention_potential": "medium",
  "recommended_strategy": "apology_plus_future_discount",
  "offer_type": "future_stay_discount",
  "offer_level": "low",
  "approval_required": true,
  "draft_status": "pending_human_review",
  "created_at": "2026-05-24T00:00:00Z"
}
```

This case links:

```text
Review complaint
+ conversation history
+ operational issue
+ customer recovery strategy
+ manager decision
+ outcome
```

---

## 5. Communication strategy types

Not every review needs a discount.

The system should choose from a set of strategies.

### Strategy 1 — Thank-you and relationship reinforcement

Use when:

- Review is positive
- No meaningful complaint
- Guest says they would return
- Guest is a good candidate for repeat booking

Message goal:

- Thank guest
- Reinforce relationship
- Invite direct booking or future stay if appropriate

Offer:

- Usually no discount
- Optional loyalty code for direct booking strategy

---

### Strategy 2 — Thank-you plus minor improvement acknowledgment

Use when:

- Review is positive overall
- Minor complaint exists
- Issue is low severity and easy to fix

Example issue:

- More coffee pods
- More task lighting
- Clearer parking instruction
- More towels

Message goal:

- Thank guest
- Acknowledge specific suggestion
- Tell them it is being improved
- Optionally invite return

Offer:

- Usually no immediate discount
- Optional small goodwill code if direct-retention strategy exists

---

### Strategy 3 — Service recovery apology

Use when:

- Review is neutral or negative
- Guest had a real complaint
- Issue affected stay quality
- No major compensation required yet

Message goal:

- Apologize specifically
- Acknowledge impact
- Explain what will be fixed
- Invite the guest to return with confidence

Offer:

- Optional future-stay discount
- No refund unless policy supports it

---

### Strategy 4 — Goodwill gesture / future discount

Use when:

- Complaint is valid
- Complaint was preventable
- Guest had a materially degraded experience
- Guest may be retained
- Property manager policy allows offer

Examples:

- Wi-Fi unavailable during work stay
- Check-in failure
- Maintenance issue not resolved
- Parking confusion caused significant inconvenience
- Supplies missing during long stay

Message goal:

- Repair relationship
- Show accountability
- Offer future incentive without asking for review change

Offer types:

- Future stay discount
- Direct booking credit
- Free early check-in / late checkout next time
- Complimentary add-on
- Partial refund only where appropriate and approved

---

### Strategy 5 — Escalated human recovery

Use when:

- Safety/security issue
- Severe hygiene issue
- Angry guest
- Legal/compliance issue
- Discrimination/privacy concern
- Large refund request
- Public reputational risk

Message goal:

- Do not auto-handle
- Route to senior manager
- Prepare evidence summary
- Draft only after human review

Offer:

- Human-decided

---

### Strategy 6 — No private outreach

Use when:

- Review is positive and no retention campaign exists
- Complaint is vague and low confidence
- Guest has already been contacted
- Outreach would seem excessive
- Platform or policy makes outreach inappropriate

Message goal:

- Do nothing privately
- Optionally send public review response only

---

## 6. Recovery decision matrix

The system should decide outreach based on severity, resolution status, and retention potential.

| Review / conversation state | Suggested action |
|---|---|
| Positive review, no complaint | Thank-you / loyalty nurture |
| Positive review, minor issue | Thank + acknowledge improvement |
| High rating but complaint in text | Hidden dissatisfaction outreach |
| Neutral review, specific complaint | Service recovery message |
| Negative review, complaint not raised in chat | Apology + ask for more detail + fix confirmation |
| Negative review, issue raised in chat and unresolved | Apology + escalation + possible goodwill gesture |
| Complaint raised in chat and resolved, positive review | Thank guest + reinforce relationship |
| Complaint raised in chat and repeated in review | Service recovery + operational escalation |
| Safety/security complaint | Human escalation, no auto-send |
| Review asks for refund | Human escalation |
| Guest is repeat/high-value | Prioritize private outreach |
| Guest booked direct | Prioritize retention flow |
| Guest booked OTA | Use channel-compliant messaging path |

---

## 7. Offer decisioning

The system should recommend offers, not automatically grant them.

### Offer types

| Offer type | When to use |
|---|---|
| No offer | Positive review or minor issue |
| Apology only | Low-severity issue, fix already underway |
| Fix confirmation | Issue has been or will be resolved |
| Future-stay discount | Retain guest after valid complaint |
| Direct booking credit | Encourage repeat direct booking where allowed |
| Complimentary add-on | Early check-in, late checkout, local experience, parking support |
| Partial refund | Only for severe verified service failure and with approval |
| Escalation only | Safety/legal/refund dispute |

### Offer level

Use simple levels in v1.

| Level | Meaning |
|---|---|
| None | No financial gesture |
| Low | Small future-stay discount or minor add-on |
| Medium | Meaningful future credit or complimentary service |
| High | Refund/large credit, requires manager approval |
| Escalated | Human/senior approval required |

Avoid dollar amounts in v1 unless the customer provides a policy table.

### Factors in offer recommendation

| Factor | Effect |
|---|---|
| Complaint severity | Higher severity increases offer level |
| Preventability | Preventable issues increase recovery need |
| Resolution failure | Unresolved issues increase recovery need |
| Guest value | Repeat/direct/high-value guests may justify higher gesture |
| Evidence strength | Strong evidence increases confidence |
| Manager policy | Caps offer type/level |
| Owner constraints | May prevent discounts/refunds |
| Safety/legal risk | Forces human review |

### Example offer policy

```text
If severity = low:
    no offer or apology only

If severity = medium and preventability = high:
    apology + fix confirmation + optional low future discount

If severity = high and unresolved:
    apology + escalation + medium goodwill gesture with approval

If safety/legal:
    human escalation only
```

---

## 8. Message generation principles

The draft message should be:

- Specific
- Human
- Short enough to read
- Honest
- Non-defensive
- Not manipulative
- Clear about the fix
- Careful about compensation language
- Free of pressure to change the review

### Required parts

| Part | Purpose |
|---|---|
| Greeting | Human tone |
| Thanks | Acknowledge time taken to leave feedback |
| Specific issue acknowledgment | Show that the feedback was actually read |
| Accountability | Avoid excuses |
| Action/fix | Tell guest what will happen |
| Goodwill offer if applicable | Retention or recovery |
| Closing | Invite direct reply |

### Avoid

- “Please change your review”
- “We disagree with your feedback”
- “This has never happened before”
- “You should have told us earlier”
- “Here is a discount if you update your rating”
- Legal admissions in sensitive cases
- Blaming cleaners, vendors, or the guest

---

## 9. Draft templates

These are templates, not final messages. The LLM should fill them with property-specific and issue-specific context.

### Positive review, no complaint

```text
Hi {{guest_first_name}},

Thank you for staying with us and for sharing such kind feedback. We’re glad you enjoyed {{positive_aspect}} at {{property_name}}.

We’d be happy to host you again whenever your plans bring you back to {{city}}.

Warmly,
{{manager_name_or_team}}
```

---

### Positive review with minor issue

```text
Hi {{guest_first_name}},

Thank you for staying with us and for the thoughtful feedback. We’re glad you enjoyed {{positive_aspect}}.

We also noted your comment about {{minor_issue}}. That is helpful, and we’re already reviewing how to improve it for future guests.

We’d be happy to welcome you back again.

Warmly,
{{manager_name_or_team}}
```

---

### Neutral review with specific complaint

```text
Hi {{guest_first_name}},

Thank you for staying with us and for taking the time to share detailed feedback.

I’m sorry that {{specific_issue}} affected your stay. We understand how that would have been frustrating, especially given {{stay_context_if_relevant}}.

We’re taking the following step: {{specific_fix_or_next_action}}.

We appreciate you flagging this and would like the opportunity to host you again under a better experience.

Warmly,
{{manager_name_or_team}}
```

---

### Negative review, unresolved issue, goodwill offer

```text
Hi {{guest_first_name}},

Thank you for your feedback. I’m sorry that {{specific_issue}} affected your stay at {{property_name}}.

Looking at the stay history, this should have been handled better. We are now {{specific_fix_or_escalation}} so future guests do not face the same issue.

As a goodwill gesture, we’d like to offer {{offer_description}} toward a future stay, with no obligation attached. If you’re open to it, we’d be glad to host you again and provide a better experience.

Warmly,
{{manager_name_or_team}}
```

---

### Review complaint not raised during stay

```text
Hi {{guest_first_name}},

Thank you for staying with us and for sharing your feedback after checkout.

I’m sorry to hear that {{specific_issue}} affected your experience. We did not realize this during the stay, so we appreciate you bringing it to our attention now.

We’re using your feedback to improve {{specific_prevention_step}}, so future guests have clearer guidance and a smoother stay.

Warmly,
{{manager_name_or_team}}
```

---

## 10. Human approval workflow

The system should produce a draft and recommendation, but a person approves the final send.

### Approval levels

| Case | Approval |
|---|---|
| Thank-you only | Can be approved by guest support |
| Minor apology | Guest support approval |
| Future discount low level | Property manager approval |
| Medium/high credit | Senior manager approval |
| Refund | Senior manager / finance approval |
| Safety/legal/privacy | Senior manager + legal/policy review |
| Angry guest | Senior manager review |

### Approval card

The human reviewer should see:

```text
Guest:
Property:
Review rating:
Complaint:
Conversation history summary:
Was issue raised during stay?
Resolution status:
Recommended strategy:
Offer recommendation:
Draft message:
Risks:
Approve / Edit / Reject / Escalate
```

---

## 11. Integration with operational action

Customer outreach should not be disconnected from property improvement.

If the message says “we are fixing this,” the system must create or link the operational task.

Example:

```text
Customer recovery case:
Apologize for bathroom ventilation issue.

Linked operational task:
Inspect bathroom ventilation fan and report completion.

Do not send final fix-confirmation message until task is created.
```

This prevents empty apologies.

---

## 12. Retention and CRM logic

Not every guest is equally likely to return, but the system should avoid unfair or discriminatory assumptions.

Use behavioral and booking data, not personal demographic assumptions.

Good retention signals:

- Repeat guest
- Direct booking
- Long stay
- High booking value
- Positive comments despite complaint
- Guest says they would return
- Guest praised location/property but had fixable issue
- Complaint was preventable and fixable

Avoid using:

- Name
- Nationality
- Ethnicity
- Sensitive personal traits
- Inferred wealth or protected characteristics

### Retention score

A simple v1 score can use:

```text
retention_potential =
repeat_guest_signal
+ direct_booking_signal
+ positive_text_signal
+ complaint_fixability
+ guest_return_intent
- severe_dissatisfaction
```

Use only as prioritization support, not as a decision that reduces service quality.

---

## 13. Metrics

The post-review communication system should be measured.

### Operational metrics

| Metric | Meaning |
|---|---|
| Draft approval rate | Are AI drafts usable? |
| Edit distance | How much humans rewrite |
| Time to send | Speed of recovery |
| Escalation rate | How many cases need senior review |
| Offer approval rate | Whether recommended offers fit policy |
| Message send rate | How often outreach is actually sent |

### Guest outcome metrics

| Metric | Meaning |
|---|---|
| Guest reply rate | Whether guests engage |
| Positive reply rate | Whether recovery lands well |
| Repeat booking rate | Retention outcome |
| Discount redemption rate | Offer effectiveness |
| Direct booking conversion | Commercial impact |
| Complaint recurrence | Whether property issue was fixed |
| Future review sentiment | Longer-term experience impact |

### Guardrail metrics

| Metric | Target |
|---|---:|
| Messages asking for review change | 0 |
| Compensation without approval | 0 |
| Safety/legal auto-send | 0 |
| Unsupported issue claims | 0 |
| PII leakage to unnecessary systems | 0 |

---

## 14. Evaluation plan

Evaluate the communication engine separately from the review-analysis engine.

### Gold set

Create 50 post-review cases with human labels.

Each case should include:

- Review
- Conversation summary
- Complaint category
- Severity
- Resolution status
- Recommended strategy
- Whether offer is appropriate
- Draft quality rating

### Metrics

| Task | Metric | v1 target |
|---|---|---:|
| Recovery-needed classification | Precision | ≥ 0.85 |
| Recovery-needed classification | Recall | ≥ 0.70 |
| Offer appropriateness | Human agreement | ≥ 0.80 |
| Escalation detection | Recall | ≥ 0.95 |
| Draft factuality | Evidence-grounded | ≥ 0.95 |
| Draft tone quality | Human-rated acceptable | ≥ 0.85 |
| Policy-violation rate | Critical failures | 0 |

For this workflow, false positives can be costly. A bad outreach message can make an unhappy guest more upset. Prefer precision and human approval.

---

## 15. Phased roadmap

---

# Phase R0 — Policy and recovery design

## Goal

Define what the system is allowed to recommend before generating any messages.

## Build

- Offer policy table
- Approval rules
- Escalation rules
- Message tone guide
- Platform/channel restrictions
- Human approval workflow
- Compensation levels: none/low/medium/high/escalated

## Output

- Recovery policy config
- Approval matrix
- Message guardrails

## Success criteria

- Managers agree the policy reflects how they want to treat guests
- The system knows when not to recommend compensation
- Sensitive cases are routed to humans

---

# Phase R1 — Review and conversation triage

## Goal

Decide whether private outreach is needed.

## Build

- Complaint severity classifier
- Review sentiment classifier
- Conversation alignment input
- Resolution-status input
- Recovery-needed classifier
- Escalation detector

## Output

- Customer recovery case table
- Outreach/no-outreach decision
- Escalation queue

## Success criteria

- Negative/high-risk cases are not missed
- Positive/no-issue reviews are not over-contacted
- Safety/legal issues are escalated

---

# Phase R2 — Strategy and offer recommendation

## Goal

Choose the right recovery strategy.

## Build

- Strategy selector
- Offer-level recommender
- Retention-potential score
- Approval requirement detector
- Policy compliance checker

## Output

- Recommended communication strategy
- Offer recommendation
- Approval requirement

## Success criteria

- Managers agree with recommended strategy in most cases
- Offers are aligned with policy
- No contingent-review incentives are generated

---

# Phase R3 — Draft generation

## Goal

Generate human-quality private outreach drafts.

## Build

- Template library
- LLM draft generator
- Evidence-grounding check
- Tone and policy lint
- Human-edit interface

## Output

- Draft messages
- Message rationale
- Risk notes

## Success criteria

- Drafts require minimal editing
- Drafts are specific and empathetic
- Drafts avoid unsafe or manipulative language

---

# Phase R4 — Send, track, and follow up

## Goal

Turn approved drafts into tracked customer communication.

## Build

- Send queue
- Approval workflow
- CRM note creation
- Guest reply tracking
- Reminder follow-ups
- Offer redemption tracking

## Output

- Sent message log
- Guest response log
- Follow-up tasks
- Offer redemption log

## Success criteria

- Messages are sent quickly after approval
- Guest replies are tracked
- Offers are not lost operationally

---

# Phase R5 — Retention learning loop

## Goal

Learn which recovery strategies work.

## Build

- Outcome tracking
- Repeat booking tracking
- Discount redemption tracking
- Guest response sentiment
- Recommendation re-ranking
- Manager preference learning

## Output

- Recovery performance dashboard
- Strategy effectiveness report
- Updated offer policy recommendations

## Success criteria

- The system identifies which outreach strategies improve retention
- Managers trust the recovery recommendations more over time
- The business can measure recovery cost vs repeat-booking benefit

---

## 16. How this connects to the full AI agent system

The complete loop becomes:

```text
Guest conversation
→ issue episode
→ review
→ review–conversation alignment
→ operational recommendation
→ customer recovery case
→ private outreach draft
→ human approval
→ message sent
→ guest response / repeat booking tracked
→ system learns
```

This creates two action tracks from the same intelligence:

### Track A — Fix the property

Examples:

- Repair bathroom ventilation
- Improve parking guide
- Add better task lighting
- Upgrade Wi-Fi
- Rewrite listing copy

### Track B — Recover the customer

Examples:

- Send apology
- Confirm fix
- Offer future discount
- Invite direct booking
- Escalate serious complaint
- Nurture happy guest

Both tracks should be linked. A recovery message without an operational fix is weak. An operational fix without customer recovery may miss a retention opportunity.

---

## 17. Final narrative

The review-intelligence system tells us what went wrong.

The conversation-history system tells us whether the guest already raised it and how the host responded.

The customer recovery system decides what to say next.

Together, they turn guest feedback into a complete business loop:

```text
Detect issue
→ diagnose cause
→ fix property
→ recover guest
→ learn from outcome
```

This is much stronger than review response automation.

The goal is not to “reply to reviews.”

The goal is to **protect guest trust, increase repeat bookings, reduce preventable complaints, and help property managers act like professional hospitality operators.**

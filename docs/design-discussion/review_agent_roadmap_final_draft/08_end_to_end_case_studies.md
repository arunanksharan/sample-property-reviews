# 08 — End-to-End Case Studies

## Purpose

This file shows how the full system should behave in realistic scenarios.

Each case connects:

```text
Listing promise
+ guest conversation
+ review
+ ratings
+ diagnosis
+ property action
+ customer recovery
+ metrics
+ learning loop
```

---

# Case Study 1 — Parking confusion not reported during stay

## Scenario

Property:

```text
Melbourne Laneway Loft
Urban loft, CBD access, parking mentioned in listing or arrival instructions
```

Guest journey:

- Guest arrives by car.
- Parking exists but is tight and unclear.
- Guest does not message during stay.
- Guest leaves a positive review but says parking was tighter than expected.

Review evidence:

```text
“Parking was tighter than expected for a larger car, so clearer guidance on the best spot to use would help future guests.”
```

## Extraction

Review aspect:

```json
{
  "aspect_category": "parking_access",
  "aspect_subcategory": "parking_instructions_vehicle_size",
  "polarity": "negative",
  "severity": "medium",
  "actionability": "high",
  "evidence_snippet": "Parking was tighter than expected for a larger car",
  "action_category": "guest_communication",
  "suggested_action": "Add parking photos and vehicle-size guidance to the pre-arrival message."
}
```

Conversation alignment:

```text
review_complaint_not_in_chat
```

Interpretation:

- Guest did not report issue during stay.
- This is likely not a slow-response support failure.
- It is likely a pre-arrival instruction / expectation-setting issue.
- Future guests can be helped before arrival.

## Recommendation card

```text
Action:
Add photo-based parking instructions and vehicle-size guidance to the pre-arrival message.

Why:
Guests mention parking tightness or confusion in reviews, but do not raise it during the stay. This suggests the issue is discovered at arrival and should be prevented through clearer pre-arrival guidance.

Evidence:
- “Parking was tighter than expected for a larger car...”

Cost:
Low

Impact:
Medium

Confidence:
High if ≥ 3 supporting mentions; early signal if fewer.

Suggested owner:
Guest communications / operations

Caveat:
This mainly affects guests arriving by car.
```

## Operational task

```text
Create parking guide:
- exact parking bay photo
- map pin
- vehicle-size note
- backup parking option
- include in house manual
- send 24 hours before arrival
```

## Customer recovery

If review is positive overall:

Strategy:

```text
Thank-you + minor improvement acknowledgment
```

Private message:

```text
Thank you for staying with us and for the helpful note about parking. We’re glad you enjoyed the location, and we’re improving the pre-arrival parking instructions so future guests have clearer guidance before they arrive.
```

Offer:

```text
No discount by default.
```

## Metrics to track

- future parking complaint recurrence
- guest questions about parking before arrival
- check-in friction
- rating-text discrepancy
- manager acceptance
- task completion

## Learning

If parking complaints drop after adding guide:

```text
Mark action type as effective for urban properties with parking complexity.
```

---

# Case Study 2 — Wi-Fi complaint raised during stay and repeated in review

## Scenario

Property:

```text
Perth Sunset Villa
Family-friendly villa, listed with practical amenities and possibly strong Wi-Fi expectations
```

Conversation:

```text
Guest: “The Wi-Fi is very slow when all our devices are connected.”
Host: “Please try restarting the router.”
Guest: “It is still slow in the evening.”
Host: no further response for several hours.
```

Review:

```text
“The Wi-Fi was generally good, but it slowed down briefly during the evening when several devices were connected.”
```

## Conversation issue episode

```json
{
  "issue_category": "wifi",
  "issue_subcategory": "multi_device_evening_slowdown",
  "first_reported_at": "2025-02-01T19:10:00Z",
  "host_response_minutes": 22,
  "resolution_status": "partially_resolved",
  "guest_sentiment_initial": "negative",
  "guest_sentiment_after_response": "neutral",
  "severity": "medium"
}
```

## Alignment

```json
{
  "alignment_type": "chat_complaint_repeated_in_review",
  "resolution_status_before_review": "partially_resolved",
  "preventability": "high",
  "service_recovery_state": "failed_or_incomplete_recovery"
}
```

## Diagnosis

This is not only a Wi-Fi amenity problem.

It is also:

- a capacity problem
- potentially a long-stay / multi-device guest issue
- an incomplete troubleshooting process
- a possible promise gap if listing claims strong Wi-Fi

## Recommendation card

```text
Action:
Test evening Wi-Fi load and upgrade router/mesh coverage if needed.

Why:
The issue was raised during the stay and repeated in the review, suggesting current troubleshooting did not fully resolve guest experience.

Conversation evidence:
- “The Wi-Fi is very slow when all our devices are connected.”
- “It is still slow in the evening.”

Review evidence:
- “It slowed down briefly during the evening when several devices were connected.”

Cost:
Medium

Impact:
Medium-high

Confidence:
High

Suggested owner:
Maintenance / operations

Customer recovery:
Recommended if review rating is neutral/negative or guest was materially affected.
```

## Operational task

```text
- Run Wi-Fi speed test in evening peak window.
- Test with 5+ connected devices.
- Add mesh node if bedroom/living area weak.
- Update house manual with router instructions.
- If listing says “fast Wi-Fi,” verify speed before keeping claim.
```

## Customer recovery

If guest gave low/neutral review:

Strategy:

```text
Service recovery apology + fix confirmation + optional low future-stay discount
```

Message:

```text
Thank you for flagging the Wi-Fi issue. We’re sorry it affected your stay, especially with multiple devices connected in the evening. We are testing the router under higher load and reviewing whether an upgrade is needed. We appreciate you bringing this to our attention and would be glad to welcome you back under a better setup.
```

Offer:

```text
Low future-stay discount if direct-retention policy allows.
```

## Metrics

- Wi-Fi complaint recurrence
- in-stay Wi-Fi message rate
- value rating
- long-stay satisfaction
- repeat booking after outreach
- router upgrade completion

---

# Case Study 3 — Bathroom ventilation complaint in chat and review

## Scenario

Conversation:

```text
Guest: “The bathroom fan is loud and the room stays damp after showers.”
Host: “Thanks for letting us know.”
No maintenance action recorded.
```

Review:

```text
“The bathroom ventilation was a little loud at night, and the room took some time to dry after showers.”
```

## Diagnosis

The same complaint appears privately and publicly.

This means:

- issue was known during the stay
- no visible resolution happened
- guest repeated the issue in review
- operational process failed to convert known complaint into maintenance action

## Recommendation

```text
Action:
Create bathroom ventilation maintenance task and add a post-shower drying check to the cleaning inspection.

Why:
Bathroom ventilation was mentioned during the stay and repeated in the review, suggesting the issue remains active.

Evidence:
- Chat: “bathroom fan is loud”
- Review: “bathroom ventilation was a little loud... room took time to dry”

Cost:
Medium

Impact:
Medium

Confidence:
High

Owner:
Maintenance + cleaning operations
```

## Customer recovery

If review is positive but complaint specific:

```text
Thank-you + improvement acknowledgment
```

If review neutral/negative:

```text
Apology + fix confirmation
```

No discount unless issue was severe or materially affected stay.

## Metrics

- bathroom ventilation complaint recurrence
- maintenance task completion
- cleaning checklist compliance
- review sentiment after repair
- future issue status: active → monitoring → resolved

---

# Case Study 4 — Complaint in chat but absent from review

## Scenario

Conversation:

```text
Guest: “We can’t find the lockbox.”
Host responds in 3 minutes with annotated photo.
Guest: “Found it, thank you!”
```

Review:

```text
“Great check-in and very responsive host.”
```

## Interpretation

This is a successful service recovery.

The system should not treat the issue as a major negative property problem. It should treat it as:

- minor check-in friction
- strong communication recovery
- useful training example

## Recommendation

```text
Action:
Add the annotated lockbox photo to the standard check-in guide.

Why:
Although the issue did not appear in the review, the conversation shows a preventable check-in question. The host recovered well, but the same question can be prevented.

Cost:
Low

Impact:
Medium

Confidence:
Medium
```

## Customer recovery

No apology needed.

Potential thank-you message if guest gave positive review:

```text
Thank you for the kind review. We’re glad the check-in support was helpful.
```

No discount.

## Metrics

- future lockbox questions
- check-in rating
- host response time
- reduced support load

---

# Case Study 5 — Negative review with no prior complaint

## Scenario

Conversation:

```text
No relevant complaint during stay.
```

Review:

```text
“The apartment was nice but morning street noise made it hard to sleep.”
```

## Diagnosis

This is not a support response failure. It is likely:

- expectation mismatch
- listing copy issue
- location/noise reality
- comfort/sleep-quality issue

## Recommendation

```text
Action:
Update listing copy to mention possible morning street noise and add earplugs/white-noise machine.

Why:
Noise complaint appeared only after stay, suggesting guests may not expect it before arrival.

Evidence:
- “Morning street noise made it hard to sleep.”

Cost:
Low to medium

Impact:
Medium

Owner:
Listing + operations
```

## Customer recovery

If rating low:

```text
Apology + expectation/fix confirmation + optional low goodwill.
```

Message:

```text
We’re sorry the morning noise affected your sleep. We are updating the listing guidance and adding practical support for light sleepers so future guests can make a better-informed decision.
```

## Metrics

- noise complaint recurrence
- value rating
- booking conversion after disclosure
- cancellation rate
- guest satisfaction among light sleepers

---

# Case Study 6 — Safety issue

## Scenario

Conversation:

```text
Guest: “The front door lock is not closing properly.”
```

Review:

```text
“We felt unsafe because the lock was not working.”
```

## Diagnosis

This bypasses normal ranking.

## Action

```text
Immediate escalation:
- senior manager
- maintenance
- safety inspection
- human-approved customer communication
```

No automated discount or apology should be sent before human review.

## Guardrails

- classify as safety/security
- create urgent task
- block auto-send
- require senior approval
- document resolution
- monitor future recurrence

## Metrics

- safety escalation time
- repair completion time
- customer contact time
- recurrence
- compliance audit

---

# Case Study 7 — Offer decisioning for retention

## Scenario

Guest booked direct, stayed 5 nights, complained about unresolved Wi-Fi during stay, left 3-star review, and said property location was good.

Signals:

- direct booking
- high stay length
- fixable issue
- valid complaint
- positive property sentiment remains
- unresolved service episode

## Recovery strategy

```text
Apology + fix confirmation + future direct booking credit
```

Offer level:

```text
Low or medium, depending on policy
```

Message should not mention review change.

## Operational action

```text
Upgrade/test Wi-Fi before sending fix-confirmation wording.
```

## Metrics

- guest reply
- offer redemption
- repeat direct booking
- Wi-Fi complaint recurrence
- future review sentiment

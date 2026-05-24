# 02 — Unified Data Model and Schemas

## 1. Purpose

This file defines the canonical data model for the full SympleHost AI guest-experience system.

The system must connect:

```text
Property/listing data
+ reservation data
+ guest review data
+ conversation messages
+ issue episodes
+ review–conversation alignment
+ recommendation cards
+ operational tasks
+ customer recovery cases
+ outreach messages
+ manager decisions
+ outcomes
```

A consistent schema prevents the three tracks — review intelligence, conversation history, and customer recovery — from becoming disconnected products.

## 2. Core entity graph

```text
PropertyManager
  └── Property
        ├── ListingSnapshot
        ├── Reservation
        │     ├── GuestProfile
        │     ├── ConversationThread
        │     │     └── Message
        │     │           └── MessageIssue
        │     ├── IssueEpisode
        │     ├── Review
        │     │     └── ReviewAspect
        │     ├── ReviewConversationAlignment
        │     ├── RecommendationCard
        │     │     └── ManagerDecision
        │     ├── OperationalTask
        │     ├── CustomerRecoveryCase
        │     │     └── OutreachMessage
        │     └── OutcomeRecord
```

## 3. Property manager

```json
{
  "property_manager_id": "pm_001",
  "name": "Example Property Management",
  "market": "Australia",
  "portfolio_size": 48,
  "default_language": "en",
  "timezone": "Australia/Sydney",
  "created_at": "2026-05-24T00:00:00Z"
}
```

Important future fields:

- response SLA
- offer approval limits
- owner approval workflow
- brand tone guide
- communication channels
- PMS integration source

## 4. Property

```json
{
  "property_id": "prop_bondi_coastal_apartment",
  "property_manager_id": "pm_001",
  "property_name": "Bondi Coastal Apartment",
  "property_type": "Apartment",
  "city": "Sydney",
  "region": "New South Wales",
  "country": "Australia",
  "neighborhood": "Bondi",
  "latitude": -33.8908,
  "longitude": 151.2743,
  "bedrooms": 2,
  "bathrooms": 1,
  "max_guests": 4,
  "average_nightly_rate_aud": 365,
  "active": true
}
```

## 5. Listing snapshot

The listing can change over time. Store snapshots.

```json
{
  "listing_snapshot_id": "ls_001",
  "property_id": "prop_bondi_coastal_apartment",
  "source_platform": "Airbnb",
  "captured_at": "2026-05-24T00:00:00Z",
  "title": "Bright coastal apartment near Bondi",
  "description": "Bright coastal apartment with beach access, compact work space, and easy access to restaurants.",
  "amenities": ["Ocean view", "Balcony", "Pool", "Secure parking", "Kitchenette", "Air conditioning"],
  "claims": ["beach access", "compact workspace", "secure parking"],
  "photos": []
}
```

Why snapshot:

- Review may reference a listing claim from the past.
- A later listing rewrite should not erase what was promised at time of booking.
- Promise-gap analysis needs listing-as-of-booking where possible.

## 6. Reservation / stay

```json
{
  "reservation_id": "res_001",
  "reservation_number": "RES-202501-00001",
  "property_id": "prop_bondi_coastal_apartment",
  "guest_id_hash": "guest_hash_001",
  "source_platform": "Booking.com",
  "booking_date": "2025-01-01",
  "check_in": "2025-01-05",
  "check_out": "2025-01-07",
  "nights": 2,
  "guests": 2,
  "price_paid_aud": null,
  "cleaning_fee_aud": null,
  "discount_applied_aud": null,
  "status": "completed"
}
```

Derived fields:

```text
stay_length_bucket = short / medium / long
occupancy_density = guests / max_guests
booking_channel_type = OTA / direct
```

## 7. Guest profile

Use minimal, privacy-preserving fields.

```json
{
  "guest_id_hash": "guest_hash_001",
  "first_seen_at": "2025-01-05",
  "repeat_guest": false,
  "direct_booking_count": 0,
  "total_completed_stays": 1,
  "do_not_contact": false,
  "deletion_requested": false
}
```

Avoid storing or inferring:

- ethnicity
- nationality where not operationally necessary
- religion
- political views
- health status
- sensitive demographic attributes
- inferred wealth

## 8. Review

```json
{
  "review_id": "rev_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "guest_id_hash": "guest_hash_001",
  "source_platform": "Booking.com",
  "review_date": "2025-01-09",
  "rating_overall": 5,
  "cleanliness_rating": 5,
  "accuracy_rating": 5,
  "communication_rating": 5,
  "location_rating": 5,
  "checkin_rating": 5,
  "value_rating": 4,
  "review_title": "Great location and easy check-in",
  "what_guest_liked": "The location made the stay easy...",
  "what_guest_didnt_like": "The only improvement would be adding task lighting.",
  "additional_comments": "Overall, we would recommend the property.",
  "full_review": "What guest liked: ...",
  "provided_sentiment": "positive",
  "response_status": "not_responded"
}
```

## 9. Review aspect

The central object for review intelligence.

```json
{
  "review_aspect_id": "ra_001",
  "review_id": "rev_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "aspect_category": "lighting",
  "aspect_subcategory": "task_lighting",
  "polarity": "negative",
  "severity": "low",
  "actionability": "high",
  "specificity": "high",
  "evidence_snippet": "adding a little more task lighting in the living area",
  "evidence_field": "what_guest_didnt_like",
  "action_category": "amenity_capex",
  "suggested_action": "Add a task lamp or floor lamp in the living area.",
  "confidence": "high",
  "extractor_version": "review_extractor_v0.1",
  "extracted_at": "2026-05-24T00:00:00Z"
}
```

## 10. Conversation thread

```json
{
  "thread_id": "thread_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "guest_id_hash": "guest_hash_001",
  "channel": "airbnb",
  "started_at": "2025-01-05T15:12:00Z",
  "ended_at": "2025-01-07T10:32:00Z",
  "message_count": 12,
  "source_system": "airbnb_messages"
}
```

## 11. Message

```json
{
  "message_id": "msg_001",
  "thread_id": "thread_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "guest_id_hash": "guest_hash_001",
  "channel": "airbnb",
  "sender_type": "guest",
  "sent_at": "2025-01-05T18:42:00Z",
  "message_text": "Hi, the Wi-Fi is not working in the bedroom.",
  "language": "en",
  "attachments": [],
  "source_system": "airbnb_messages"
}
```

## 12. Message issue

Message-level extraction.

```json
{
  "message_issue_id": "mi_001",
  "message_id": "msg_001",
  "thread_id": "thread_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "issue_category": "wifi",
  "issue_subcategory": "bedroom_signal",
  "message_intent": "complaint",
  "polarity": "negative",
  "urgency": "medium",
  "severity": "medium",
  "requires_action": true,
  "evidence_snippet": "the Wi-Fi is not working in the bedroom",
  "confidence": "high",
  "extractor_version": "message_issue_extractor_v0.1"
}
```

Message intent values:

- complaint
- question
- request
- praise
- clarification
- cancellation/refund
- safety/security
- noise/party risk
- logistics
- other

## 13. Issue episode

Groups related messages into one operational episode.

```json
{
  "issue_episode_id": "ie_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "guest_id_hash": "guest_hash_001",
  "issue_category": "wifi",
  "issue_subcategory": "bedroom_signal",
  "first_guest_message_id": "msg_001",
  "first_reported_at": "2025-01-05T18:42:00Z",
  "first_host_response_at": "2025-01-05T18:55:00Z",
  "host_response_minutes": 13,
  "resolved_at": "2025-01-05T20:10:00Z",
  "time_to_resolution_minutes": 88,
  "resolution_status": "resolved",
  "escalated": false,
  "compensated": false,
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

Resolution status values:

- unresolved
- resolved
- partially_resolved
- unknown
- escalated
- compensated
- guest_abandoned

## 14. Review–conversation alignment

Connects public review outcome to private service journey.

```json
{
  "alignment_id": "align_001",
  "review_id": "rev_001",
  "review_aspect_id": "ra_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "matching_issue_episode_id": "ie_001",
  "alignment_type": "chat_complaint_repeated_in_review",
  "review_evidence": "Wi-Fi was unreliable in the bedroom.",
  "conversation_evidence": "Hi, the Wi-Fi is not working in the bedroom.",
  "resolution_status_before_review": "partially_resolved",
  "preventability": "high",
  "service_recovery_state": "failed_recovery",
  "recommended_internal_action": "Upgrade router or add mesh node.",
  "recommended_customer_action": "Send apology and mention fix before offering repeat-stay incentive.",
  "confidence": "high"
}
```

Alignment types:

- chat_complaint_repeated_in_review
- chat_complaint_absent_from_review
- review_complaint_not_in_chat
- chat_resolved_positive_review
- chat_unresolved_negative_review
- positive_chat_negative_review
- negative_chat_positive_review
- no_chat_no_issue

## 15. Recommendation card

```json
{
  "recommendation_id": "rec_001",
  "property_id": "prop_bondi_coastal_apartment",
  "reservation_id": null,
  "recommendation_type": "property_action",
  "action": "Add a photo-based parking guide to the pre-arrival message.",
  "why": "Guests repeatedly mention parking is tight or unclear.",
  "evidence": [
    {
      "source_type": "review",
      "source_id": "rev_001",
      "snippet": "Parking was tighter than expected."
    },
    {
      "source_type": "conversation",
      "source_id": "msg_019",
      "snippet": "Where exactly do we park?"
    }
  ],
  "affected_aspects": ["parking"],
  "supporting_count": 5,
  "cost_level": "low",
  "impact_level": "medium",
  "confidence": "high",
  "owner_role": "guest_communications",
  "counterargument": "This only affects guests arriving by car.",
  "evidence_type": "property_review_and_conversation_evidence",
  "status": "pending_decision"
}
```

## 16. Manager decision

```json
{
  "decision_id": "dec_001",
  "recommendation_id": "rec_001",
  "manager_id": "mgr_001",
  "decision": "accepted",
  "decision_at": "2026-05-24T00:00:00Z",
  "reason_free_text": "Agree, guests keep asking about this.",
  "reason_tag": "valid_issue",
  "target_completion_date": "2026-06-01"
}
```

Decision values:

- accepted
- rejected
- deferred
- escalated
- completed
- reopened

## 17. Operational task

```json
{
  "task_id": "task_001",
  "recommendation_id": "rec_001",
  "property_id": "prop_bondi_coastal_apartment",
  "task_type": "guest_instruction_update",
  "title": "Create parking guide with photos.",
  "description": "Add exact parking bay photos, vehicle-size guidance, and backup parking options.",
  "owner_role": "operations",
  "priority": "medium",
  "status": "open",
  "created_at": "2026-05-24T00:00:00Z",
  "completed_at": null
}
```

## 18. Customer recovery case

```json
{
  "recovery_case_id": "case_001",
  "review_id": "rev_001",
  "reservation_id": "res_001",
  "property_id": "prop_bondi_coastal_apartment",
  "guest_id_hash": "guest_hash_001",
  "case_type": "private_post_review_outreach",
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
  "draft_status": "pending_human_review",
  "created_at": "2026-05-24T00:00:00Z"
}
```

## 19. Outreach message

```json
{
  "outreach_message_id": "out_001",
  "recovery_case_id": "case_001",
  "guest_id_hash": "guest_hash_001",
  "channel": "email",
  "message_type": "private_post_review_recovery",
  "draft_text": "Hi Alex, thank you for staying with us...",
  "policy_lint_passed": true,
  "requires_approval": true,
  "approval_status": "pending",
  "approved_by": null,
  "sent_at": null,
  "guest_replied_at": null,
  "guest_reply_sentiment": null
}
```

## 20. Outcome record

```json
{
  "outcome_id": "outcome_001",
  "entity_type": "recommendation",
  "entity_id": "rec_001",
  "property_id": "prop_bondi_coastal_apartment",
  "outcome_window_start": "2026-06-01",
  "outcome_window_end": "2026-09-01",
  "metric_name": "parking_complaint_recurrence",
  "baseline_value": 5,
  "post_action_value": 1,
  "interpretation": "Complaint recurrence decreased after action; not causal without controls.",
  "confidence": "medium"
}
```

## 21. Evidence and audit fields

Every extracted record should carry:

- source text reference
- evidence snippet
- extractor version
- model version
- prompt version
- extracted timestamp
- validation status
- reviewer override if manually corrected

## 22. Data quality checks

Run on every batch:

| Check | Failure action |
|---|---|
| Duplicate IDs | Stop pipeline |
| Review without property | Quarantine |
| Conversation without reservation | Quarantine |
| Evidence snippet not found in source | Reject extraction |
| Missing timestamp | Quarantine message |
| PII present in LLM payload | Block call |
| Invalid enum | Retry / reject |
| Schema validation failure | Retry / reject |
| Manager card without evidence | Do not surface |
| Recovery message with review-change incentive | Block send |

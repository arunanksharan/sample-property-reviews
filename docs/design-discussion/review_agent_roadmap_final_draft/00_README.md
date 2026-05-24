# SympleHost AI Agent Roadmap Pack

## Purpose

This documentation pack consolidates the three previously separate design tracks into one coherent roadmap:

1. **Review Intelligence** — analyse property reviews, ratings, and listing attributes.
2. **Conversation History Intelligence** — connect in-stay guest messages, complaints, host replies, and issue resolution with later reviews.
3. **Post-Review Customer Recovery & Retention** — decide how to communicate with guests after reviews, including apology, fix confirmation, goodwill gestures, future discounts, and retention workflows.

The product should not be framed as “LLM sentiment analysis.”

It should be framed as:

> **An AI guest-experience decision system for property managers that connects listings, reviews, conversations, ratings, operational actions, customer recovery, and outcome learning into one continuous improvement loop.**

## Canonical operating loop

```text
Guest stay happens
→ guest messages / complaints may occur
→ host responds / issue is resolved or not
→ guest leaves review
→ system extracts review aspects
→ system extracts conversation issue episodes
→ system aligns review complaints with conversation history
→ system diagnoses property, process, and communication failures
→ system creates manager-facing recommendation cards
→ system creates customer recovery cases where appropriate
→ manager accepts / rejects / defers
→ operational task and/or guest outreach is completed
→ future reviews, conversations, repeat bookings, and outcomes are tracked
→ system learns which actions work
```

## File map

| File | Purpose |
|---|---|
| `00_README.md` | Orientation and canonical system loop |
| `01_master_product_roadmap.md` | End-to-end product vision and phased roadmap |
| `02_data_model_and_schemas.md` | Unified data contracts across review, conversation, action, recovery, and outcomes |
| `03_methodology_and_intelligence_layers.md` | Quantitative, interpretative, and feedback-learning methodology |
| `04_agent_workflows_and_operating_system.md` | Practical agent/component design and implementation flow |
| `05_metrics_evaluation_and_guardrails.md` | Evaluation plan, metrics, thresholds, safety, policy, and quality gates |
| `06_conversation_review_alignment.md` | Deep roadmap for integrating in-stay conversation history with reviews |
| `07_post_review_recovery_and_retention.md` | Deep roadmap for private post-review customer outreach and retention |
| `08_end_to_end_case_studies.md` | Detailed case studies showing how the system behaves |
| `09_implementation_backlog.md` | Build plan, milestones, sprint breakdown, dependencies, and deliverables |

## Core product rule

> **If an output does not help a property manager decide what to do next, it is noise.**

Every manager-facing output should answer:

```text
What happened?
Why does it matter?
What should we do?
What evidence supports it?
How confident are we?
What should we not overclaim?
Did the action work?
```

## What this pack deliberately avoids in v1

Do not begin by building:

- A giant agent zoo
- Fully autonomous review replies
- Fully autonomous customer recovery messages
- Dollar-level ROI estimates without cost/revenue data
- Strong causal claims without intervention outcomes
- Complex graph databases before simple tables prove value
- Dynamic pricing automation
- Cross-customer benchmarking
- Guest-facing marketplace/search products

The v1 goal is narrower:

> **Build a trusted review + conversation → diagnosis → recommendation → human decision → outcome tracking loop.**

Once this loop is trusted, richer prediction, pricing, owner reporting, and automation can be layered on.

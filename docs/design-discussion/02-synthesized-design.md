# Symple-Host — Design

> Working design doc. The longer exploratory thinking lives in
> `01-deep-design-exploration.md`. This doc is what we build to.

---

## What we're building

A property manager runs anywhere from a handful to a thousand short-term
rental listings. Reviews are the richest signal they have about what's
actually happening on the ground — but reviews are messy natural language,
and a spreadsheet can't read them.

Symple-Host is an **AI decision copilot for property managers**. It reads
guest reviews and listing data, and turns each review into something the
manager can act on: a maintenance task, a listing edit, a pricing flag, a
guest message, an owner report.

One rule overrides everything else:

> **If an output doesn't shorten time-to-decision or change a decision, it's
> noise.**

Every recommendation comes with evidence (verbatim quotes from real reviews),
a confidence level, and a clear action. No essays, no dashboards-for-the-sake-
of-dashboards.

---

## The dataset we start with

- **16 properties** — id, name, type, city, neighborhood, amenities
  (pipe-separated list), bedrooms, capacity, nightly rate.
- **420 reviews** — seven numeric ratings (overall + cleanliness, accuracy,
  communication, location, check-in, value), free text already split into
  `whatGuestLiked`, `whatGuestDidntLike`, and `additionalComments`, the
  source platform (Airbnb, Booking.com, …), dates, and a coarse sentiment
  label.

About 26 reviews per property on average. Enough to find patterns *per
property*. Thin for portfolio-wide statistical claims — we'll respect that.

This is a starting sample. The production version will eventually add
reservation data, maintenance logs, pricing history, in-stay chat, and
photos. We don't wait for them to start.

---

## The three layers

This is the core architecture. Each layer has a clear job, and they feed
into each other.

### Layer 1 — Interpretative layer (LLM reads reviews)

This is where messy text becomes structured. For every review, the LLM
extracts a list of aspects. Each aspect is tagged with:

- **Category** — cleanliness, parking, wifi, check-in, location, value, …
- **Polarity** — positive, negative, neutral
- **Severity** — annoyance, problem, blocker
- **Evidence** — the exact words from the review that justify the tag
- **Action category** — operations / maintenance / listing / pricing /
  communication

One review produces many aspect records. Example:

> *"The place was lovely, but parking was confusing and the bathroom stayed
> damp."*

becomes three aspects: positive on design, negative on parking guidance,
negative on bathroom ventilation. Each carries its own quote.

The interpretative layer is also where we draft responses to guests, write
owner reports, and rewrite listing copy. The same engine, different prompts.

### Layer 2 — Quantitative layer (stats on the structured aspects)

Once aspects are extracted, they're just data. We aggregate, correlate, and
rank without LLMs. Plain SQL and pandas.

The quantitative layer does four things at MVP:

1. **Per-property aspect summary** — what are the most frequent complaints
   and praises, weighted by severity and how recent they are?
2. **Amenity–aspect promise gap** — cross-reference each property's listed
   amenities with what reviews actually say. Listed "fast wifi" + repeated
   wifi complaints = a promise gap. Repeated workspace requests + no
   workspace listed = latent demand worth a capex conversation.
3. **Rating-text discrepancy** — a 5-star review with complaint text is a
   courtesy rating. We compute the gap between the numeric rating and the
   text-derived sentiment, and surface it. These are hidden problems that
   averages miss.
4. **Driver analysis** — which subratings move with overall? Which amenities
   are associated with stronger value ratings? *Associated*, not *caused* —
   see "How we talk about claims" below.

### Layer 3 — Feedback-based learning loop

The interpretative and quantitative layers produce ranked recommendations.
The manager hits **Accept**, **Reject**, or **Defer** on each, with an
optional reason. That decision is captured, and the system gets smarter.

Three things happen with feedback:

- **Suppression.** Rejected recommendation types are silenced for that
  property until the manager manually reopens them. If the owner has refused
  to buy coffee pods three times, the system stops asking.
- **Outcome tracking.** For accepted recommendations, we watch the next
  batch of reviews. Did the issue surface less often? More? At all?
- **Learning over time.** As the corpus of decisions grows, we learn which
  recommendation types this manager values and which interventions actually
  move ratings.

We're honest about scope here. At MVP, "learning" is a suppression list and
outcome counts. The serious causal measurement — controlled comparisons,
attribution windows — comes later, once we have enough decisions and
outcomes to be honest about effect sizes.

The full picture:

```
Raw reviews ──► Interpretative layer ──► Aspect records
                                              │
                                              ▼
                                       Quantitative layer
                                              │
                                              ▼
                                    Ranked recommendations
                                              │
                                              ▼
                                    Manager accepts/rejects
                                              │
                                              ▼
                                       Learning loop
                                              │
                          (feeds back into ranking and synthesis)
```

---

## The central data object

Everything downstream eats this record. One per aspect, many per review:

```json
{
  "review_id": "rev_001",
  "property_id": "prop_adelaide_garden_cottage",
  "aspect": "parking",
  "polarity": "negative",
  "severity": "medium",
  "evidence": "Parking was tighter than expected",
  "action_category": "guest_instructions",
  "suggested_action": "Add parking photos and vehicle-size guidance to the pre-arrival message"
}
```

The evidence quote is verifiable against the source review. No quote means
no claim — the system refuses to surface anything it can't cite.

---

## What the manager actually sees

Not a dashboard. A stack of recommendation cards, one per decision:

```
Action       Add parking photos and vehicle-size guidance to the
             check-in guide.

Why          Parking confusion is the most frequent complaint at this
             property — appears in 6 of the last 30 reviews.

Evidence     "Parking was tighter than expected, clearer guidance
              would help."
             "We didn't know which spot was ours…"
             (+ 4 more)

Affected     Adelaide Garden Cottage

Cost         Low — 30 minutes to update the guide

Impact       Medium — reduces check-in friction

Confidence   Medium-high

Counterpoint Only affects guests arriving by car.

Owner        Guest comms

Decision     [ Accept ]   [ Reject ]   [ Defer ]
```

This is the product. The manager works through the stack, makes calls, moves
on.

---

## The MVP pipeline

Seven steps, end to end:

1. **Validate** — load the data, check foreign keys, hash guest emails,
   produce a one-page schema report.
2. **Extract aspects** — run each review through the interpretative layer
   with strict JSON output and a citation check (the quoted evidence must
   actually appear in the source review).
3. **Aggregate** — group aspects per property and category; compute
   rating-text discrepancy per review.
4. **Build the promise-gap view** — cross-reference each property's
   amenities with its aspect frequencies.
5. **Rank and synthesize** — for each property, generate the top three
   recommendations with evidence and counterpoint.
6. **Show and capture** — render recommendation cards in a simple UI;
   capture Accept / Reject / Defer with reason.
7. **Track outcomes** — when new reviews arrive, check whether the issue is
   mentioned less often after an accepted recommendation.

That's the whole loop. Five components do the work: ingestion + validation,
aspect extractor, aggregator, recommendation synthesizer, decision capture.

We deliberately don't build a critic agent, a sequencer, a memory keeper, or
a portfolio Q&A in v1. They come later if the data justifies them.

---

## Three things we don't compromise on

**Evidence or nothing.** Every claim is backed by a verbatim quote from a
real review. If the model can't cite, it doesn't speak.

**Honest language.** Until we have real intervention outcomes, we say
"associated with," "appears in N reviews," "suggests," "hypothesis." We
don't say "causes," "proves," or invent dollar uplifts. This is enforced in
the synthesizer's prompt.

**Defensive basics.** Guest emails are hashed before any LLM call. Review
text is passed to the model as data, not instructions (so a guest can't
write "ignore previous instructions" and hijack the output). LLM outputs are
validated against a schema and retried on failure. A small hand-labeled set
of ~50 reviews is built in week one so we can tell if the extractor is
getting worse over time.

---

## Phasing

**Now** — the MVP pipeline above, run end-to-end on the 420 reviews. Demo
to one real property manager. Collect their decisions on the top
recommendations.

**Next** — once we have decisions to learn from: drafted public review
responses, manager Q&A over the corpus, real intervention-outcome tracking,
a critic check on recommendations before they ship.

**Later** — portfolio-wide ranking across many managers, predictive
review-risk for upcoming stays, listing-copy rewriting with A/B tests,
cross-tenant pattern learning, PMS integrations.

We promote items from "Later" to "Next" when something concrete triggers it:
acceptance rate stalls and we need a quality lift, enough
intervention-outcome pairs to talk about effects honestly, a second
customer that exposes multi-tenant needs.

---

## What we still need to confirm with the customer

1. **Who's the user?** A single host with one property, a manager with
   10–100, or an enterprise with 1000+? The product looks different at each
   scale.
2. **What's the success metric?** Rating uplift, response SLA, time saved
   per week, churn reduction, or acquisition support? Different metrics →
   different ranking priorities.
3. **What additional data can be added?** Reservation-level data is the
   biggest unlock (it tells us which guests didn't review, which lets us
   correct selection bias). Maintenance logs close the outcome loop. Pricing
   history enables real value modeling.

# Symple-Host — Deep Design Exploration

> Status: design-only, pre-implementation. This is a written record of the
> reasoning before any code is written. Goal: produce a smart set of
> LLM-powered AI agents to analyse property listings + guest reviews and turn
> them into ranked, evidence-backed, decision-ready recommendations for
> property managers.

---

## 1. What problem are we *really* solving?

A property manager's job is **deciding where to spend the next dollar and the
next hour**. Every insight we generate has to map to a decision — install AC,
rewrite the listing, fire the cleaner, raise the rate, divest the property,
draft a reply, message the guest. If an output doesn't shorten time-to-decision
or change a decision, it's noise. That framing separates a "review dashboard"
from an agent system.

This reframes everything that follows: not "extract aspects," but **"produce
decision-ready, evidence-backed, ranked recommendations the PM can
accept/reject in one click."**

---

## 2. The dataset we have

**Properties (15 rows)** — `propertyId, propertyName, description,
propertyType, city, region, country, neighborhood, latitude, longitude,
amenities` (pipe-separated, ~8 per property), `bedrooms, bathrooms, maxGuests,
averageNightlyRateAud`.

**Reviews (419 rows)** — `reviewId, propertyId, reservationId, guestName,
guestEmail, sourcePlatform`, seven numeric ratings (overall + cleanliness,
accuracy, communication, location, checkin, value), `reviewDate, checkIn,
checkOut, nights, guests, reviewTitle, whatGuestLiked, whatGuestDidntLike,
additionalComments, fullReview, sentiment, responseStatus`.

Key shape: structured ratings **AND** text already split into liked / disliked
/ extra. That split makes aspect extraction much cleaner than a single blob.

419 reviews ÷ 15 properties ≈ **28 reviews/property**. Enough for qualitative
aspect mining; thin for statistical claims. Plan accordingly.

---

## 3. The two-layer idea, refined into three concerns

The instinct of "ML layer + LLM layer" is right, but the real power is in how
they feed each other:

### Layer A — Quantitative analytics (no LLM)
Classical stats / ML on the structured side. One-hot the amenities, regress
against ratings, look at platform bias, seasonality, stay length vs rating,
price vs value-rating, response-rate vs subsequent rating. A gradient-boosted
model with SHAP gives "which property attributes drive overall rating" cheaply
and interpretably.

### Layer B — LLM aspect & insight extraction
Aspect-Based Sentiment Analysis (ABSA) on the three text fields. Per review,
emit structured records:
`[{aspect, polarity, severity, snippet, suggested_action_category}]`. Plus
theme clustering across reviews, summarization, draft responses, manager Q&A.

### Layer C — The feedback loop (the differentiator)
LLM-extracted aspects become **new structured features** that flow back into
Layer A.

Example: LLM tags `parking_complaint=1` on 30 reviews → Layer A correlates
that flag with `amenities contains 'Free parking'` → "parking complaints are
5× higher on properties without dedicated parking, regardless of
neighborhood." Neither layer alone surfaces that; the loop does.

> Most "AI for reviews" projects stop at B. The compounding value is C.

---

## 4. What's easy to under-weight

1. **Selection bias** — only ~30-50% of guests review. Aspect-frequency counts
   are biased toward people who chose to type. The silent-unhappy are
   invisible.
2. **Negative space in reviews** — silence on cleanliness ≠ clean. Absence is
   itself a signal.
3. **Rating-text discrepancy** — "5 stars" + complaint paragraph is a
   *courtesy rating*. The magnitude of discrepancy is one of the most
   predictive features for true satisfaction.
4. **Time-causality** — did responding improve the rating, or did better
   cleaning improve both? Without diff-in-diff design we'll claim causes we
   don't have.
5. **The aspect ontology shouldn't be fixed** — seed with ~12 buckets, but
   cluster the LLM's free-form extractions monthly to discover emerging
   categories (EV charger, noise-cancelling provided, post-COVID hygiene
   language).

---

## 5. Aspect taxonomy (seed)

Hospitality ABSA usually clusters into ~12 buckets:

- **Cleanliness** — linen, bathroom, kitchen, smell, dust
- **Comfort** — bed/mattress, pillows, temperature, noise
- **Check-in/out** — instructions clarity, lockbox, key handover, timing
- **Host communication** — responsiveness, tone, helpfulness
- **Listing accuracy** — photos vs reality, description match, surprises
- **Amenities — present-but-poor** — slow wifi, weak AC, missing utensils
- **Amenities — missing/wanted** — workspace, blackout, BBQ
- **Location** — transit, food, attractions, safety, street noise
- **Maintenance** — broken items, plumbing, appliances
- **Value for money**
- **Safety/security** — locks, detectors, neighborhood
- **Special-segment fit** — family, pets, long-stay, remote work

Per aspect, tag: **polarity, severity (annoyance → blocker), specificity
(vague vs fixable), and action_category** (Ops / Maintenance / Listing copy /
Pricing / Amenity capex). **Severity × frequency** is the prioritization
signal.

---

## 6. Deeper signal extraction

### 6.1 From free text, beyond ABSA

- **Emotion granularity** — delight / frustration / surprise / disappointment
  / gratitude / resignation. Different actions per emotion.
- **Specificity score** — vague praise carries less info than specific. "Loved
  it" vs "the Nespresso machine and rain shower made the morning." Use
  specificity as a confidence weight on the extracted aspect.
- **Comparison mentions** — "better than my last Airbnb in Bondi" is
  competitive intel. Extract separately.
- **Repeat-intent classifier** — "would book again," "won't stay here again,"
  "recommend to friends." Three-way classifier independent of star rating.
  Closer proxy for LTV than rating.
- **Question detection** — guests asking things in reviews ("why no kettle?")
  become FAQ entries / house-manual additions automatically.
- **Politeness/hostility scale** — drives response-tone selection.
- **Hidden compliments** — guests are diplomatic: "Nothing major to complain
  about" buried in `whatGuestDidntLike` is a strong positive.
- **Reading level & code-switching** — proxies for segment (business vs
  casual, native vs ESL); helps tone-match responses and surfaces multilingual
  gaps.
- **Emotional arc within a single review** — "Started rough… but the host
  fixed it." Story-arc detection separates resolved-issue (positive trust
  signal) from unresolved-issue (churn signal).

### 6.2 From the structured ratings

The 7-dim ratings are under-used if you just average them.

- **Rating residuals**: `overall − predicted(overall | 6 subs)`. Positive
  residual = halo / "vibes premium." Negative = hidden dissatisfaction.
  Properties with consistent negative residuals have something the subratings
  aren't capturing.
- **Co-movement matrix per property type** — which subrating most drives
  overall? Likely cleanliness for serviced apartments, location for city
  stays, host communication for first-time-host segment. Tells you *where to
  invest improvement budget per property type*.
- **Rating-text discrepancy as a feature** — combine sentiment(text) − rating.
  Large gaps = courtesy ratings (true sentiment worse than stars) or stoic
  ratings (true sentiment better than stars). Treat ratings as biased
  estimators; use the text-derived score as ground truth.
- **Within-guest rating norming** — if we had reservation history per guest,
  we could de-mean. With emails as IDs, do partial norming for repeat guests.

### 6.3 From metadata most teams ignore

- **`reviewDate − checkOut` = review lag.** Unhappy guests review faster.
  Lag is a sentiment signal and a recency-bias correction.
- **`guests ÷ maxGuests` = occupancy density.** Over-capacity stays correlate
  with cleanliness/wear complaints.
- **Nights distribution** — short stays surface check-in pain; long stays
  surface wifi / workspace / laundry.
- **`guestEmail` entity resolution** — repeat guests across properties create
  a loyalty graph. Repeat-guest reviews are much more trustworthy and
  segmentation-rich.
- **Source platform × rating distribution** — Airbnb 5-star inflation vs
  Booking.com 1–10 scale needs explicit calibration before any cross-platform
  comparison. Without this, every dashboard lies.
- **`lat/lon` enrichment** — pull walk score, transit score, city noise
  complaints, crime, distance to attractions. "Great location" claims become
  measurable.

---

## 7. The amenity↔aspect causal bipartite graph

The highest-value structural idea. Two sides:

- **Amenities** parsed from the pipe-separated list — Pool, AC, Workspace,
  Parking, BBQ, Heating, …
- **Aspects** extracted from reviews — too hot, parking confusion, slow wifi,
  no workspace, etc.

Edges carry weights:

- **Expected positive coverage** — amenity X *should* prevent complaint Y.
  Pool listed → no "no pool" complaint. AC listed → no "too hot" complaint.
- **Promise gaps** — amenity listed AND complaint about it = false
  advertising or broken equipment. ("Has AC" + "AC was weak" → maintenance
  ticket.)
- **Latent demand** — amenity NOT listed AND repeated complaint = capex
  opportunity. (Workspace-complaint frequency × business-traveler share =
  build-vs-skip decision.)
- **Surplus features** — amenity listed but never mentioned = either not used
  or not photographed/marketed. Usually a marketing fix, not removal.

This one graph drives capex, maintenance, listing rewrites, and pricing — four
decision types from one structure.

---

## 8. Counterfactual & uplift simulation

Don't describe; predict effect of action.

- **Synthetic controls** — pair properties that differ only by amenity X to
  estimate X's rating lift. "Properties with workspace in CBD areas average
  +0.4 on value rating vs matched controls."
- **Treatment-effect estimation** from natural experiments — did any property
  add an amenity mid-period? If yes, diff-in-diff. If not, propensity-matched
  cross-section.
- **Suggested A/B tests** — agent proposes the next listing-copy or price test
  with hypothesized lift and required sample size.
- **Divest simulation** — "if we drop the bottom 2 properties, what happens to
  portfolio metrics?" Re-aggregate with exclusion.

---

## 9. Actionable insight categories — grouped by *who acts*

### Operations & maintenance
Ranked per-property issue list. Auto-drafted maintenance tickets. Cleaning
checklist deltas. House-manual edits for recurring confusion (parking, wifi,
appliances).

### Listing & marketing
Listing-accuracy gaps ("guests expected X from photos"). Missing photo
subjects. Copy rewrite per platform. Amenity-discoverability fixes.

### Pricing & revenue
Value-rating ÷ price flags overpriced units. Amenity ROI ("does Pool justify
upkeep?"). Platform mix analysis — which platform brings higher-rated,
higher-paying guests?

### Portfolio strategy
Neighborhood × property-type performance matrix. Amenity-gap analysis
("workspace missing in 40% of Bondi complaints → next acquisition spec").
Under-performer divest list.

### Guest experience
`needs_response` SLA queue with drafted replies in property voice. Pre-arrival
nudge content auto-personalized to address that property's top-3 confusion
points. Repeat-guest detection via email.

### Risk
Safety mentions (detectors, locks) auto-escalated. Party / noise / council
mentions flagged for compliance.

---

## 10. Multi-agent topology

Role-based agents:

1. **Ingestion / normalization** — load, dedupe, language detect, PII scrub.
2. **Aspect extractor** — per-review structured ABSA (LLM + strict JSON
   schema / Pydantic).
3. **Theme aggregator** — clusters aspects across reviews per property and
   portfolio; spots recurring themes.
4. **Insight synthesizer** — turns themes + ML correlations into ranked,
   actionable recommendations.
5. **Response drafter** — generates replies for `needs_response`, tuned per
   property voice.
6. **Trend / anomaly watcher** — flags emerging issues (week-over-week
   complaint spike, rating drift).
7. **Manager Q&A agent** — RAG over reviews + structured stats. "Which
   properties under-perform on value in winter and why?" with citations.

Quality-raising agents most teams skip:

- **Devil's-advocate / critic** — re-reads each insight, argues against it
  (alternative explanations, sample size, confounders). Forces the synthesizer
  to strengthen or downgrade.
- **Cost-benefit agent** — estimates fix-cost ($X cleaner retraining, $Y AC
  install) and revenue impact (rating uplift → search rank → ADR →
  occupancy). Every recommendation gets a back-of-envelope ROI.
- **Sequencing agent** — orders all recommendations portfolio-wide by ROI,
  dependencies, and labor capacity. What goes into the PM's Monday standup.
- **Memory keeper** — per-property institutional memory of what we tried,
  what worked, what didn't. Prevents re-recommending failed fixes.

Orchestration patterns:

- **Cost-tiered routing** — Haiku for per-review extraction (cheap, high
  volume), Sonnet for theme synthesis, Opus for manager Q&A.
- **Critic-judge loops** on ABSA extraction with hand-labeled gold sets.
- **Tool use** — ticketing (Linear/Jira), email/WhatsApp drafts, calendar
  holds, vendor RFQs.

---

## 11. Memory architecture

Three layers (most teams build one and wonder why agents feel amnesiac):

- **Episodic** — per-review extraction results, immutable, append-only event
  log.
- **Semantic** — per-property aspect history, rating trajectory, prior
  recommendations and outcomes. The "what's true about this property right
  now" knowledge graph.
- **Procedural** — institutional lessons across portfolio. "AC complaints in
  summer at Bondi properties are usually filter clogging, not refrigerant."

**Bitemporal storage** matters: `aspect-as-of-extraction-date` vs
`aspect-as-of-review-date`. When the extractor is upgraded, don't silently
mutate the past — keep both versions.

---

## 12. The "trust deliverable" — what the PM actually sees

Every recommendation ships with:

1. **The action** (verb-led, one sentence).
2. **The evidence** — verbatim quoted snippets from N reviews, anonymized IDs.
3. **The expected impact** (rating lift, occupancy lift, dollars) with
   confidence band.
4. **The cost** (time, money, vendor).
5. **The critic's counter-argument** — the alternative explanation that was
   considered.
6. **One-click Accept / Reject / Defer** with a free-text reason field.

That feedback loop is the training data for the ranker. Without it the system
is read-only and dies.

---

## 13. Forward-direction (generative) use cases

- **Listing copy rewriter** with per-platform variants and predicted CTR.
- **Photo brief generator** — "Property X needs new photos showing: workspace
  nook, parking spot location, ocean-view angle." Hand to photographer.
- **House-manual generator** from clustered confusion points — auto-updated.
- **Welcome-message personalizer** — guest name + that property's top-3
  historical confusion points → pre-arrival WhatsApp.
- **Local-guide generator** from positively-mentioned cafes, walks,
  attractions across that property's reviews.
- **Pre-arrival expectation-setter** — "Other guests have noted [parking is
  tight / morning sun is strong]; here's how to handle." Reduces complaints
  before they happen.

---

## 14. Reverse-direction (guest-facing) use cases

The same engine can power:

- A natural-language property matchmaker on the booking surface.
- Honest review summaries — better than the platform default, citation-backed.
- Search-time amenity boosting based on actual usage signal, not just listed
  amenities.

---

## 15. Risk, ethics, compliance

- **PII hygiene** — `guestEmail` must be hashed before any LLM call.
  `guestName` likewise unless required for response drafting (then first name
  only).
- **Bias auditing** — is aspect-extraction quality consistent across guest
  demographics? Periodic eval on stratified samples.
- **Decision fairness** — never use guest names/origin to differentiate
  service. Audit any guest-facing personalization for this.
- **Hallucination guardrails** — every aspect claim must cite a quote from a
  specific `reviewId`. No quote → no claim.
- **Right-to-deletion** — guest deletion request propagates through derived
  stores (vector DB, aggregated stats can stay, individual extractions
  purged).
- **Defamation risk** — auto-drafted public responses go through a guardrail
  layer.

---

## 16. Evaluation framework

Without this nothing improves.

- **Aspect-extraction F1** on a 100-review hand-labeled gold set;
  inter-annotator agreement target ≥0.7.
- **LLM-as-judge calibration** against humans on a 50-review subset.
- **Recommendation acceptance rate** in the manager UI.
- **Response edit distance** — how much does the manager rewrite the drafted
  reply? Tracks drafter quality.
- **Outcome lift** — for accepted recommendations, did the rating actually
  move? (12-week lag.)
- **Cost per review processed** and **end-to-end latency** for response-draft
  SLA.
- **Drift detection** — monthly behavioral diff when prompts/models change.

---

## 17. Cost engineering

At 419 reviews this is irrelevant. At 419,000 it dominates. Design for it now:

- Extract once per review, cache forever, version-tag the extractor.
- Embed once, query many.
- Batch APIs for nightly bulk.
- Real-time path only for `safety` / `severe` flagged reviews and
  `needs_response` SLA.
- Token-budget per review with structured output to bound cost.

---

## 18. Australia-specific texture

- **Inverted seasonality** (summer Dec–Feb) — time-series analysis must be
  hemisphere-aware.
- City-specific peaks: Sydney Christmas/NYE, Adelaide Festival in March,
  ski-adjacent winter properties.
- International vs domestic guest mix changes value perception (currency,
  expectations).
- AU-specific amenities matter differently — BBQ, outdoor, heating in southern
  winter, screened windows in summer.
- Council noise complaints / party policies are a real legal vector — worth
  its own aspect bucket.

---

## 19. Data enrichment opportunities (to request from client)

- **Reservation-level data** (booking date, cancellations, no-shows, repeat
  bookings) — unlocks selection-bias correction and churn modeling.
- **Cleaning and maintenance logs** — closes the loop: did the cleaning ticket
  get done before the next review?
- **In-stay chat transcripts** — richer than post-stay reviews; catches issues
  *before* they become reviews.
- **Listing photos + guest-uploaded photos** — multimodal listing-accuracy
  auditing.
- **Pricing history** — actual paid rate, not just static average; enables
  true price-elasticity.
- **Channel-level performance** (impressions, CTR, search rank) — closes the
  funnel.
- **Weather and local-event data** — explains rating dips that aren't the
  property's fault.

---

## 20. The five most differentiated bets

If forced to pick the highest-leverage angles that most teams *won't* build:

1. **Amenity↔aspect causal graph** — one structure, four decision types.
2. **Rating-residual + rating-text discrepancy** — finds hidden dissatisfaction
   that averages hide.
3. **LLM-extracted aspects fed back into the ML layer** — the closed loop is
   the moat.
4. **Counterfactual ROI per recommendation** with critic-agent stress-testing —
   turns advice into trustable decisions.
5. **Bitemporal append-only aspect store with reasoning traces** — makes the
   system auditable, improvable, and defensible.

---

## 21. Open questions before any building

- **Who is the user of the output?** Single-property hosts, mid-size PM
  (10–100 props), or enterprise (1000+)? Agent design forks meaningfully here.
  Enterprise wants the portfolio sequencer + Q&A; small hosts want
  auto-drafted replies and a weekly digest.
- **What success metric does the client care about?** Rating uplift,
  response-SLA, time saved per property per week, churn reduction, or
  acquisition decision support? Different metrics → different agents get
  prioritized.
- **What data can be added beyond the current four files?** Section 19 is the
  shopping list.
- **Build vs integrate?** PMS (Guesty, Hostaway, OwnerRez, Mews), channel
  managers, and ticketing platforms (Breezeway) define the integration
  surface. Greenfield or plug-in?

---

*Next step (when ready to leave design space): pick a thin vertical slice —
likely the aspect extractor + amenity↔aspect graph + a single manager-facing
recommendation surface — and prototype on the 419 reviews to pressure-test the
ontology and the trust-deliverable format.*

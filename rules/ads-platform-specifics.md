# Ads Platform Specifics

This document defines platform-specific safety logic for ad mutations in CND-OS.

The goal is not API correctness alone. The goal is to prevent expensive operational mistakes before they happen.

---

## Meta Ads

### Purpose

Meta needs a stricter mutation layer than AdLoop's Google pattern because Meta failures are usually not syntax failures. They are delivery-state failures:

- too many ad sets fragment learning
- weak format choice burns spend before the algorithm gets signal
- profitable entities get damaged by "cleanup" edits
- multiple small edits can reset or destabilize delivery

CND-OS must treat Meta writes as controlled change management, not CRUD.

### Past Failure Modes This Section Must Prevent

1. Budget supported `2` ad sets, but `6` were launched.
2. Carousel was used as the default cold premium format and produced `0` purchases.
3. A `4.27x` ROAS campaign was paused to restructure, wasting about `500 EUR`.

Everything below exists to stop those exact mistakes from recurring.

---

## Meta Safety Model

### Core Principles

1. Preserve profitable delivery before improving account structure.
2. Budget determines structure, not operator preference.
3. Creative format must match audience temperature and product class.
4. Active winners are protected assets, not editable scaffolding.
5. Clone-and-test is safer than edit-in-place on live winners.
6. One mutation should solve one problem.
7. No Meta write is allowed without pre-write validation.

### Risk Classes

`Low risk`
- add a new ad inside a stable ad set
- rename paused entities
- small budget increase on a stable winner

`Medium risk`
- create campaign
- create ad set
- launch a new creative test
- moderate budget increase

`High risk`
- pause profitable campaign or ad set
- change optimization event or promoted object path
- restructure live acquisition
- bulk pause or bulk archive
- combine structure, creative, and budget changes together

### Default Posture

- Safe default if uncertain: do less.
- Prefer fewer ad sets.
- Prefer adding ads over adding ad sets.
- Prefer adding ad sets over creating campaigns.
- Prefer cloning over editing profitable entities.
- Prefer preserving the original winner while testing a replacement.

---

## Meta Safety Layer

### 1. Hard Guards

These block the mutation. They are not warnings.

#### Budget Cap

- Reject any daily budget above the client or account cap.
- Reject any budget increase above configured maximum without double confirmation.
- Reject any scaling plan that materially increases spend before validating profitability and learning state.

#### Learning Phase Math

Meta generally needs about `50 optimization events per ad set per 7 days` for stable learning.

Use:

```text
required_daily_budget_per_ad_set = (target_cpa_or_cost_cap * 50) / 7
max_supported_ad_sets = floor(total_daily_budget / required_daily_budget_per_ad_set)
```

Target CPA source priority:

1. last `30d` median purchase CPA for the same account and same funnel stage
2. last `30d` blended account purchase CPA
3. launch CPA derived from economics, margin, and AOV

Never use optimistic guessed CPA just to justify more structure.

Hard rules:

- Reject if `planned_ad_sets > max_supported_ad_sets`.
- Reject if any ad set would receive less than the required daily budget.
- Reject cold purchase launches with more than `4` ad sets unless budget math supports it and segmentation has a business reason.
- Reject age, gender, placement, or message-split ad sets when the budget does not support separate learning loops.

Operational reading:

- If the budget supports `2`, do not build `6`.
- Consolidation beats elegant starvation.

#### Budget-Per-Ad-Set Validation

Before creating or expanding ad sets, validate:

- total campaign budget
- expected daily budget per ad set
- event volume needed for optimization
- whether the split changes strategy or only creates reporting neatness

Reject if:

- the split is cosmetic
- one ad set becomes underfunded
- the only reason for the split is audience neatness, age bands, or placement preference

#### Winner Protection

Profitable active entities are protected by default.

Block pause, major edit, or restructure if an entity meets both:

- `ROAS >= 3.0` over the last `7-14` days
- at least `3` purchases or otherwise meaningful revenue contribution in that window

Explicit hard block:

- any restructure that pauses or materially disrupts an entity around `4.27x` ROAS without a break-fix reason

Allowed exceptions:

- policy rejection
- broken tracking
- destination failure
- wrong promoted object or wrong optimization event
- explicit user approval after the preview states downside in money and lost-learning terms

#### Blocked Operations

These are blocked globally or require explicit override config:

- pausing profitable active campaigns for restructure only
- bulk pausing active campaigns without performance review
- deleting campaigns, ad sets, ads, creatives, or audiences instead of pausing
- creating more ad sets than learning math supports
- launching cold premium prospecting with carousel as the primary format without proof
- moving a winner into a new structure and pausing the original in the same mutation
- changing campaign objective or optimization path on a live profitable campaign without clone-first handling
- combining restructure and scale in one write

#### Double Confirmation Operations

Require double confirmation for:

- any delete or irreversible remove action
- any pause on a profitable active entity
- any budget increase above `50%`
- any campaign-level optimization event or bid-strategy change
- any mutation touching campaign structure and active creatives in one plan
- any mutation with expected downside above a configured money threshold

### 2. Creative Format Hierarchy

Format selection is not aesthetic. It is a safety decision.

#### Cold Traffic

##### Cold Premium Products

Default order:

1. proof-led video or UGC
2. strong single-image static
3. founder, authority, or transformation static/video
4. collection if catalog behavior already works
5. carousel last

Rules:

- Do not default to carousel for cold premium.
- Carousel is allowed only with prior proof, a strong educational comparison need, or a very clear browsing advantage.
- If carousel has meaningful spend and `0` purchases, treat it as disconfirmed, not "still testing forever."

Reason:

Cold premium traffic needs fast belief transfer. Carousel often delays the proof and weakens the hook.

##### Cold Commodity or Catalog-Led Ecommerce

Default order:

1. short video or motion-first asset
2. strong single image
3. collection or DPA
4. carousel only if multi-SKU browsing materially helps the sale

#### Warm Traffic

Default order:

1. best cold winner
2. proof-heavy testimonial or comparison creative
3. collection or DPA
4. carousel if the user already knows the brand and needs browsing help

#### Hot / Remarketing

Default order:

1. DPA or catalog format
2. proven winner from cold or warm
3. urgency, offer, bundle, or stock-based static/video
4. carousel if it helps product review, variant review, or bundle logic

#### Format Rules

- Single-message formats come first for cold premium.
- Browse-heavy formats come later in the funnel unless proven otherwise.
- If budget is limited, test fewer formats with conviction.
- Do not change audience, format, and offer at the same time unless the brief explicitly prioritizes exploration over control.

### 3. Mutation Decomposition Rules

One mutation should change one lever whenever possible.

Reject or split plans that combine:

- restructure + scale
- new audience + new format + new offer
- pause winner + launch replacement same day
- creative refresh + budget increase + audience split in one step

Preferred decomposition:

1. preserve winner
2. launch clone or side test
3. observe results
4. scale or migrate later

---

## Meta Decision Trees

### 1. Campaign Creation

Use this before any new Meta campaign is drafted.

#### Step 1: Should a new campaign exist at all?

Create a new campaign only if one of these is true:

- a missing funnel stage must be added
- a new optimization goal is required
- a new market or language requires isolation
- budget ownership must be separated
- the existing campaign cannot support the job without harming winners

Do not create a campaign if:

- the real need is only new creatives
- the current profitable campaign can absorb the test
- the motivation is naming cleanliness or account symmetry

Preferred alternative:

- add ads to an existing ad set
- clone an ad set if true separation is needed

#### Step 2: Can the budget support the intended shape?

- Calculate `max_supported_ad_sets`.
- If the budget supports `1-2`, build `1-2`.
- Collapse audience ideas unless segmentation changes strategy materially.

#### Step 3: Choose structure by funnel temperature

`Cold`
- broad or minimal strategic segmentation
- no decorative splits

`Warm`
- website visitors, engagement, video viewers only if audience size is real

`Hot`
- remarketing with real custom audiences
- no broad automation by default

#### Step 4: Validate setup before draft

Must be correct before creation:

- pixel
- promoted object
- optimization event
- attribution setting
- special ad category
- automation settings by funnel stage

#### Step 5: Choose creative entry point

- cold premium: proof-led video or static
- cold commodity: motion or static first, catalog second
- warm: reuse winners plus proof
- hot: DPA plus best converting proof/offer creative

#### Step 6: Launch policy

- start controlled
- do not open too many ad sets at once
- do not restructure and scale in the same step

Decision:

- if objective is valid and budget supports structure, create
- if objective is valid but structure is too large, simplify
- if the job can live inside an existing winner, do not create

### 2. Scaling

Scaling questions, in order:

1. Is the entity profitable enough to deserve more spend?
2. Is performance stable across multiple days, not one spike?
3. Is the campaign stable or still learning?
4. Is spend capped by budget, or capped by weak performance?

Preferred scaling order:

1. increase budget gradually on existing winners
2. add budget to the winning ad set before opening more ad sets
3. expand creative volume after structure is funded
4. clone into adjacent tests only after the original remains active

Rules:

- do not "scale" by multiplying ad sets
- do not segment a winner apart just to look cleaner
- do not combine large budget increases with format or audience changes

Guard logic:

- small increase on a stable winner: warning or allow
- large increase on a stable winner: double confirmation
- scaling during learning: warn
- scaling plus restructure: reject or split

### 3. Creative Strategy

Decision path:

1. What is the audience temperature?
2. Is the product premium, commodity, hero SKU, or catalog-led?
3. What proof already exists in this account?
4. Which format has converted here already?
5. Is the budget large enough to test more than one major format class?

Rules:

- in-account proof outranks generic best practice
- if a static or video winner exists, iterate adjacent angles first
- cold premium does not lead with browse-heavy formats
- remarketing should prioritize memory, offer, urgency, and product relevance
- creative diversity is not a goal by itself; the goal is faster proof and better conversion

Decision:

- if proven format exists, iterate inside that class first
- if no proof exists, start with the safest format hierarchy for the stage
- if the proposed format conflicts with account evidence, block or escalate

### 4. Troubleshooting Workflows

#### Problem: No delivery or weak spend

Check:

- budget too low for ad set count
- audience too narrow
- bid strategy too restrictive
- overlap across too many ad sets
- excessive fragmentation keeping ad sets in learning

Fix order:

1. consolidate ad sets
2. remove unnecessary splits
3. raise budget only if economics support it
4. keep the strongest creative live during cleanup

#### Problem: Spend but no purchases

Check:

- tracking integrity
- optimization event correctness
- landing page conversion health
- creative-message mismatch
- wrong audience-product fit
- wrong format for stage

Fix order:

1. verify tracking and destination integrity
2. verify offer and landing page
3. replace weak format or message
4. simplify structure before adding more segmentation

#### Problem: Cold premium campaign not converting

Check:

- carousel leading cold traffic
- weak proof
- browse-first rather than belief-first creative
- too many ad sets for the budget

Fix order:

1. stop leading with carousel
2. replace with proof-led video or strong static
3. simplify structure
4. test angles before opening more ad sets

#### Problem: Profitable campaign "needs a better structure"

Check:

- real business need versus account neatness
- whether a clone can test the new structure safely
- whether the original winner can remain active

Required action:

1. keep the profitable original live
2. clone into test structure
3. compare before moving budget

Blocked action:

- pause the profitable original first

---

## Pre-Write Validation

Before any Meta mutation, CND-OS must validate the relevant items below. If an item cannot be checked, the system must either block the write or downgrade it to a documented warning based on severity.

### A. Account and Tracking Integrity

Must validate:

- pixel is connected and receiving events
- promoted object matches the intended optimization
- optimization event is correct
- conversion data is recent enough to trust
- destination URL or product destination is live
- attribution setting is explicit
- CAPI or external attribution status is known when relevant

Block if:

- purchase tracking is broken or missing
- destination is invalid
- promoted object is wrong
- optimization event is wrong for the campaign intent

### B. Current Performance Context

Must validate:

- last `7`, `14`, and `30` day spend
- purchases, CPA, ROAS, CTR, CPC, CPM
- whether the target entity is profitable
- whether the target entity is learning, limited-learning, or stable
- whether another active winner already solves the same job

Escalate or block if:

- the mutation touches a profitable winner
- the change mixes troubleshooting and restructure
- the campaign is unstable and the proposed change would add more variables

### C. Structure Sufficiency

Must validate:

- total daily budget
- target CPA or cost cap source
- required daily budget per ad set
- max supported ad set count
- planned ad set count
- audience size and overlap rationale

Block if:

- structure exceeds budget support
- audience splits are cosmetic
- multiple ad sets exist only for age, gender, placement, or angle reporting

### D. Creative and Format Fit

Must validate:

- audience temperature
- product class
- existing account proof by format
- creative count and quality
- whether the proposed format is appropriate for the stage

Block if:

- cold premium launch leads with carousel without evidence
- proposed format conflicts with stronger in-account proof
- the test depends on multiple weak assumptions at once

### E. Mutation Severity and Blast Radius

Must classify:

- risk level
- layers touched: campaign, ad set, ad, budget, audience, creative
- whether the write is reversible
- expected downside if wrong

Rules:

- high-risk writes require explicit downside language in preview
- destructive writes require double confirmation
- multi-layer high-risk writes should be decomposed into separate plans

### F. Alternatives Check

Before any approval, ask:

- can this be solved by adding a test instead of editing the winner?
- can this be solved by consolidating instead of expanding?
- can this be solved by changing creative before changing structure?
- can this be solved by budget adjustment before ad set multiplication?

If yes, prefer the safer path.

---

## Mutation Policy by Operation

### Create Campaign

Must check:

- a real funnel or objective gap exists
- promoted object and optimization event are correct
- budget supports ad set count
- audience segmentation is justified
- creative entry format matches temperature and product class

Default posture:

- suspicious until budget-to-structure math is clean

### Create Ad Set

Must check:

- required daily budget support
- audience rationale
- overlap risk
- whether the real need is another ad, not another ad set

Default posture:

- restrictive by default because this is where Meta accounts get fragmented

### Create Ad

Must check:

- destination integrity
- format-stage fit
- meaningful differentiation from existing ads
- existing winner context

Default posture:

- usually the safest write, unless it reinforces a known losing format choice

### Update Budget

Must check:

- profitability
- learning state
- whether performance is constrained by budget or by weak conversion
- increase size versus recent spend stability

Default posture:

- gradual on winners, conservative on unstable campaigns

### Pause Entity

Must check:

- recent profitability
- reason for pause
- whether pause is tactical or only structural
- whether a safer alternative exists

Block if:

- the entity is a current winner and the reason is restructure only

### Delete or Remove

Default posture:

- avoid; pause instead

Allow only with:

- explicit user intent
- double confirmation
- clear irreversible warning

---

## Improvements Over AdLoop's Google Pattern

AdLoop's Google pattern is directionally correct:

- blocked operations
- budget caps
- preview-before-apply
- destructive-action confirmation
- mandatory pre-write checks

Meta needs more than that because static validation is not enough.

### 1. Delivery-State Awareness

AdLoop's guard layer is mostly static. Meta needs dynamic validation against:

- learning status
- recent event volume
- stable versus unstable delivery
- active winner profitability

Reason:

- the same write can be harmless on a paused loser and reckless on a live winner

### 2. Budget-to-Structure Math as a First-Class Guard

Google ad group expansion is not equivalent to Meta ad set expansion.

CND-OS Meta must explicitly calculate:

- required daily budget per ad set
- max supported ad set count
- whether the structure will starve learning

Reason:

- this prevents the exact "6 ad sets when budget supported 2" failure

### 3. Creative Format Gating

AdLoop validates assets and blocked operations. Meta also needs:

- format by audience temperature
- format by product class
- format by in-account conversion proof

Reason:

- this prevents repeating "carousel on cold premium with 0 purchases"

### 4. Winner Preservation Logic

Google structures often tolerate edits better than Meta acquisition.

CND-OS Meta must block:

- pause-to-restructure on profitable winners
- cleanup-driven edits on active profitable entities
- migration of winners into new structures without keeping originals live

Reason:

- this prevents repeating "paused 4.27x ROAS campaign to restructure"

### 5. Change Decomposition

AdLoop's model should be extended so Meta plans are split when too many risk variables move together.

Block or decompose:

- restructure + scale
- audience change + new creative class + offer change
- pause winner + replacement launch same day

Reason:

- Meta diagnosis becomes unreliable when too many variables move at once

### 6. Clone-First Policy

For profitable Meta entities, the safety model should encode:

1. preserve original
2. clone into test
3. compare performance
4. migrate budget only after proof

This should be a guardrail, not optional operator advice.

### 7. Opportunity-Cost Previewing

AdLoop focuses on plan preview. CND-OS Meta should preview downside in business terms too:

- expected spend at risk
- winner revenue at risk
- learning reset risk
- number of variables changed

Reason:

- the operator must see cost of disruption, not only the mutation payload

---

## Non-Negotiable Meta Safety Rules

1. Never create more ad sets than the budget can support through learning math.
2. Never pause a profitable active campaign just to restructure it.
3. Never default to carousel for cold premium traffic without proof.
4. Never treat account neatness as a valid reason to disturb a winner.
5. Never combine restructure, creative reinvention, and scaling in one uncontrolled write.
6. Every Meta mutation must pass pre-write validation before draft or apply.

---

## Implementation Notes for CND-OS

The Meta safety layer should mirror AdLoop's preview-first write model, but with richer Meta-specific guard functions:

```text
check_budget_cap(...)
check_learning_phase_support(...)
check_budget_per_ad_set(...)
check_blocked_operation(...)
check_winner_protection(...)
check_creative_format_fit(...)
check_mutation_blast_radius(...)
requires_double_confirmation(...)
```

The key difference from AdLoop's Google pattern is this:

- Google safety can often validate the proposed change.
- Meta safety must validate the proposed change against the current delivery context before the preview is even considered valid.

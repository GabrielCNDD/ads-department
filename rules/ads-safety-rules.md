# CND Ads Safety Rules

_Mandatory for all ads work. Every rule exists because breaking it cost real money._
_Read this BEFORE touching any ad account. No exceptions._

---

## How This System Works

```
USER REQUEST
     │
     ▼
┌─────────────────────────────┐
│  GATE 1: SAFETY RULES       │  ← You are here. Hard stops.
│  "Can I even do this?"      │  If blocked → STOP. Explain why.
│  Sections 1-7 below         │
└─────────────┬───────────────┘
              │ PASS
              ▼
┌─────────────────────────────┐
│  GATE 2: DATA PULL          │  Pull 30-90 day performance.
│  "What's actually happening?"│  Identify protected campaigns.
│  Cross-ref: Platform+GA4+   │  Calculate cost of change.
│  Shopify                    │
└─────────────┬───────────────┘
              │ DATA READY
              ▼
┌─────────────────────────────┐
│  GATE 3: DECISION TREE      │  Follow ads-decision-trees.md
│  "What's the right action?" │  for the specific scenario.
│  Pre-write validation       │  Calculate everything.
└─────────────┬───────────────┘
              │ DECISION MADE
              ▼
┌─────────────────────────────┐
│  GATE 4: DRAFT & PREVIEW    │  Create the change plan.
│  "Here's what I want to do" │  Show: what, why, expected
│  Present to Gabriel         │  impact, risks, cost.
└─────────────┬───────────────┘
              │ GABRIEL APPROVES
              ▼
┌─────────────────────────────┐
│  GATE 5: DRY RUN            │  Execute with dry_run=true.
│  "Does the API accept this?"│  Verify no errors.
└─────────────┬───────────────┘
              │ DRY RUN OK
              ▼
┌─────────────────────────────┐
│  GATE 6: EXECUTE & LOG      │  Apply the change.
│  "Done. Logged. Verified."  │  Log to ads/CHANGELOG.md.
│                             │  Verify delivery after 24h.
└─────────────────────────────┘
```

**Why 6 gates?** Because every past mistake skipped at least one:
- Gossip 0.01 RON bids → skipped Gate 2 (no data pull) and Gate 3 (no calculation)
- Gossip 500 EUR rebuild → skipped Gate 1 (safety rule: never rebuild working) and Gate 2 (didn't check what was converting)
- MaxConv on 0-history → skipped Gate 3 (decision tree says Manual CPC first for small budgets)

---

## Section 1: Destructive Operations

**Rule:** NEVER permanently delete campaigns, ad sets, ads, or keywords on ANY platform.

**Why:** Deleted entities lose all historical data — performance metrics, audience learnings, conversion data. This data is irreplaceable and needed for future decisions.

**What to do instead:**
- Pause the entity
- Rename with `ARCHIVE_` or `OLD_` prefix
- Move to a "paused" campaign if platform supports it

**Double confirmation required for:**
- Any delete/remove operation (if Gabriel explicitly requests it)
- Budget increases > 50% in one step
- Pausing any campaign with ROAS > 2x
- Enabling a campaign that's been paused > 30 days
- Changing bidding strategy on a campaign in learning phase

**Audit log format** (append to project's `ads/CHANGELOG.md`):
```
## YYYY-MM-DD HH:MM — [PLATFORM] [ACTION]
- **What:** [specific change]
- **Why:** [business reason]
- **Impact:** [expected result]
- **Approved by:** Gabriel / pre-authorized
- **Reversible:** Yes/No
- **Dry run result:** [pass/fail/skipped]
```

---

## Section 2: Working Account Protection

**Rule:** NEVER pause, restructure, or replace campaigns that are currently profitable.

**Why:** On 2026-04-01, Gossip had 4.27x ROAS across 3 campaigns (362 purchases/month). We rebuilt from scratch — 6 new Kerastase ad sets, paused the Google Shopping workhorse. Wasted ~500 EUR and lost a week of revenue. Gabriel caught it on 2026-04-06.

**The protocol:**

### Step 1: Baseline Before Anything
Before proposing ANY account changes:
1. Pull last 30 days performance for ALL active campaigns
2. Pull Shopify revenue by source for the same period
3. Cross-reference: platform-reported vs Shopify-attributed

### Step 2: Mark Protected Campaigns
A campaign is **PROTECTED** if:
- ROAS > 2x (platform-reported) in the last 30 days
- Spending consistently (no days with 0 spend in last 14 days)
- Has 10+ conversions in the last 30 days

Protected campaigns **CANNOT** be:
- Paused
- Budget-reduced
- Bidding strategy changed
- Audience narrowed
- Creative paused (if only 1-2 ads)

...without Gabriel's **explicit, specific approval** stating he understands the revenue impact.

### Step 3: Calculate Change Cost
Before proposing any change to a protected campaign:
```
Daily revenue at risk = Campaign's 30-day revenue ÷ 30
Weekly revenue at risk = Daily × 7
```
Present this number when asking for approval.

### Step 4: Test Alongside, Never Replace
- New campaigns run NEXT TO working ones
- New campaigns must PROVE themselves (7+ days, comparable metrics)
- Only after proof: gradually shift budget (20% increments)
- Old campaign stays active until new one matches or beats its ROAS

---

## Section 3: Approval Gate

**Rule:** ALL ad account changes must be presented to Gabriel before execution.

**Why:** The agent makes mistakes. Corrections cost real money. Gabriel is the final check.

**What "present" means:**
```
## Proposed Change: [short title]

**Account:** [client name] — [platform]
**Action:** [what specifically changes]
**Why:** [business reason / data justification]
**Expected impact:** [what should improve, by how much]
**Risk:** [what could go wrong]
**Revenue at risk:** [if touching a working campaign: daily revenue number]
**Cost of change:** [any direct costs — new budget, etc.]
**Reversible:** [yes/no — how to undo if it fails]
**Dry run:** [pass/fail]
```

**Pre-authorized actions** (can execute without asking each time):
- Adding negative keywords to campaigns with > 10% wasted spend
- Pausing ads with CTR < 0.5% after 1000+ impressions and 0 conversions
- Adjusting bids within ±15% of current value based on impression share data

Everything else: ask first.

---

## Section 4: Bidding Safety

**Rule:** Follow the bidding ladder. Never skip steps. Never guess bids.

**Why:** MaxConv on 0-history campaign = Gossip lost 9+ hours of Brand Search coverage (0 impressions). 0.01 RON bids = Shopping campaign invisible. Both cost real money, both were avoidable.

### The Bidding Ladder (Google Ads)

| Step | Strategy | Minimum Data | When to Move Up |
|------|----------|-------------|-----------------|
| 1 | Manual CPC | 0 conversions | Starting point for ALL new campaigns |
| 2 | Enhanced CPC | 10+ conversions | After 2 weeks, if CPCs are stable |
| 3 | Maximize Conversions | 15+ conv/month | After 30+ conversions accumulated |
| 4 | Target CPA | 30+ conv/month | Stable CPA for 2+ weeks |
| 5 | Target ROAS | 50+ conv/month | Stable ROAS, enough volume |

**Exception:** Campaigns with > 1,000 RON/day budget AND high search volume category CAN start at Maximize Conversions. Document the reasoning.

### Bid Calculation (mandatory)

Every bid MUST be calculated from:
```
Max CPC = Target CPA × Expected CVR
```
Then cross-checked against:
1. Product price economics (reference: ads-bidding-logic.md)
2. Industry benchmarks for Romania
3. Start at 60-70% of calculated max

**Blocked:**
- Setting bids at 0.01 RON (or any obviously non-competitive amount)
- Setting round-number bids without calculation (e.g., "let's try 2 RON")
- Setting bids higher than the product's margin can justify

### Meta Ads Bidding

- Default: `LOWEST_COST_WITHOUT_CAP` for new ad sets
- Cost cap: only after 50+ conversions with stable CPA
- ROAS cap: only after 100+ conversions with stable ROAS
- Budget per ad set: must support learning phase exit (see Section 5)

---

## Section 5: Budget Safety

**Rule:** Calculate budget requirements BEFORE proposing campaign structure.

**Why:** On Gossip, we built 6 Kerastase ad sets when the budget only supported 2 for learning phase exit. All 6 got stuck in learning, none converted, budget wasted.

### The Math (do this FIRST)

**Meta Ads learning phase:**
```
Minimum budget per ad set = 50 conversions/week × Cost Per Purchase (CPP)
Max viable ad sets = Total weekly budget ÷ Minimum per ad set
```
Example: 2,000 RON/week budget, 40 RON CPP
- Min per ad set = 50 × 40 = 2,000 RON/week
- Max viable ad sets = 2,000 ÷ 2,000 = **1 ad set**
- Building 6 ad sets with this budget = guaranteed failure

**Google Ads budget:**
```
Minimum daily budget = Target CPA × 5 (minimum)
Recommended daily budget = Target CPA × 10 (comfortable)
```
Example: Target CPA 100 RON
- Minimum: 500 RON/day
- Recommended: 1,000 RON/day

### Budget Caps

Per-client budget caps stored in `projects/{client}/ads/config.json`:
```json
{
  "client": "gossip",
  "platforms": {
    "google": {
      "max_daily_budget": 200,
      "currency": "RON",
      "account_id": "xxx-xxx-xxxx"
    },
    "meta": {
      "max_daily_budget": 300,
      "currency": "RON",
      "account_id": "act_xxxxxxxxx"
    }
  },
  "targets": {
    "target_roas": 4.0,
    "target_cpa": 40,
    "aov": 375
  },
  "protected_campaigns": []
}
```

**Blocked:**
- Proposing budget above client's max_daily_budget
- Scaling budget > 40% in one step (20% increments, every 3-4 days)
- Creating more ad sets than the budget can support through learning phase
- Creating campaigns without checking budget sufficiency first

---

## Section 6: Platform-Specific Blocks

### Google Ads — Blocked Operations

| Operation | Why Blocked | Alternative |
|-----------|-------------|-------------|
| Broad Match + Manual CPC | #1 cause of wasted spend. Broad without Smart Bidding matches irrelevant queries | Use Phrase or Exact match. Or switch to Smart Bidding first |
| Campaign without geo targeting | Untargeted = global serving = wasted budget | Always set geo_target_ids. Romania = 2642 |
| Campaign without language targeting | Shows ads to wrong language speakers | Always set language_ids. Romanian = 1032 |
| MaxConv on 0-history + < 1000 RON/day | Algorithm has no signal, sits idle | Start Manual CPC, follow the ladder |
| RSA with unverified URLs | 404 landing pages waste budget + destroy QS | Verify URL responds 200 before creating ad |
| Shopping without feed verification | Disapproved products = invisible ads | Check Merchant Center status first |

### Meta Ads — Blocked Operations

| Operation | Why Blocked | Alternative |
|-----------|-------------|-------------|
| More ad sets than budget supports | Learning phase failure guaranteed | Calculate first (Section 5 math) |
| Carousel for cold premium audiences | Proven 0 purchases on Gossip cold traffic | Use Video or DPA catalog |
| Advantage+ Audience on remarketing | Broadens beyond your warm audience | Set advantage_audience: 0 |
| product_set_id in promoted_object for non-DPA | API error / wrong optimization | Only use for DPA catalog ads |
| multi_share_optimized in template_data | Causes creative serving issues | Never use |
| Campaign without promoted_object (pixel+PURCHASE) | Can't change after creation | Set at creation, always |
| Budget sharing enabled | Unpredictable ad set spending | is_adset_budget_sharing_enabled: false |

---

## Section 7: Data Integrity

**Rule:** Every decision must be backed by cross-referenced data. No theory, no "proper structure," no gut feeling.

**Why:** On 2026-04-06, Gabriel caught that we wasted 500 EUR building campaigns based on theory ("proper full-funnel structure") instead of analyzing what was already working. The data showed 2.78x ROAS on Google Shopping and 4.99x on Meta. We ignored it.

### The Three Truth Sources

| Source | What It Tells You | Trust Level |
|--------|-------------------|-------------|
| Platform (Google/Meta) | Clicks, impressions, platform-attributed conversions | Medium — each platform over-claims |
| GA4 | Sessions, on-site behavior, GA4-attributed conversions | Medium — GDPR consent reduces counts |
| Shopify | Actual orders, actual revenue, actual attribution | **High — this is the ground truth** |

### Cross-Reference Protocol
Before any account decision:
1. Pull platform data (last 30 days)
2. Pull GA4 data (same period, by source/medium)
3. Pull Shopify data (same period, by source)
4. Compare: if platform says 100 conversions but Shopify shows 60 orders from that source → real number is closer to 60

### GDPR Awareness (EU/Romania)
- Google Ads clicks > GA4 sessions is **normal** (2:1 to 5:1 ratio)
- Users who reject cookies are invisible to GA4
- Only flag tracking as broken when the gap can't be explained by consent
- Signs of real tracking issues: zero sessions for ALL sources, organic also anomalous

### Verify Before Presenting
- Never present audit findings without verifying them
- Platform diagnostic tools can give false positives
- Cross-reference every claim with actual data
- If an automated check says "tracking broken" but Shopify shows orders → tracking isn't broken

---

## Quick Reference: Is This Blocked?

| I want to... | Blocked? | Gate |
|--------------|----------|------|
| Delete a campaign | YES — pause + rename instead | Section 1 |
| Pause a profitable campaign | YES — unless Gabriel explicitly approves | Section 2 |
| Create a campaign | NO — but must pass Gates 2-6 | Section 4, 5 |
| Change bidding strategy | DEPENDS — check the ladder | Section 4 |
| Increase budget > 40% | YES — use 20% increments | Section 5 |
| Add Broad Match keywords | DEPENDS — only with Smart Bidding | Section 6 |
| Use MaxConv on new campaign | DEPENDS — check budget + history | Section 4 |
| Add negative keywords | PRE-AUTHORIZED if > 10% waste | Section 3 |
| Pause 0-conversion ad | PRE-AUTHORIZED if 1000+ imps | Section 3 |

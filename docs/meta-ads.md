# Meta Ads — Platform Logic

_Meta-specific rules, gotchas, and operational knowledge._
_Detailed safety layer and decision trees: see `rules/ads-platform-specifics.md`_

---

## Why Meta Needs Stricter Rules

Meta failures are usually not syntax failures. They are **delivery-state failures:**
- Too many ad sets fragment learning (each needs 50 conversions/week)
- Weak format choice burns spend before the algorithm gets signal
- Profitable entities get damaged by "cleanup" edits
- Multiple small edits can reset or destabilize delivery

Meta writes must be treated as **controlled change management**, not CRUD.

---

## Learning Phase Math

This is the single most important calculation for Meta Ads.

```
Required daily budget per ad set = (Target CPA × 50) ÷ 7
Max viable ad sets = Total daily budget ÷ Required per ad set
```

**Example:** Budget 2,000 RON/week, CPP 40 RON
- Required per ad set = (40 × 50) ÷ 7 = ~286 RON/day
- Max viable ad sets = (2000 ÷ 7) ÷ 286 = **1 ad set**

Building 6 ad sets with this budget = all 6 stuck in learning = 100% waste.

**Target CPA source (priority order):**
1. Last 30-day median purchase CPA for same account, same funnel stage
2. Last 30-day blended account purchase CPA
3. Calculated from economics: AOV × Margin × Target Ad Spend %

---

## Creative Format Hierarchy

### Cold Premium (e.g., Kerastase, K18)
1. Proof-led video or UGC
2. Strong single-image static
3. Authority / transformation creative
4. Collection (if catalog behavior proven)
5. **Carousel — LAST** (proven 0 purchases on cold premium at Gossip)

### Cold Commodity / Catalog-Led
1. Short video or motion
2. Strong single image
3. DPA / collection
4. Carousel (only if multi-SKU browsing helps the sale)

### Warm (website visitors, engaged users)
1. Best cold winner (reuse what works)
2. Proof-heavy testimonial or comparison
3. DPA / collection
4. Carousel (user already knows brand)

### Hot / Remarketing
1. DPA catalog (show them what they viewed)
2. Proven winner from cold/warm
3. Urgency, offer, bundle, stock-based
4. Carousel (variant review, bundle logic)

---

## Campaign Setup Checklist

**Set at creation (immutable after):**
- `promoted_object`: pixel_id + optimization event (PURCHASE)
- `objective`: OUTCOME_SALES (for ecommerce)
- `special_ad_categories`: ["NONE"] (unless regulated industry)

**Set on ad sets:**
- `optimization_goal`: OFFSITE_CONVERSIONS
- `billing_event`: IMPRESSIONS
- `bid_strategy`: LOWEST_COST_WITHOUT_CAP (for new)
- `targeting_automation.advantage_audience`: 0 (on remarketing)
- `is_adset_budget_sharing_enabled`: false (always)

**Never use:**
- `product_set_id` in promoted_object for non-DPA ads
- `multi_share_optimized` in template_data
- `instagram_actor_id` unless system user has IG permissions

---

## Scaling Protocol

1. **Verify profitability** — Shopify-verified, not Meta-reported
2. **Verify stability** — 7+ consecutive profitable days
3. **Verify learning status** — must be "Active" (not Learning/Limited)
4. **Increase 20% per step** — every 3-4 days
5. **Monitor after each increase** — if CPA rises > 30%, pause increase

**Budget split:** 70% on proven campaigns, 30% on testing

**Blocked:**
- Increase > 40% in one step
- Scaling + restructuring simultaneously
- Scaling based on Meta-reported ROAS without Shopify cross-reference

---

## Meta Attribution Reality

Meta uses:
- 1-day view-through attribution (default)
- 7-day click-through attribution (default)
- Data-driven attribution model

This means Meta claims credit for conversions from users who:
- Saw an ad but didn't click (view-through)
- Clicked 6 days ago and came back through Google (click-through)

**Reality check:** Meta typically over-reports by 30-80% vs Shopify.

**Rule:** Always use Shopify-verified numbers for decisions. Meta numbers for platform optimization only.

---

## Common Meta Mistakes

### Too Many Ad Sets
Each ad set needs ~50 conversions per week to exit learning. Most budgets support 1-3 ad sets, not 6-10.

### Carousel for Cold Premium
Cold premium audiences need fast belief transfer. Carousel delays the hook across multiple cards. Video and strong single-image outperform.

### Pausing Winners to Restructure
A profitable campaign that "looks messy" is still profitable. Pausing it to rebuild "properly" resets all learning and wastes the accumulated data.

### Trusting Meta-Reported ROAS
A 4x ROAS on Meta might be a 2.5x on Shopify. Decisions based on inflated numbers lead to overspending.

### Advantage+ Audience on Remarketing
Advantage+ broadens targeting beyond your custom audiences. On remarketing, you want to talk to people who already know you — not random lookalikes.

---

## API Gotchas

- `promoted_object` is **immutable** after campaign creation — set it right the first time
- App must be in **LIVE** mode (not Development) for production campaigns
- System user needs Page access + Instagram access + Catalog access
- Image ads: upload as multipart file, not URL
- `special_ad_categories` must be set even if ["NONE"]
- Catalog/DPA ads require product feed in Commerce Manager

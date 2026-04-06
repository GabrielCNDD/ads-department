# Google Ads — Platform Logic

_Google-specific rules, gotchas, and operational knowledge._

---

## Bidding Strategy Ladder

Google Ads bidding strategies are NOT interchangeable. Each requires a minimum amount of conversion data to function. Using the wrong strategy for the data level is the #1 cause of campaign failure.

```
Level 1: Manual CPC
├── Requirements: none (starting point)
├── How it works: you set the exact bid per click
├── Risk: bids too low = 0 impressions, bids too high = overspending
├── When to use: new campaigns, low budget, niche markets
└── Start bid: Target CPA × Expected CVR × 0.65

Level 2: Enhanced CPC (eCPC)
├── Requirements: 10+ conversions
├── How it works: Google adjusts your manual bid ±30% based on conversion likelihood
├── Risk: low — still bounded by your base bid
└── When to upgrade: after 2 weeks of stable Manual CPC data

Level 3: Maximize Conversions
├── Requirements: 15+ conversions/month
├── How it works: Google sets bids to get the most conversions within your budget
├── Risk: can spend full budget on low-quality clicks
├── CRITICAL: NEVER use with 0 conversion history + small budget — algorithm sits idle
└── When to upgrade: after 30+ conversions accumulated

Level 4: Target CPA
├── Requirements: 30+ conversions/month, stable CPA for 2+ weeks
├── How it works: Google bids to hit your target cost per acquisition
├── Risk: if target CPA is unrealistic, campaign stops spending
└── When to upgrade: after stable Maximize Conversions performance

Level 5: Target ROAS
├── Requirements: 50+ conversions/month, stable ROAS
├── How it works: Google bids to hit your target return on ad spend
├── Risk: requires accurate conversion value tracking
└── This is the endgame for ecommerce
```

**The exception:** Campaigns with > 1,000 RON/day budget AND high-volume category CAN start at Level 3 (Maximize Conversions). Document the reasoning.

---

## Romania-Specific Benchmarks

Romania CPCs are 50-70% lower than Western Europe. These are RO-specific ranges:

| Vertical | Avg CPC Search | Avg CPC Shopping | Avg CVR |
|----------|---------------|-----------------|---------|
| Beauty / Cosmetics | 0.80-1.80 RON | 0.50-1.50 RON | 1.5-3% |
| Ecommerce General | 0.60-1.50 RON | 0.40-1.20 RON | 1-2.5% |
| Auto Parts | 1.00-2.50 RON | 0.80-2.00 RON | 1-2% |
| Fashion | 0.50-1.20 RON | 0.30-0.90 RON | 1.5-3% |
| Premium Haircare | 5.00-6.00 RON | 2.00-4.00 RON | 1-2% |

**Premium haircare is an outlier** — competitors bid aggressively. Starting conservative (< 3 RON) means zero impression share.

---

## Common Google Ads Mistakes

### Broad Match + Manual CPC
The #1 wasted spend combination. Broad Match without Smart Bidding matches any vaguely related query. A search for "kerastase shampoo" on Broad could match "how to wash hair" or "cheap shampoo near me."

**Rule:** Broad Match only with Smart Bidding (tCPA, tROAS, MaxConv). Otherwise Phrase or Exact.

### Campaign Without Geo Targeting
A campaign without geo targeting serves globally. An ad in Romanian shown to someone in Japan wastes the click cost.

**Rule:** Always set geo_target_ids. Romania = 2642.

### Campaign Without Language Targeting
Even with geo targeting, an ad can show to someone browsing in English in Romania.

**Rule:** Always set language_ids. Romanian = 1032.

### Conversion Tracking Not Verified
Creating campaigns without verifying conversions are tracked means the algorithm can never learn. Zero conversions ≠ zero sales — it might mean tracking is broken.

**Rule:** Run attribution check (Ads vs GA4 vs Shopify) before creating new campaigns.

---

## GAQL Quick Reference

```sql
-- Top campaigns by spend
SELECT campaign.name, campaign.status, metrics.cost_micros, metrics.clicks, metrics.conversions
FROM campaign
WHERE segments.date DURING LAST_30_DAYS AND campaign.status = 'ENABLED'
ORDER BY metrics.cost_micros DESC
LIMIT 10

-- Keywords with low quality score
SELECT ad_group_criterion.keyword.text, ad_group_criterion.quality_info.quality_score,
       metrics.impressions, metrics.clicks, metrics.cost_micros
FROM keyword_view
WHERE segments.date DURING LAST_30_DAYS
  AND ad_group_criterion.quality_info.quality_score < 5
ORDER BY metrics.cost_micros DESC

-- Search terms wasting money
SELECT search_term_view.search_term, metrics.clicks, metrics.cost_micros, metrics.conversions
FROM search_term_view
WHERE segments.date DURING LAST_30_DAYS AND metrics.clicks > 5 AND metrics.conversions = 0
ORDER BY metrics.cost_micros DESC
LIMIT 20

-- Ad groups in a campaign
SELECT ad_group.id, ad_group.name, ad_group.status
FROM ad_group
WHERE campaign.id = 12345678
```

**GAQL rules:**
- No `SELECT *` — name every field
- ORDER BY fields must be in SELECT
- Metrics can't be in WHERE with resource fields (use HAVING or filter in code)
- `cost_micros` ÷ 1,000,000 = actual currency amount
- Status values are strings: `'ENABLED'`, `'PAUSED'`, `'REMOVED'`

---

## Google Ads API (v23)

**Known issues we've hit:**
- EU political advertising field is mandatory on some operations
- proto-plus objects with `maximize_clicks` can cause serialization issues
- Budget sharing between campaigns causes unexpected behavior — avoid
- Snippet headers must be from the official list (Amenities, Brands, Courses, etc.)
- Policy exemptions may be needed for healthcare/beauty terms in some regions

**Geo target IDs (common):**
| ID | Country |
|----|---------|
| 2642 | Romania |
| 2276 | Germany |
| 2040 | Austria |
| 2840 | USA |
| 2826 | UK |

**Language IDs (common):**
| ID | Language |
|----|----------|
| 1032 | Romanian |
| 1000 | English |
| 1001 | German |

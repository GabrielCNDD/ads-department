---
description: Cross-platform performance analysis with Shopify truth
---

Analyze ads performance for: $ARGUMENTS

## Boot

1. Read `reference/ads-safety-rules.md`
2. Read `reference/ads-decision-trees.md`
3. Read project's `ads/config.json` + `ads/learnings.md`

## Data Collection (parallel)

### Platform Data (last 30 days unless specified)
- Google Ads: campaigns, spend, clicks, conversions, CPA, ROAS, impression share
- Meta Ads: campaigns, spend, purchases, ROAS, CPA, CPM, CTR, frequency

### GA4 Data
- Sessions by source/medium
- Conversions by source/medium
- GDPR check: if clicks >> sessions (ratio > 2:1), note as normal EU consent

### Shopify Data (ground truth)
- Orders by attributed source
- Revenue by attributed source
- AOV, return rate

## Cross-Reference (workflow XP-1)

Build truth table:
| Source | Platform Reports | GA4 Reports | Shopify Reports | True ROAS |
|--------|-----------------|-------------|-----------------|-----------|

## Analysis

1. **Protected campaigns** — ROAS > 2x, mark as DO NOT TOUCH
2. **Problem campaigns** — spending but not converting (Shopify-verified)
3. **Wasted spend** — search terms, audience overlap, creative fatigue
4. **Opportunities** — impression share gaps, scaling candidates
5. **Trends** — CPA/CPM direction over time

## Present

Summary with Shopify-verified numbers. Flag discrepancies between platform-reported and actual. Prioritize: what to protect, what to fix, what to scale.

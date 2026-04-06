---
description: Weekly POAS (Profit on Ad Spend) audit across all platforms
---

Run a profit audit for: $ARGUMENTS

## Boot

1. Read `reference/ads-safety-rules.md`
2. Read `reference/ads-decision-trees.md` — Step 0 + XP-1
3. Read project's `ads/config.json` — unit_economics section required

## Gate 2: Data Collection

### Step 1: Shopify Ground Truth (last 7 days + last 30 days)
Pull from Shopify Admin API:
- Total revenue (gross)
- Total COGS (if available via product cost fields)
- Total shipping charged
- Total shipping cost (if tracked)
- Total refunds/returns
- Orders by attributed source (utm_source)

If COGS not in Shopify, calculate from config.json:
```
COGS = Revenue × unit_economics.cogs_pct
Shipping = Orders × unit_economics.shipping_per_order
Returns = Revenue × unit_economics.return_rate_pct
```

### Step 2: Platform Spend (same periods)
- Google Ads API: spend per campaign (last 7d + 30d)
- Meta Marketing API: spend per campaign (last 7d + 30d)

### Step 3: Attribution Cross-Reference
- Compare platform-reported revenue vs Shopify-attributed revenue
- If discrepancy > 20%: flag as Attribution Gap before proceeding
- Use Shopify numbers for all POAS calculations

## The POAS Calculation

For each active campaign AND for the account as a whole:

```
POAS = (Revenue - COGS - Shipping - Returns) ÷ Ad Spend
```

### Thresholds:

| POAS | Status | Action |
|------|--------|--------|
| > 1.2 | Scalable | Check if Protected. Propose +20% scale (MA-3). |
| 1.0 - 1.2 | Stable | Monitor. Check frequency > 5.0 (fatigue). Check creative freshness. |
| < 1.0 | Critical | Losing money per sale. Identify wasted spend. Propose -20% budget or kill. |

### Anti-Panic Rule:
If a campaign has HIGH ROAS but LOW POAS → the problem is product margins, not the ad. Flag the product economics (COGS too high, shipping eating margins) rather than blaming creative or targeting.

## Gate 4: Present to Gabriel

### Weekly Audit Summary Table:

| Campaign | Platform | Spend | Shopify Rev | Shopify ROAS | POAS | Status | Proposed Action |
|----------|----------|-------|-------------|-------------|------|--------|-----------------|

### Action Plan:
1. **POAS < 1.0 campaigns:** Propose 20% budget reduction. If spent > 3× AOV with no profit → propose kill.
2. **POAS > 1.2 campaigns:** Propose 20% scale-up (follow MA-3).
3. For every change: calculate Revenue at Risk ("If we pause X, we lose Y daily revenue but gain Z daily profit")
4. Check blocked operations: are we pausing a protected campaign without a test alongside?

### MER (Overall Health):
```
MER = Total Shopify Revenue ÷ Total Ad Spend (all platforms)
```
- MER > 3x: healthy
- MER 2-3x: acceptable
- MER < 2x: investigate

## Gate 6: Log

Log all findings and approved changes to `ads/CHANGELOG.md` with:
- POAS per campaign
- MER overall
- Actions taken
- 24-hour verification task: "Check POAS again tomorrow to confirm trend"

---
description: Generate monthly/quarterly ads performance report
---

Generate ads report for: $ARGUMENTS

## Boot

1. Read `reference/ads-operations-standard.md` — reporting standards (Section 4)
2. Read `reference/report-standard.md` — CN/D report format
3. Read project's `ads/config.json` + `PROJECT-CONTEXT.md`

## Data Collection (all platforms, specified period)

### Revenue (Shopify — ground truth)
- Gross revenue, discounts, returns, net revenue — month by month
- Source breakdown: Meta, Google, Organic, Direct (from Shopify attribution)
- Top 10 products sold
- Top brands by revenue
- New vs returning customers
- AOV trend

### Google Ads
- Spend, clicks, impressions, conversions, ROAS, CPA, CTR
- Campaign-level breakdown
- Search terms: top performers + top wasters
- Impression share on brand terms
- Quality score distribution

### Meta Ads
- Spend, purchases, ROAS, CPA, CPM, CTR, frequency
- Campaign-level breakdown
- Creative performance: top by ROAS, top by CTR
- Audience performance
- Learning phase status

### Cross-Platform
- Total ad spend as % of net revenue
- True ROAS per platform (Shopify-verified)
- Cost trends: CPM, CPC month over month
- Funnel: Impressions → Clicks → VC → ATC → IC → Purchase
- Device breakdown
- Day of week analysis
- Top cities

## Report Format

- HTML, mobile-first (reference/report-standard.md)
- CN/D header + footer (reference/proposal-template-standard.md)
- Client's language (Romanian for RO clients)
- Tables in scrollable `.table-wrap`
- `html, body { overflow-x: hidden }` mandatory
- Base64 platform logos from reference/assets/
- Deploy to tmp.cn-dots.com/{client}/

## Sections

1. Executive Summary (3 bullets: what happened, what worked, what to do next)
2. Revenue Overview (Shopify data)
3. Google Ads Performance
4. Meta Ads Performance
5. Cross-Platform Analysis
6. Top Products & Brands
7. Recommendations (prioritized, with budget impact)
8. Next Month Plan

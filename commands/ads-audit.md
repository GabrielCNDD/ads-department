---
description: Full multi-platform ads audit with safety gates
---

Run a full ads audit for: $ARGUMENTS

## Boot

1. Read `reference/ads-safety-rules.md`
2. Read `reference/ads-decision-trees.md`
3. Read `reference/ads-platform-specifics.md`
4. Read project's `PROJECT-CONTEXT.md` + `ads/config.json` + `ads/learnings.md`

## Step 0: Baseline (MANDATORY — before anything else)

1. Pull last 30 days performance for ALL active campaigns (all platforms)
2. Pull Shopify revenue by source (same period)
3. Cross-reference: platform-reported vs Shopify-attributed
4. Save snapshot to project's `ads/baselines/{date}-baseline.json`
5. Identify **protected campaigns** (ROAS > 2x, spending consistently, 10+ conversions)

## Platform Audits (dispatch in parallel where possible)

### Google Ads (workflow GA-6)
- Conversion tracking health (Ads vs GA4 vs Shopify)
- Wasted spend: search terms with cost + 0 conversions
- Keyword quality scores (flag < 5)
- Impression share (flag < 50% on brand)
- Bidding strategy alignment with conversion volume
- Feed quality (Shopping/PMax)
- Account structure: duplicates, orphans, overlap

### Meta Ads (workflow MA-6)
- Learning phase status per ad set
- Creative fatigue: frequency > 5.0
- Audience overlap
- DPA catalog health
- CAPI / pixel health
- Attribution gap: Meta-reported vs Shopify

### Cross-Platform (workflows XP-1, XP-2)
- Revenue reconciliation: Meta + Google vs Shopify actual
- Attribution overlap (both platforms claiming same purchase)
- Budget efficiency comparison (true ROAS per platform)
- Budget allocation recommendation

## Present Findings

Structure as:
1. **PROTECTED** — campaigns working well. Do not touch.
2. **WASTING** — campaigns/spend losing money. Quantify in RON.
3. **MISSING** — opportunities not being captured.
4. **RECOMMENDED ACTIONS** — prioritized, with risk assessment per XP-3.

## WAIT FOR GABRIEL'S APPROVAL before proposing any changes.

---
description: Scale profitable campaigns with safety guardrails
---

Scale campaigns for: $ARGUMENTS

## Boot

1. Read `reference/ads-safety-rules.md` — budget safety (Section 5), working account protection (Section 2)
2. Read `reference/ads-decision-trees.md` — MA-3 (Meta scaling), GA-4 (Google bidding)
3. Read project's `ads/config.json` — budget caps

## Gate 1: Is Scaling Justified?

1. Pull last 30 days performance
2. Cross-reference with Shopify (platform over-reports)
3. Verify profitability is REAL (Shopify-verified ROAS, not platform-reported)
4. Check stability: profitable for 7+ consecutive days, not a spike
5. Check learning status: must be stable (not "Learning" or "Learning Limited")

If not profitable on Shopify data → STOP. Don't scale what isn't working.

## Gate 2: Safety Limits

- Maximum budget increase: 20% per step
- Minimum wait between increases: 3-4 days
- BLOCKED: > 40% increase in one step
- BLOCKED: scaling + restructuring simultaneously
- Budget must stay within config.json caps

## Scaling Protocol

### Meta (MA-3):
1. Increase budget 20% on winning ad set
2. Wait 3-4 days, monitor CPA
3. If CPA rises > 30% → pause increase, stabilize
4. Budget split: 70% proven, 30% testing
5. Add creatives to winning ad sets before opening new ad sets
6. Clone winners for testing, keep originals live

### Google:
1. If limited by budget (Limited by Budget status): increase 20%
2. If limited by bids (low impression share): adjust bids per ads-bidding-logic.md
3. Check bidding ladder: enough conversions to upgrade strategy?
4. For Shopping: segment by price tier before scaling budget

## Present to Gabriel

- Current performance (Shopify-verified)
- Proposed increase (amount in RON)
- Expected impact
- Risk: "If CPA rises 30%, we lose X RON before correcting"
- Rollback plan

## After Approval

- Apply increase
- Log to ads/CHANGELOG.md
- Monitor 48h before next increase
- Report back on Day 3-4 with updated numbers

---
description: Create a new ad campaign with full safety checks
---

Create a new campaign for: $ARGUMENTS

## Boot

1. Read `reference/ads-safety-rules.md` — load ALL safety gates
2. Read `reference/ads-decision-trees.md` — follow GA-2 (Google) or MA-2 (Meta)
3. Read `reference/ads-bidding-logic.md` — for bid calculations
4. Read `reference/ads-platform-specifics.md` — platform rules
5. Read project's `ads/config.json` — budget caps, targets, account IDs

## Gate 1: Safety Check

- Is this a new campaign alongside existing ones? (OK)
- Or is this replacing a working campaign? (BLOCKED — Section 2 of safety rules)
- Does the account have conversion tracking working? If not → STOP, fix tracking first.

## Gate 2: Data Pull

1. Pull last 30 days performance for all existing campaigns
2. Identify protected campaigns
3. Check: does a similar campaign already exist? Avoid duplicates.

## Gate 3: Decision Tree

### Google (GA-2):
1. Bidding strategy from the ladder (ads-bidding-logic.md)
2. Calculate bids: `Max CPC = Target CPA × Expected CVR`, start at 65%
3. Geo targeting (Romania = 2642 unless specified)
4. Language targeting (Romanian = 1032 unless specified)
5. Budget check: within config caps, >= 5x target CPA

### Meta (MA-2):
1. Learning phase math: `Min budget/ad set = 50 × CPP ÷ 7`
2. Max viable ad sets = total budget ÷ min per ad set
3. Choose funnel stage (COLD/WARM/HOT)
4. Set promoted_object (pixel + PURCHASE) — immutable after creation
5. Creative format from hierarchy (ads-platform-specifics.md)

## Gate 4: Draft & Preview

Present to Gabriel:
- Campaign name (using ads-naming-convention.md)
- Platform, objective, bidding strategy
- Budget (daily) with calculation shown
- Targeting (geo, language, audience)
- Bid calculations with economics
- Keywords/creatives planned
- Risk assessment (XP-3)

## Gate 5 & 6: After Approval

- Dry run → verify API accepts
- Apply → campaign created as PAUSED
- Log to ads/CHANGELOG.md
- Verify delivery within 24h after enabling

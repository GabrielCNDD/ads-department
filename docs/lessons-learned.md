# Lessons Learned

_Every rule in this system exists because breaking it cost real money. Here are the incidents._

---

## Incident 1: The 0.01 RON Bids

**Date:** 2026-03-31
**Platform:** Google Ads (Shopping)
**Cost:** Full day of Shopping visibility lost

**What happened:** A new Shopping campaign was created with ad group bids set at 0.01 RON. With Manual CPC, the bid is the actual amount you're willing to pay per click. At 0.01 RON, the campaign could never win an auction — the minimum competitive bid for premium haircare in Romania is 5-6 RON.

**Why it happened:** The agent used a placeholder bid instead of calculating from economics. No pre-write validation checked if the bid was competitive.

**What it created:**
- Safety Rule: "Never set placeholder/round-number bids" (Section 4)
- Decision Tree: Bid calculation is mandatory, with formula shown (GA-2, step 5)
- Bidding Logic: Full calculation framework with industry benchmarks

---

## Incident 2: MaxConv on Zero History

**Date:** 2026-03-31
**Platform:** Google Ads (Brand Search)
**Cost:** 9+ hours of zero Brand Search coverage while competitors owned the brand terms

**What happened:** A Brand Search campaign was launched with Maximize Conversions bidding strategy but zero conversion history. The algorithm had no signal to bid on, so it bid nothing. The campaign got 0 impressions for 9+ hours while competitors captured all brand searches.

**Why it happened:** "Maximize Conversions" sounds like the right choice — but it needs 15-30 conversions of historical data to function. Without data, it's paralyzed.

**What it created:**
- Safety Rule: "Never launch MaxConv on 0-history campaigns with < 1000 RON/day budget" (Section 4)
- Decision Tree: The Bidding Ladder — Manual CPC first, upgrade only with enough data (GA-4)
- Blocked Operation: MaxConv + zero history + small budget = hard block

---

## Incident 3: The 500 EUR Rebuild

**Date:** 2026-04-01 to 2026-04-06
**Platform:** Meta Ads + Google Ads (Gossip)
**Cost:** ~500 EUR wasted + 1 week of lost revenue optimization

**What happened:** The Gossip account had 4.27x ROAS across 3 campaigns (362 purchases/month). Instead of scaling incrementally, we rebuilt from scratch: 6 new Kerastase ad sets, a WARM campaign, Display remarketing. We paused the Google Shopping workhorse.

The 6 Meta ad sets starved — the budget only supported 2 through learning phase. The Google Shopping replacement had worse bids. The working campaigns were paused.

Gabriel caught it on 2026-04-06.

**Why it happened:**
1. No data was pulled before proposing changes (skipped Gate 2)
2. No one checked what the current ROAS was (4.27x — exceptional)
3. The motivation was "proper structure" not "better results"
4. Learning phase math was not done (6 ad sets, budget for 2)
5. Winners were paused to make room for untested replacements

**What it created:**
- Safety Rule: "NEVER rebuild working accounts" (Section 2)
- Safety Rule: "Protected campaigns" concept — ROAS > 2x = untouchable
- Safety Rule: "Calculate change cost" — show revenue at risk
- Meta Safety: Learning phase math as a hard guard
- Meta Safety: "Clone-first policy" — test alongside, never replace
- Decision Tree: Step 0 (universal baseline pull) exists because of this incident

---

## Incident 4: Carousel on Cold Premium

**Date:** 2026-04 (Gossip Meta campaigns)
**Platform:** Meta Ads
**Cost:** Entire ad set budget with 0 purchases

**What happened:** Carousel ads were used as the default format for cold prospecting on premium haircare products (Kerastase, K18). After meaningful spend, the result was 0 purchases.

**Why it happened:** Carousel is a browse-heavy format. Cold premium audiences need fast belief transfer — proof, transformation, authority. Carousel delays the hook and weakens it across multiple cards.

**What it created:**
- Meta Safety: Creative format hierarchy by funnel stage (ads-platform-specifics.md)
- Blocked Operation: Carousel as primary cold premium format without prior proof
- Decision Tree: MA-4 (Creative Strategy) checks format-stage fit before proposing

---

## Incident 5: The Feed That Was Fine

**Date:** 2026-03-31
**Platform:** Google Ads (Shopping)
**Cost:** 9+ hours debugging a non-problem

**What happened:** A Shopping campaign wasn't getting impressions. The agent kept saying "check Merchant Center" and "verify the feed" based on generic troubleshooting advice. Meanwhile, another Shopping campaign on the exact same feed was serving 1,510 impressions.

The feed was fine. The real problems were: 0.01 RON bids, wrong bidding strategy, missing ad entity.

**Why it happened:** The agent followed generic "Shopping not working" troubleshooting instead of reading the data it already had. If another campaign on the same feed is serving, the feed isn't the problem.

**What it created:**
- Core Rule: "DATA FIRST — read the data you already have before chasing generic advice"
- Decision Tree: GA-5 (Shopping/PMax) checks existing campaign performance before diagnosing feed issues
- Safety Rule: "Verify before presenting" (Section 7)

---

## Incident 6: Meta Attribution Over-Reporting

**Date:** Ongoing
**Platform:** Meta Ads (all clients)
**Impact:** Decisions based on inflated numbers

**What happened:** Meta consistently reports 30-80% more conversions than Shopify attributes to Meta traffic. This is a known platform behavior — Meta uses view-through attribution and claims credit for conversions that would have happened anyway.

Making decisions based on Meta-reported ROAS leads to over-spending on underperforming campaigns.

**What it created:**
- Core Rule: "Cross-reference Platform + GA4 + Shopify. Shopify is truth for revenue."
- Decision Tree: XP-1 (Attribution Reconciliation) — mandatory before budget allocation
- Every workflow that mentions ROAS specifies "Shopify-verified ROAS"

---

## The Pattern

Every incident follows the same pattern:
1. An action was taken without sufficient data or validation
2. A guardrail that should have existed didn't
3. The mistake was caught by the operator (Gabriel), not the system
4. Real money was lost in the gap between action and correction

The 6-gate system exists to catch these mistakes at the system level, before they cost money.

# CND Ads Decision Trees

_Step-by-step workflows for every ads scenario. Follow the steps. Don't skip._
_Requires: ads-safety-rules.md loaded first._

---

## Step 0: Universal Pre-Action Checklist

**Every workflow starts here. No exceptions.**

Before proposing any change to any ad account:

1. Read project's `ads/config.json` — budget caps, targets, account IDs
2. Read project's `ads/learnings.md` — past mistakes and wins for this client
3. **Seasonality & external context check (Gate 0.5):**
   - What is today's date? Is it within 7 days of a major shopping event?
   - Client-specific events: check PROJECT-CONTEXT.md for seasonality notes
   - Common RO events: Black Friday (Nov), Easter, Back to School (Sep), Craciun (Dec)
   - **If within a seasonal window:** prioritize last 7 days over 30-day averages for scaling/bidding decisions. The 30-day lookback includes "normal" days that are no longer relevant during peaks.
   - **If post-seasonal:** don't panic at declining metrics — return to baseline is normal.
4. Pull last 30 days performance for ALL active campaigns (all platforms)
5. Pull Shopify revenue by source for the same period
6. Cross-reference: platform-reported revenue vs Shopify-attributed revenue
7. **Verify attribution windows match:** Google and Meta must use the same window (7-day click recommended) for the cross-platform comparison table to be valid. Note any mismatches.
8. Identify **protected campaigns** — a campaign is protected if:
   - Shopify-verified ROAS > 1.2× the `target_roas` in `ads/config.json`
   - Spending consistently (no 0-spend days in last 14 days)
   - 10+ conversions in last 30 days
   - _Note: "ROAS > 2x" is the default if no target_roas is set. But a client with target_roas: 4.0 needs ROAS > 4.8 to be protected, while a volume client with target_roas: 1.5 is protected at ROAS > 1.8._
9. Calculate the cost of any proposed change: "If we pause X, we lose Y RON/day"
10. Calculate **MER (Marketing Efficiency Ratio):** `Total Shopify Revenue ÷ Total Ad Spend (all platforms)`. This is the CEO metric — the one number that tells you if the ads department is making or losing money overall.
11. Check: is this account in a working state? If yes, tread carefully.

**If Step 0 reveals the account is profitable and stable:**
- Default to adding alongside, not replacing
- Default to small increments, not restructures
- Ask: "Does this change risk breaking what's working?"

---

## Google Ads Workflows

### GA-1: Performance Analysis

**Trigger:** "How are our Google Ads doing?" / "What's the ROAS?" / periodic review

1. Pull campaign performance (last 30 days) — spend, clicks, conversions, CPA, ROAS
2. Pull Shopify orders attributed to Google (same period)
3. Cross-reference: Google reports X conversions, Shopify shows Y orders
4. Pull GA4 session data by source/medium — check for GDPR gaps
5. If clicks >> GA4 sessions (ratio > 2:1): note as normal EU consent behavior, not broken tracking
6. Check search terms report — look for wasted spend (clicks + cost, 0 conversions)
7. Check keyword quality scores — any below 5 need attention
8. Check impression share — if < 50%, bids may be too low

**Present:** Summary table, protected campaigns marked, problem areas flagged, Shopify-verified numbers.

### GA-2: Campaign Creation

**Trigger:** "Create a new Google campaign" / "Launch Search/Shopping/PMax"

1. **Step 0** — full baseline pull
2. Check: does a similar campaign already exist? Avoid duplicates.
3. Check: is conversion tracking working? Run attribution check.
   - If zero conversions across the account: **STOP**. Fix tracking first. A new campaign won't help.
4. Determine bidding strategy (from ads-bidding-logic.md):
   - Budget < 1000 RON/day: Manual CPC (follow the ladder)
   - Budget >= 1000 RON/day + high volume: Maximize Conversions acceptable
5. Calculate bids:
   ```
   Max CPC = Target CPA × Expected CVR
   Start bid = Max CPC × 0.65 (60-70% of max)
   ```
   Cross-check against industry benchmarks (ads-bidding-logic.md)
6. Verify geo targeting (Romania = 2642, or client-specific)
7. Verify language targeting (Romanian = 1032, or client-specific)
8. Check budget against config.json caps
9. Check budget sufficiency: daily budget >= 5x target CPA (minimum), 10x (recommended)
10. Draft campaign — create as PAUSED
11. **Present to Gabriel:** campaign name, bidding, budget, targeting, keywords, bid calculations
12. Gabriel approves → dry run → apply
13. After creation: verify serving within 24h, check impression share

**Naming:** `CND_G_[TEMP]_[OBJECTIVE]_[STRATEGY]_[GEO]`

### GA-3: Keyword Management

**Trigger:** "Add keywords" / "Check search terms" / "Negative keywords"

1. Check campaign's bidding strategy first
2. If Manual CPC/Manual CPM: ONLY Exact or Phrase match. **Broad Match is BLOCKED.**
3. If Smart Bidding: Broad Match is allowed (but still verify it makes sense)
4. Pull existing keywords — avoid duplicates
5. Pull search terms report — understand what's already triggering
6. For new keywords: calculate expected CPC, verify search volume exists
7. For negative keywords: group by theme, check for conflicts with positive keywords
8. Draft → present → approve → apply (one batch at a time)

### GA-4: Bidding Strategy Changes

**Trigger:** "Switch to MaxConv" / "Change bidding" / "Improve bids"

1. Check current strategy and conversion volume
2. Reference the bidding ladder:
   - Manual CPC → eCPC: need 10+ conversions
   - eCPC → MaxConv: need 15+ conv/month
   - MaxConv → tCPA: need 30+ conv/month, stable CPA 2+ weeks
   - tCPA → tROAS: need 50+ conv/month, stable ROAS
3. If moving DOWN the ladder (e.g., MaxConv → Manual): warn this is unusual, ask why
4. Check if campaign is in learning phase — if yes, DO NOT change. Wait.
5. Present: current state, proposed change, data justification, risk
6. Gabriel approves → apply

### GA-5: Shopping / PMax Campaigns

**Trigger:** "Launch Shopping" / "PMax performance" / "Feed issues"

1. Check Merchant Center: feed approved? Products active? Disapprovals?
2. If feed issues: fix those FIRST. Don't create campaigns on a broken feed.
3. For Shopping: segment by product price tier (different bids for different margins)
4. For PMax: check asset group ad strength, need minimum diversity:
   - 5+ headlines (30 char), 5+ descriptions (90 char), 5+ images, 1+ logo
5. **Cannibalization check (CRITICAL):** Compare Brand Search conversion volume before and after PMax launch. If Brand Search volume dropped >20% since PMax went live, PMax is likely cannibalizing high-intent traffic that would have converted cheaper through Search. **Action:** Add brand terms as negative keywords in PMax, or pause PMax and let Brand Search recapture that traffic.
6. Cross-reference PMax network breakdown: `segments.ad_network_type` shows SEARCH vs CONTENT vs YOUTUBE vs MIXED. If most PMax spend is on SEARCH, it's competing with your Search campaigns.
7. Budget: PMax needs higher minimum (50 RON/day absolute minimum for RO market)

### GA-6: Account Audit

**Trigger:** "Audit the Google account" / periodic review

1. Step 0 — full baseline (including seasonality check and MER)
2. Check conversion tracking: Ads conversions vs GA4 vs Shopify
3. Wasted spend: search terms with cost + 0 conversions → propose negatives
4. Quality scores: keywords below 5 → investigate ad relevance + landing page
5. Impression share: if < 50% on brand terms → bids too low
6. Bidding strategy alignment: Manual CPC on high-volume campaigns → propose upgrade
7. Feed quality (Shopping/PMax): disapprovals, missing attributes
8. **PMax cannibalization check:** Compare Brand Search volume before/after PMax. If Brand Search dropped >20%, PMax is stealing high-intent traffic. Propose brand negatives in PMax.
9. Account structure: duplicate campaigns, overlapping keywords, orphaned ad groups
10. Present findings in priority order: tracking > cannibalization > wasted spend > QS > bidding > structure

---

## Meta Ads Workflows

_Reference: ads-platform-specifics.md for detailed Meta safety rules._

### MA-1: Performance Analysis

**Trigger:** "How are Meta Ads doing?" / "What's the ROAS on Meta?"

1. Pull all active campaigns (last 30 days) — spend, purchases, ROAS, CPA, CPM, CTR
2. Pull Shopify orders attributed to Meta (same period)
3. Cross-reference: Meta says X purchases, Shopify attributes Y
4. Check frequency per ad set — >5.0 = fatigue signal
5. Check learning phase status per ad set — learning/limited/stable
6. Check creative performance — which ads have declining CTR?
7. Check cost trends — CPM increasing? CPA increasing?

**Present:** Campaign table with Shopify-verified numbers, fatigue flags, learning status.

### MA-2: Campaign Creation

**Trigger:** "Create a Meta campaign" / "Launch prospecting" / "Set up remarketing"

1. **Step 0** — full baseline pull
2. **Learning phase math FIRST:**
   ```
   Min budget per ad set = 50 × CPP ÷ 7 (daily)
   Max viable ad sets = Total daily budget ÷ Min per ad set
   ```
   If budget supports 2 ad sets: build 2. Not 6.
3. Choose funnel stage:
   - COLD: broad or strategic segmentation, proof-led creative
   - WARM: website visitors, video viewers, engagement (real audiences only)
   - HOT: remarketing with custom audiences, ADV+ audience OFF
4. Set at creation (immutable):
   - `promoted_object`: pixel_id + PURCHASE
   - `is_adset_budget_sharing_enabled`: false
   - `special_ad_categories`: ["NONE"]
5. Choose creative format (from ads-platform-specifics.md hierarchy):
   - Cold premium: video/UGC > static > DPA >> carousel (last resort)
   - Cold commodity: video > static > DPA/collection
   - Warm: best cold winner + proof
   - Hot: DPA + proven winners
6. Set bid strategy: `LOWEST_COST_WITHOUT_CAP` for new campaigns
7. Set `targeting_automation.advantage_audience: 0` on remarketing ad sets
8. Draft → present with learning phase math shown → Gabriel approves → apply

**Naming:** `CND_M_[TEMP]_[OBJECTIVE]_[STRATEGY]_[GEO]`

### MA-3: Scaling

**Trigger:** "Scale Meta" / "Increase Meta budget" / "This campaign is doing well"

#### Step 1: The "Profit Floor" Verification
1. **Shopify truth check:** Verify the campaign has been profitable (Shopify-verified ROAS > target_roas from config.json) for at least **7 consecutive days**. Not a spike — sustained profit.
2. **Stability check:** Ad set must be in "Active" state (not "Learning" or "Learning Limited").
3. **EXCEPTION:** If status is "Learning Limited" AND the budget increase is specifically calculated to push it past the 50-conversion threshold — this is allowed. Document the math.

#### Step 2: The Scaling Math (20/20 Rule)
1. **Incremental cap:** Maximum 20% budget increase per step.
2. **Temporal cap:** Wait minimum **48-72 hours** between increases. The algorithm needs time to stabilize at the new "auction temperature."
3. **Anti-shock rule:** Total budget increases must not exceed **40% within a single 7-day window**.
4. **Budget split:** 70% on proven campaigns, 30% on testing.
5. **BLOCKED:** scaling + restructuring in the same change.

#### Step 3: The "Revenue at Risk" Pitch (Gate 4)
Present to Gabriel with these exact numbers:
- **Current state:** "This campaign generates **X RON/day** at **Y ROAS** (Shopify-verified)."
- **The proposal:** "Increase budget from **A** to **B** (20% increase)."
- **Revenue at risk:** "If the algorithm resets and performance drops to account average, we risk **Z RON/day** in existing profit."
- **The kill switch:** "If CPA rises >30% over the next 48h, we revert to previous budget."

#### Step 4: The ROAS-vs-Volume Decision
If scaling fails (CPA rises after increase):
1. **First move:** Revert budget to pre-increase level. Don't lower bids.
2. **Diagnose:** Is the issue auction saturation (CPM rising) or creative fatigue (CTR dropping)?
3. **If CPM rising:** The audience is tapped at this budget. Add new creative or expand audience before retrying.
4. **If CTR dropping:** Creative fatigue. Refresh creatives first, then retry scaling.
5. **Never:** Lower bids to compensate for failed scaling. That just gets worse delivery at worse quality.

### MA-4: Creative Strategy

**Trigger:** "New creatives" / "Creative fatigue" / "What formats to test"

1. Check what's currently working (CTR, conversion rate by ad)
2. Check frequency — >5.0 on any ad = refresh needed
3. Check format performance: which format class converts best in this account?
4. Follow format hierarchy for the funnel stage (ads-platform-specifics.md)
5. In-account proof > generic best practice
6. If proven format exists: iterate angles within that format first
7. If no proof exists: start with safest format for the stage
8. Test one variable at a time: new angle in same format, OR new format with proven angle
9. Minimum 3-4 creatives per ad set

### MA-5: Troubleshooting

**Trigger:** "Campaign not spending" / "CPA too high" / "Learning phase stuck"

**No delivery / weak spend:**
1. Check: too many ad sets for the budget? → consolidate
2. Check: audience too narrow? → broaden or use ADV+
3. Check: bid too restrictive? → try LOWEST_COST
4. Check: creative disapproved? → fix creative
5. Fix order: consolidate → simplify → then adjust budget

**Spend but no purchases:**
1. Check tracking (pixel firing? CAPI healthy?)
2. Check optimization event (optimizing for PURCHASE, not LINK_CLICK?)
3. Check landing page (loads fast? mobile-friendly? clear CTA?)
4. Check creative-to-landing-page match
5. Check: wrong format for stage? (carousel on cold premium?)
6. Fix order: tracking → landing page → creative → structure

**Learning phase stuck (>7 days):**
1. Check: enough budget per ad set? (50 × CPP ÷ 7)
2. Check: too many ad sets splitting the budget?
3. Check: audience too narrow for the conversion event?
4. Fix: consolidate ad sets first, then increase budget if needed

### MA-6: Account Audit

**Trigger:** "Audit Meta account" / periodic review

1. Step 0 — full baseline with Shopify cross-reference
2. Learning phase: how many ad sets are stuck in learning?
3. Creative fatigue: frequency >5.0 on any active ad?
4. Audience overlap: check with Meta's tool — overlapping ad sets waste budget
5. DPA catalog health: all products active? Prices correct? Images loading?
6. CAPI status: server-side events firing?
7. Attribution: Meta-reported vs Shopify — what's the gap?
8. Structure: too many campaigns? Unnecessary ad sets? Budget fragmentation?
9. Protected campaigns: mark profitable ones, do NOT propose changes to them
10. Present: what's working (protect), what's wasting (fix), what's missing (add)

---

## Cross-Platform Workflows

### XP-1: Attribution Reconciliation

**Trigger:** Monthly review / "What's the real ROAS?" / budget allocation

1. Pull platform data: Google (spend, conversions, revenue), Meta (same)
2. Pull GA4 data: sessions, conversions by source/medium
3. Pull Shopify data: orders, revenue, by attributed source
4. Build the truth table:

| Source | Platform Says | GA4 Says | Shopify Says | **Truth** |
|--------|-------------|----------|-------------|-----------|
| Google | X conv | Y conv | Z orders | Shopify |
| Meta | A conv | B conv | C orders | Shopify |
| Organic | — | D conv | E orders | Shopify |

5. Both platforms will over-report (they both claim credit for the same purchase)
6. Shopify order data is the ground truth for revenue
7. GA4 is the ground truth for on-site behavior (with GDPR caveat)
8. Calculate **MER (Marketing Efficiency Ratio):** `Total Shopify Revenue ÷ Total Ad Spend`. This is the single number that tells you if the ads department is profitable overall. MER > 3x is healthy for ecommerce. Present this alongside per-platform ROAS.
9. Present: true ROAS per platform, true CPA per platform, MER, budget efficiency

### XP-2: Budget Allocation Across Platforms

**Trigger:** "Where should we spend more?" / "Allocate budget"

1. Run XP-1 first — need true ROAS per platform
2. Calculate marginal efficiency: which platform's next RON produces the most revenue?
3. Allocate more to the platform with better Shopify-verified ROAS
4. Minimum viable budget per platform: don't go below learning phase requirements
5. Test new platforms with 10-15% of total budget, not 50%
6. Monthly review: rebalance based on actual results

### XP-3: Risk Assessment (Before Any Change)

**Trigger:** Before any proposed change passes to Gabriel

Answer these questions:
1. **What could go wrong?** List specific failure modes
2. **What's the worst-case cost?** In RON, not theory
3. **Is there a less risky alternative?** Add alongside instead of replace?
4. **Can we test first?** Small budget test before full rollout?
5. **What's the rollback plan?** How to undo if it fails?
6. **Does this touch a protected campaign?** If yes, extra scrutiny

### XP-4: Emergency Red Alert

**Trigger:** Technical failure, tracking loss, or "black swan" performance drops.

**This is the ONLY workflow where the agent can act without Gabriel's prior approval.**

#### Auto-Pause Triggers (act first, notify immediately):
1. **Tracking failure:** Shopify reports 0 orders from a source (Meta/Google) for 12+ hours while the platform reports >500 RON spend. → PAUSE affected campaigns. Log to CHANGELOG: "EMERGENCY PAUSE — tracking loss detected."
2. **Landing page down:** Destination URL returns 404 or 500 error. → PAUSE ALL campaigns pointing to that URL immediately. Log: "EMERGENCY PAUSE — landing page [URL] returning [status code]."
3. **Budget runaway:** Campaign spend exceeds 3x daily budget in a single day (platform glitch). → PAUSE the campaign. Log: "EMERGENCY PAUSE — spend anomaly."

#### Investigate-First Triggers (flag to Gabriel, don't auto-pause):
4. **CPA spike:** CPA rises >100% compared to 7-day average across all campaigns. → Flag as urgent, present data, recommend pause.
5. **Zero conversions:** 48+ hours of spend with zero Shopify-verified conversions across entire account. → Flag as urgent, check tracking first.

#### After Any Emergency:
1. Log the full incident to `ads/CHANGELOG.md` with timestamp, action taken, reason
2. Notify Gabriel immediately with: what happened, what was paused, estimated revenue protected
3. Do NOT re-enable without Gabriel's approval
4. Investigate root cause before any campaigns go back live

---

## Quick Reference: Which Workflow?

| User wants to... | Workflow | Platform |
|-------------------|----------|----------|
| Check performance | GA-1 / MA-1 | Google / Meta |
| Create campaign | GA-2 / MA-2 | Google / Meta |
| Add keywords | GA-3 | Google |
| Change bidding | GA-4 | Google |
| Shopping/PMax work | GA-5 | Google |
| Scale a campaign | MA-3 | Meta |
| New creatives | MA-4 | Meta |
| Fix a problem | MA-5 | Meta |
| Full audit | GA-6 / MA-6 | Google / Meta |
| Monthly review | XP-1 + XP-2 | Cross-platform |
| Propose any change | XP-3 | All |
| Tracking down / page 404 / budget runaway | XP-4 (Red Alert) | All — auto-pause allowed |

## Quick Reference: Safety Limits

| Action | Limit | Reason |
|--------|-------|--------|
| Scaling | Max 20% per step, 40% per 7-day window | Avoids re-entering learning phase |
| New ad sets | (Total Budget ÷ CPP × 50) / 7 | Prevents starving ad sets of data |
| Broad Match | BLOCKED on Manual CPC | Prevents uncontrolled wasted spend |
| Protected status | ROAS > 1.2× client target (or > 2x default) | Prevents restructuring what works |
| Emergency pause | Auto-allowed for tracking loss / 404 / budget runaway | Protects client capital |
| Budget increase | Needs 7 days profitable + 48h between steps | Ensures stable profit floor |

# How the 6-Gate System Works

Every ad account mutation — creating a campaign, changing a bid, pausing an ad, scaling budget — passes through 6 gates. Each gate can stop the process. This document explains what each gate does, why it exists, and what it catches.

---

## Gate 1: Safety Rules

**File:** `rules/ads-safety-rules.md`

**What it does:** Checks if the proposed operation is outright blocked before anything else happens.

**What it catches:**
- Attempting to delete an ad entity (blocked — pause + rename instead)
- Attempting to pause a profitable campaign (blocked — unless operator explicitly approves with revenue-at-risk shown)
- Broad Match + Manual CPC combination on Google (blocked — #1 wasted spend cause)
- Maximize Conversions on a new campaign with zero history and small budget (blocked — algorithm sits idle)
- More Meta ad sets than the budget can support through learning phase (blocked — all will fail)
- Budget increase > 40% in one step (blocked — resets learning)

**Why it exists:** Every blocked operation in this list cost real money when it wasn't blocked. The 0.01 RON bid incident cost a full day of Shopping visibility. The Gossip rebuild cost ~500 EUR. The 6-ad-set Meta launch wasted an entire week's budget.

**How it works:** The AI agent reads the safety rules at boot. When a mutation is proposed, it checks against the blocked operations list. If blocked, it stops and explains why. No exceptions without explicit operator override.

---

## Gate 2: Data Pull

**What it does:** Pulls current performance data from all relevant sources before any decision is made.

**Sources (in trust order):**
1. **Shopify** — actual orders, actual revenue (ground truth)
2. **GA4** — sessions, on-site behavior (reduced by GDPR consent)
3. **Ad Platform** — clicks, impressions, platform-attributed conversions (over-reports)

**What it catches:**
- Proposing changes to an account without knowing what's working
- Missing that a campaign has 4.27x ROAS (and should be protected, not restructured)
- Trusting platform-reported ROAS when Shopify tells a different story
- Misdiagnosing GDPR consent gaps as tracking issues

**The cross-reference:**
```
Platform says 100 conversions
GA4 says 60 conversions (GDPR consent reduces this)
Shopify says 55 orders from that source
→ True number is ~55. Platform over-reports by ~80%.
```

**Why it exists:** On 2026-04-06, we rebuilt an account that had 4.27x ROAS because we didn't pull the data first. We looked at the structure and decided it "needed improvement" instead of looking at the numbers and seeing it was printing money.

---

## Gate 3: Decision Tree

**File:** `rules/ads-decision-trees.md`

**What it does:** Routes the request to the correct step-by-step workflow based on what the operator wants to do. Each workflow includes platform-specific pre-write validation.

**Workflows available:**
- GA-1 through GA-6: Google Ads (performance, campaign creation, keywords, bidding, Shopping/PMax, audit)
- MA-1 through MA-6: Meta Ads (performance, campaign creation, scaling, creative, troubleshooting, audit)
- XP-1 through XP-3: Cross-platform (attribution reconciliation, budget allocation, risk assessment)

**What it catches:**
- Skipping pre-write validation (e.g., creating a campaign without checking if conversion tracking works)
- Wrong bidding strategy for the conversion volume available
- Wrong creative format for the funnel stage
- Budget that doesn't support the proposed structure
- Missing geo/language targeting

**How it works:** Each workflow has numbered steps. The agent follows them in order. Steps that say "check" or "verify" must produce a concrete answer before proceeding. Steps that say "calculate" must show the math.

---

## Gate 4: Draft & Preview

**What it does:** Builds a structured change plan and presents it to the operator for approval.

**The preview includes:**
- What specifically changes
- Why (data justification, not theory)
- Expected impact (quantified)
- Risk (what could go wrong)
- Revenue at risk (if touching a working campaign)
- Cost of the change
- Whether it's reversible
- Dry run result

**What it catches:**
- Changes that look reasonable in isolation but are risky in context
- Missing risk that the agent didn't flag
- Operator catching a mistake before it costs money

**Why it exists:** The operator is the last human check. Every past mistake could have been caught here if the preview had been shown. The rule is: present everything, hide nothing, let the operator decide.

---

## Gate 5: Dry Run

**What it does:** Executes the change with `validate_only=true` (Google Ads) or equivalent API validation. The API checks if the request is valid without actually applying it.

**What it catches:**
- Invalid entity IDs
- Policy violations (ad disapprovals before they go live)
- API-level errors (wrong field types, missing required fields)
- Budget constraint violations

**Why it exists:** AdLoop's original dry run was fake — it logged the request and returned success without actually calling the API's validation mode. We found this during our review. Real dry runs catch real errors.

---

## Gate 6: Execute & Log

**What it does:** Applies the change and creates an audit trail.

**The audit log records:**
- Timestamp
- Platform and operation
- What changed (specific fields and values)
- Why (business reason)
- Who approved
- Whether it's reversible
- Dry run result
- Expected impact

**Post-execution verification:**
- Check delivery within 24h (is the campaign serving?)
- Check impression share (are bids competitive?)
- Check for disapprovals (any ads rejected?)

**Why it exists:** Without an audit trail, you can't diagnose what went wrong. When CPA doubles, the first question is "what changed?" — the changelog answers it instantly.

---

## What Makes This Different from AdLoop

AdLoop pioneered the safety-first approach for Google Ads. We extended it in several ways:

1. **Multi-platform:** Google + Meta + TikTok, not just Google
2. **Shopify as truth:** Three-way attribution reconciliation, not just Ads vs GA4
3. **Protected campaigns:** The concept of "don't touch what's working" enforced at the system level
4. **Meta-specific safety:** Learning phase math, creative format hierarchy, mutation decomposition
5. **Agency scale:** Per-client configs, not single-account
6. **Real dry runs:** Actual API validation, not just logging
7. **Context-aware:** AI agent can understand "this campaign has 4.27x ROAS" — code-only guards can't

# CND Ads Department

**A universal AI-powered ad operating system with safety guardrails for multi-platform paid advertising.**

Built by [Connecting The Dots (CN/D)](https://cn-dots.com) — inspired by [AdLoop](https://github.com/kLOsk/adloop)'s safety-first approach to Google Ads, extended to cover Meta Ads, cross-platform attribution, profit-based optimization, and agency-scale operations.

---

## Why This Exists

Managing ad accounts with AI assistants is powerful but dangerous. Without guardrails, AI agents make expensive mistakes:

- Setting bids at 0.01 when the minimum competitive bid is 5.00
- Launching Maximize Conversions on campaigns with zero conversion history
- Rebuilding profitable accounts from scratch instead of scaling what works
- Creating 6 ad sets when the budget only supports 2 through learning phase
- Using carousel ads for cold premium audiences (proven 0 purchases)
- Targeting Russian speakers instead of Romanian (real bug we caught)

**Every rule in this system exists because breaking it cost real money.**

This system enforces a 6-gate safety model where every ad account mutation must pass through validation before execution. It's brand-agnostic, platform-agnostic, and designed to plug into any ecommerce client.

---

## Real-World Example: What Happens When You Run This

Here's what the system did on its first real audit of a client's Google Ads account (alloy wheels ecommerce, Romania):

### The Audit

The operator ran `/ads-audit` on account 843-198-7884. The system:

1. **Gate 1 (Safety Check):** Found 3 violations in the live account:
   - Maximize Conversions on campaigns with 0 conversion history → **BLOCKED by Section 4**
   - Ad group bids at 0.01 RON (minimum competitive is 2.00+) → **BLOCKED by Section 4**
   - Page View events counted as primary conversions → **BLOCKED by Section 7**

2. **Gate 2 (Data Pull):** Pulled all campaign data via Google Ads API. Cross-referenced. Found:
   - 30 total impressions across 2 enabled campaigns (essentially dead)
   - Search Impression Share: 10% — losing 90% of auctions
   - **Language targeting set to Russian (1031) instead of Romanian (1032)** — this was the root cause. Ads only showed to the ~0.1% of Romanian users with Russian language settings.

3. **Gate 3 (Decision Tree):** Followed GA-6 (Account Audit):
   - Calculated bids from economics: AOV 1,450 RON × 15% target CPA = 217 RON × 1.5% CVR = 3.26 RON max CPC → start at 2.00 RON
   - Checked all ad landing page URLs → found 2 returning 404 (Skoda and VW collections had wrong URLs)

4. **Gate 4 (Preview):** Presented all findings to the operator with revenue impact calculated.

5. **Gate 5 + 6 (Execute):** After approval, applied all fixes via the Google Ads API in one session:

### The Fixes (all via API)

| Fix | Before | After |
|-----|--------|-------|
| Conversion tracking | 5 actions primary (incl Page View) | Only Purchase primary |
| Bidding strategy | Maximize Conversions (0 history) | Manual CPC |
| Ad group bids | 0.01 RON (13 ad groups) | 2.00-2.50 RON (calculated) |
| Language targeting | Russian (1031) | Romanian (1032) |
| Broken ads | 2 ads pointing to 404 URLs | Recreated with correct URLs |
| Shopping campaign | None | Created with correct Merchant Center, 3 brand-tier ad groups |

### Then Meta

Same session, the system audited the Meta account:

- Found 25 paused campaigns from previous agency (84K RON lifetime, 4.32x ROAS)
- Ran learning phase math: 500 RON CPP → budget supports max 1 cold ad set on purchase optimization
- Created 9 DPA catalog ads by brand (Mercedes, BMW, Audi, VW, Porsche, Skoda, SUV, Reduceri, Full Catalog)
- Created 1 DPA remarketing ad for hot traffic
- All with UTM parameters, proper headlines, Shopify product catalog connected

### Result

Total time: one session. The account went from 30 impressions and Russian targeting to a fully operational 4-campaign Google setup + 2-campaign Meta setup with proper bids, tracking, and catalog ads — all through the API, all logged, all reversible.

**None of the bugs would have been caught without the system cross-referencing language targeting, bid amounts, conversion actions, and impression share data together in one pass.**

---

## The 6-Gate Safety Model

```
USER REQUEST
     │
     ▼
┌─────────────────────────────┐
│  GATE 0.5: CONTEXT          │  Seasonality check. Is this a peak period?
│  Check date, events, peaks  │  If yes: prioritize 7d data over 30d.
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  GATE 1: SAFETY RULES       │  Hard stops. Is this operation blocked?
│  ads-safety-rules.md        │  Budget caps, protected campaigns,
│                             │  blocked operations.
└─────────────┬───────────────┘
              │ PASS
              ▼
┌─────────────────────────────┐
│  GATE 2: DATA PULL          │  Pull performance from ALL sources.
│  Platform + GA4 + Shopify   │  Calculate MER + POAS.
│  (Shopify = ground truth)   │  Identify protected campaigns.
└─────────────┬───────────────┘
              │ DATA READY
              ▼
┌─────────────────────────────┐
│  GATE 3: DECISION TREE      │  Follow platform-specific workflow.
│  22 workflows available     │  Pre-write validation.
│  ads-decision-trees.md      │  Calculate bids, budgets, learning math.
└─────────────┬───────────────┘
              │ DECISION MADE
              ▼
┌─────────────────────────────┐
│  GATE 4: DRAFT & PREVIEW    │  Create change plan.
│  Present to operator        │  Show: what, why, POAS impact,
│  (or auto-approve if        │  revenue at risk, reversibility.
│   within thresholds)        │
└─────────────┬───────────────┘
              │ APPROVED
              ▼
┌─────────────────────────────┐
│  GATE 5: DRY RUN            │  Execute with validate_only=true.
│  API validation             │  Verify no errors before real apply.
└─────────────┬───────────────┘
              │ DRY RUN OK
              ▼
┌─────────────────────────────┐
│  GATE 6: EXECUTE & LOG      │  Apply the change.
│  Audit trail                │  Log to CHANGELOG. Verify delivery.
│                             │  24h verification task.
└─────────────────────────────┘
```

---

## Architecture

```
ads-department/
├── README.md                          ← You are here
├── LICENSE
│
├── rules/
│   ├── ads-safety-rules.md            ← Gate 1: Hard stops & blocked operations
│   ├── ads-decision-trees.md          ← Gate 3: 22 workflows + safety mechanisms
│   ├── ads-platform-specifics.md      ← Platform safety: Meta guards, creative hierarchy
│   └── ads-bidding-logic.md           ← Bid calculation from economics
│
├── agent/
│   └── cnd-ads-manager.md             ← AI agent definition with 6-gate enforcement
│
├── commands/                           ← Workflow entry points (slash commands)
│   ├── ads-audit.md                   ← Full multi-platform audit
│   ├── ads-create-campaign.md         ← Campaign creation with safety checks
│   ├── ads-performance.md             ← Cross-platform performance analysis
│   ├── ads-scale.md                   ← Safe scaling with guardrails
│   ├── ads-report.md                  ← Monthly/quarterly report generation
│   └── ads-profit-audit.md            ← Weekly POAS profit check
│
├── templates/
│   ├── ads-config.json                ← Per-client config (targets, economics, caps)
│   ├── ads-changelog.md               ← Mutation audit log template
│   └── ads-naming-convention.md       ← Campaign naming standard
│
└── docs/
    ├── how-it-works.md                ← Detailed explanation of every gate
    ├── google-ads.md                  ← Google-specific logic & gotchas
    ├── meta-ads.md                    ← Meta-specific logic & gotchas
    ├── cross-platform.md              ← Attribution reconciliation
    ├── codex-review.md                ← Codex's review of AdLoop + improvements
    └── lessons-learned.md             ← Real incidents that shaped the rules
```

---

## Workflows

### Google Ads
| Code | Workflow | What It Does |
|------|----------|-------------|
| GA-1 | Performance Analysis | Pull data, cross-reference with Shopify, flag problems |
| GA-2 | Campaign Creation | Bid calculation, targeting validation, budget checks |
| GA-3 | Keyword Management | Match type safety, duplicate checks, negative keywords |
| GA-4 | Bidding Strategy Changes | Follow the ladder, check conversion volume |
| GA-5 | Shopping / PMax | Feed verification, PMax cannibalization check |
| GA-6 | Account Audit | Full audit: tracking → cannibalization → exclusions → wasted spend → QS |

### Meta Ads
| Code | Workflow | What It Does |
|------|----------|-------------|
| MA-1 | Performance Analysis | Shopify cross-reference, frequency checks, learning status |
| MA-2 | Campaign Creation | Learning phase math, funnel structure, creative format selection |
| MA-3 | Scaling | Profit Floor verification, 20/20 Rule, Revenue at Risk pitch |
| MA-4 | Creative Strategy | Format hierarchy, in-account proof over generic best practice |
| MA-5 | Troubleshooting | No delivery, spend but no purchases, learning phase stuck |
| MA-6 | Account Audit | Full audit incl. exclusion hygiene and DPA catalog health |
| MA-7 | Creative Sandbox | Test → graduate → scale. Isolated creative testing with kill/promote criteria |

### Cross-Platform
| Code | Workflow | What It Does |
|------|----------|-------------|
| XP-1 | Attribution Reconciliation | Three-way truth table: Platform vs GA4 vs Shopify + MER |
| XP-2 | Budget Allocation | Allocate to Shopify-verified efficiency |
| XP-3 | Risk Assessment | Before any change: what could go wrong, worst-case cost, rollback plan |
| XP-4 | Monthly Pacing | Mid-month run-rate check, prevent cash flow exhaustion |
| XP-5 | Emergency Red Alert | Auto-pause for tracking loss, 404, budget runaway (no approval needed) |
| XP-6 | Standardized Rollback | Exact 1:1 restoration from CHANGELOG, no tweaking |

---

## Key Concepts

### POAS > ROAS

ROAS is a vanity metric. A 3x ROAS with 60% COGS and 15% shipping means you're barely breaking even.

```
POAS = (Revenue - COGS - Shipping - Returns) ÷ Ad Spend
```

| POAS | Status | Action |
|------|--------|--------|
| > 1.2 | Scalable | Check if Protected. Scale +20%. |
| 1.0 - 1.2 | Stable | Monitor. Check creative fatigue. |
| < 1.0 | Critical | Losing money per sale. Cut or kill. |

If a campaign has high ROAS but low POAS → the problem is product margins, not the ad.

### Protected Campaigns

A campaign is **protected** if:
- Shopify-verified ROAS > 1.2× the `target_roas` in config.json
- Spending consistently (no 0-spend days in last 14 days)
- 10+ conversions in last 30 days

Protected campaigns cannot be paused, restructured, or budget-reduced without showing the daily revenue at risk.

### Three Truth Sources

| Source | What It Tells You | Trust Level |
|--------|-------------------|-------------|
| Ad Platform (Google/Meta) | Clicks, impressions, platform-attributed conversions | Medium — each over-claims 30-80% |
| GA4 | Sessions, on-site behavior | Medium — GDPR reduces counts |
| **Shopify/Backend** | **Actual orders, actual revenue, actual refunds** | **Highest — ground truth** |

### The Bidding Ladder (Google Ads)

```
Manual CPC (0 conversions) — start here always
    → Enhanced CPC (10+ conversions)
        → Maximize Conversions (15+/month)
            → Target CPA (30+/month, stable 2 weeks)
                → Target ROAS (50+/month, stable)
```

### Learning Phase Math (Meta Ads)

```
Min budget per ad set = (Cost Per Purchase × 50) ÷ 7
Max viable ad sets = Total daily budget ÷ Min per ad set
```

If budget supports 2 ad sets, build 2. Not 6.

### Creative Sandbox (Test → Graduate → Scale)

New creatives never go directly into protected campaigns. They live in a Sandbox campaign first:
- **72 hours + 3× AOV spend** before judgment
- **Kill:** Hook rate < 25%, CTR < 1%, or CPA > 1.5× average
- **Graduate:** CPA < target over 7 days, volume > 3× AOV, retention in top 25%
- **Promote:** Duplicate winner into scaling campaign, swap out lowest performer, monitor 48h

---

## Blocked Operations

| Operation | Why | Alternative |
|-----------|-----|-------------|
| Delete any ad entity | Data loss is permanent | Pause + rename ARCHIVE_ |
| Pause profitable campaign to restructure | Revenue loss, learning loss | Test alongside, scale winners |
| Broad Match + Manual CPC (Google) | #1 wasted spend cause | Phrase/Exact, or Smart Bidding first |
| MaxConv on 0-history + small budget | Algorithm sits idle | Manual CPC, follow the ladder |
| More Meta ad sets than budget supports | All stuck in learning | Calculate first, consolidate |
| Carousel for cold premium (no proof) | Proven 0 purchases | Video/UGC/static first |
| Budget increase > 40% in 7 days | Resets learning, spikes CPA | 20% per step, 72h between |
| Campaign without geo/language targeting | Global serving wastes budget | Always set targeting |
| Restructure + scale simultaneously | Can't diagnose failures | One change at a time |
| Inject untested creative into protected campaign | Resets learning phase | Use Creative Sandbox (MA-7) |
| Ads on out-of-stock products | Paying for clicks on 404/OOS | Inventory < 5 = auto-pause |

---

## Safety Limits Quick Reference

| Action | Limit | Reason |
|--------|-------|--------|
| Scaling | Max 20% per step, 40% per 7-day window | Avoids re-entering learning phase |
| New ad sets | (Total Budget ÷ CPP × 50) / 7 | Prevents starving ad sets of data |
| Broad Match | BLOCKED on Manual CPC | Prevents uncontrolled wasted spend |
| Protected status | ROAS > 1.2× client target | Prevents restructuring what works |
| Emergency pause | Auto-allowed for tracking loss / 404 / budget runaway | Protects client capital |
| Auto-approve | < 15% budget change, < 50 EUR risk, ROAS 20%+ above target | Removes operator bottleneck on safe changes |
| Creative testing | 72h + 3× AOV minimum before kill/graduate | Prevents premature decisions |
| Monthly pacing | Check on 10th + 20th, flag if > 110% of budget | Prevents cash flow exhaustion |
| Rollback | Exact 1:1 restore from CHANGELOG | No "tweaking" — restore known-good state |

---

## Setup

### For Claude Code / Cursor

1. Copy `rules/` to your project's `.claude/rules/` or `.cursor/rules/`
2. Copy `agent/cnd-ads-manager.md` to `.claude/agents/`
3. Copy `commands/` to `.claude/commands/`
4. Fill out `templates/ads-config.json` for your client

### Per-Client Configuration

```json
{
  "client": "your-client",
  "platforms": {
    "google": {
      "max_daily_budget": 200,
      "currency": "EUR",
      "account_id": "xxx-xxx-xxxx"
    },
    "meta": {
      "max_daily_budget": 300,
      "currency": "EUR",
      "account_id": "act_xxxxxxxxx",
      "pixel_id": "xxxxxxxxxxxxx",
      "catalog_id": "xxxxxxxxxxxxx"
    }
  },
  "targets": {
    "target_roas": 3.0,
    "target_poas": 1.2,
    "target_cpa": 45,
    "break_even_roas": 1.8,
    "aov": 120,
    "monthly_budget_cap": 10000
  },
  "unit_economics": {
    "cogs_pct": 40,
    "shipping_per_order": 8,
    "return_rate_pct": 5,
    "contribution_margin_pct": 45
  },
  "caps": {
    "daily_scaling_limit_pct": 20,
    "weekly_scaling_limit_pct": 40,
    "hours_between_scaling": 72
  },
  "exclusions": {
    "past_purchasers_days": 180,
    "web_visitors_days_for_cold": 30
  },
  "auto_approve": {
    "enabled": true,
    "max_budget_increment_pct": 15,
    "max_daily_risk_eur": 50,
    "min_roas_above_target_pct": 20
  },
  "seasonality": {
    "peak_months": ["november", "march"],
    "client_events": ["black_friday", "spring_sale"]
  },
  "protected_campaigns": [],
  "notes": ""
}
```

### Available Commands

| Command | What It Does |
|---------|-------------|
| `/ads-audit` | Full multi-platform audit with safety checks |
| `/ads-create-campaign` | Campaign creation through all 6 gates |
| `/ads-performance` | Cross-platform performance analysis with Shopify truth |
| `/ads-scale` | Safe scaling with profit floor verification |
| `/ads-report` | Monthly/quarterly performance report |
| `/ads-profit-audit` | Weekly POAS check — are we actually making money? |

---

## Differences from AdLoop

| Feature | AdLoop | This System |
|---------|--------|-------------|
| Platforms | Google Ads only | Google + Meta + TikTok |
| Safety enforcement | Python code guards | Agent rules (LLM-enforced, context-aware) |
| Truth source | Ads + GA4 | Ads + GA4 + **Shopify** |
| Profit metric | ROAS only | **POAS** (Revenue - COGS - Shipping) ÷ Spend |
| Protected campaigns | Not implemented | Dynamic: ROAS > 1.2× client target |
| dry_run validation | Fake (logs only) | Real API validate_only |
| Broad Match blocking | Warning only | Hard error, fail closed |
| Budget-to-structure math | Not implemented | Meta learning phase math enforced |
| Creative testing | Not implemented | Sandbox → Graduate → Scale pipeline |
| Scaling logic | Not implemented | 20/20 Rule + Profit Floor + Revenue at Risk pitch |
| Emergency response | Not implemented | XP-5 Red Alert — auto-pause without approval |
| Monthly pacing | Not implemented | Run-rate governor prevents cash flow exhaustion |
| Rollback | Not implemented | Exact 1:1 restore from CHANGELOG |
| Auto-approve | Not implemented | Threshold-based for low-risk changes |
| Exclusion hygiene | Not implemented | Purchaser + visitor exclusions audited |
| Inventory safety | Not implemented | Stock < 5 = auto-pause ads |
| Cannibalization | Not implemented | PMax vs Brand Search detection |
| Seasonality | Not implemented | Gate 0.5 context check |
| Audit trail | Thin (timestamp + payload) | Full (who, why, prior state, POAS, approval, risk) |
| Client context | Single account config | Per-client with economics, margins, seasonality |

---

## How It Was Built

1. Studied [AdLoop](https://github.com/kLOsk/adloop) — a safety-first MCP server for Google Ads
2. Used [Codex](https://openai.com) to review AdLoop's code — found 12 issues including fake dry runs, dead safety code, and warning-only blocking
3. Rebuilt from scratch for multi-platform agency use
4. Field-tested on real client accounts with real ad spend
5. Iterated based on code review feedback (Gemini contributed seasonality, POAS, creative graduation, and pacing logic)

Every rule was written after a real mistake cost real money. The [lessons-learned.md](docs/lessons-learned.md) file documents the specific incidents.

---

## Credits

- **[AdLoop](https://github.com/kLOsk/adloop)** by [@kLOsk](https://github.com/kLOsk) — the safety-first approach that inspired this system
- **Codex** — reviewed AdLoop's logic, found 12 improvements, built the Meta safety layer
- **Gemini** — contributed seasonality, POAS, creative graduation, pacing, and rollback logic
- **CN/D Agency** — field-tested every rule with real ad spend and real mistakes

---

## License

MIT

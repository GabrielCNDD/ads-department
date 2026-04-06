# CND Ads Department

**An AI-powered ads operations system with safety guardrails for multi-platform paid advertising.**

Built by [Connecting The Dots (CN/D)](https://cn-dots.com) — inspired by [AdLoop](https://github.com/kLOsk/adloop)'s safety-first approach to Google Ads, extended to cover Meta Ads, cross-platform attribution, and agency-scale operations.

---

## Why This Exists

Managing ad accounts with AI assistants is powerful but dangerous. Without guardrails, AI agents make expensive mistakes:

- Setting bids at 0.01 when the minimum competitive bid is 5.00
- Launching Maximize Conversions on campaigns with zero conversion history
- Rebuilding profitable accounts from scratch instead of scaling what works
- Creating 6 ad sets when the budget only supports 2 through learning phase
- Using carousel ads for cold premium audiences (proven 0 purchases)

**Every rule in this system exists because breaking it cost real money.**

This system enforces a 6-gate safety model where every ad account mutation must pass through validation before execution.

---

## The 6-Gate Safety Model

```
USER REQUEST
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
│  GATE 2: DATA PULL          │  Pull 30-90 day performance.
│  Cross-reference:           │  Identify protected campaigns.
│  Platform + GA4 + Shopify   │  Calculate cost of change.
└─────────────┬───────────────┘
              │ DATA READY
              ▼
┌─────────────────────────────┐
│  GATE 3: DECISION TREE      │  Follow platform-specific workflow.
│  ads-decision-trees.md      │  Pre-write validation.
│                             │  Calculate everything.
└─────────────┬───────────────┘
              │ DECISION MADE
              ▼
┌─────────────────────────────┐
│  GATE 4: DRAFT & PREVIEW    │  Create change plan.
│  Present to operator        │  Show: what, why, impact, risk,
│                             │  revenue at risk, reversibility.
└─────────────┬───────────────┘
              │ OPERATOR APPROVES
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
│   ├── ads-decision-trees.md          ← Gate 3: Step-by-step workflows
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
│   └── ads-report.md                  ← Monthly/quarterly report generation
│
├── templates/
│   ├── ads-config.json                ← Per-client config template
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

## Key Concepts

### Protected Campaigns

A campaign is **protected** if it meets ALL of:
- ROAS > 2x in the last 30 days
- Spending consistently (no 0-spend days in last 14 days)
- 10+ conversions in the last 30 days

Protected campaigns **cannot** be paused, budget-reduced, bidding-changed, or audience-narrowed without explicit operator approval showing the revenue at risk.

### Three Truth Sources

| Source | What It Tells You | Trust Level |
|--------|-------------------|-------------|
| Ad Platform (Google/Meta) | Clicks, impressions, platform-attributed conversions | Medium — each over-claims |
| GA4 | Sessions, on-site behavior, GA4-attributed conversions | Medium — GDPR reduces counts |
| Shopify/Backend | Actual orders, actual revenue | **High — ground truth** |

Every decision cross-references all three. Shopify wins when they disagree.

### The Bidding Ladder (Google Ads)

```
Manual CPC (0 conversions)
    → Enhanced CPC (10+ conversions)
        → Maximize Conversions (15+/month)
            → Target CPA (30+/month, stable 2 weeks)
                → Target ROAS (50+/month, stable)
```

Never skip steps. Never start at Maximize Conversions with zero history and < 1000/day budget.

### Learning Phase Math (Meta Ads)

```
Min budget per ad set = 50 × Cost Per Purchase ÷ 7 (daily)
Max viable ad sets = Total daily budget ÷ Min per ad set
```

If budget supports 2 ad sets, build 2. Not 6. Consolidation beats elegant starvation.

### Creative Format Hierarchy (Meta, Cold Premium)

```
1. Proof-led video / UGC
2. Strong single-image static
3. Authority / transformation creative
4. Collection (if catalog behavior proven)
5. Carousel (LAST — proven 0 purchases on cold premium)
```

---

## Blocked Operations

These are **hard blocks** — the system refuses to proceed:

| Operation | Why | Alternative |
|-----------|-----|-------------|
| Delete any ad entity | Data loss is permanent | Pause + rename ARCHIVE_ |
| Pause profitable campaign to restructure | Revenue loss, learning loss | Test alongside, scale winners |
| Broad Match + Manual CPC (Google) | #1 wasted spend cause | Use Phrase/Exact, or switch to Smart Bidding |
| MaxConv on 0-history + small budget | Algorithm sits idle, 0 impressions | Start Manual CPC, follow the ladder |
| More Meta ad sets than budget supports | All stuck in learning, none convert | Calculate first, consolidate |
| Carousel for cold premium (no proof) | Proven 0 purchases | Video/UGC/static first |
| Budget increase > 40% in one step | Resets learning, spikes CPA | 20% increments, 3-4 day intervals |
| Campaign without geo/language targeting | Global serving wastes budget | Always set targeting |
| Restructure + scale simultaneously | Too many variables, can't diagnose | One change at a time |

---

## Setup

### For Claude Code / Cursor

1. Copy `rules/` to your project's `.claude/rules/` or `.cursor/rules/`
2. Copy `agent/cnd-ads-manager.md` to `.claude/agents/`
3. Copy `commands/` to `.claude/commands/`
4. Create per-client `ads/config.json` from `templates/ads-config.json`

### Per-Client Configuration

```json
{
  "client": "example",
  "platforms": {
    "google": {
      "max_daily_budget": 200,
      "currency": "RON",
      "account_id": "xxx-xxx-xxxx"
    },
    "meta": {
      "max_daily_budget": 300,
      "currency": "RON",
      "account_id": "act_xxxxxxxxx"
    }
  },
  "targets": {
    "target_roas": 4.0,
    "target_cpa": 40,
    "aov": 375
  },
  "protected_campaigns": []
}
```

---

## Differences from AdLoop

| Feature | AdLoop | CND Ads Department |
|---------|--------|-------------------|
| Platforms | Google Ads only | Google + Meta + TikTok |
| Safety enforcement | Code (Python guards) | Agent rules (LLM-enforced, context-aware) |
| Truth source | Ads + GA4 | Ads + GA4 + **Shopify** |
| Protected campaigns | Not implemented | Core concept — ROAS > 2x = untouchable |
| dry_run validation | Fake (logs only, no API validate) | Real API validate_only |
| Broad Match blocking | Warning only | Hard error, fail closed |
| Budget-to-structure math | Not implemented | Meta learning phase math enforced |
| Creative format gating | Not implemented | Format hierarchy by funnel stage |
| Bidding strategy | MaxConv from day 1 | Bidding ladder (market-dependent) |
| Mutation decomposition | One-at-a-time guidance | Enforced: reject multi-lever changes |
| Winner preservation | Not implemented | Clone-first policy for profitable entities |
| Audit trail | Thin (timestamp + payload) | Full (who, why, prior state, approval, risk) |
| Client context | Single account config | Per-client with economics, margins, targets |

---

## Credits

- **AdLoop** by [@kLOsk](https://github.com/kLOsk) — the safety-first approach to Google Ads that inspired this system
- **Codex** — reviewed AdLoop's logic, found 12 improvements, built the Meta safety layer
- **CN/D Agency** — field-tested every rule with real ad spend and real mistakes

---

## License

MIT

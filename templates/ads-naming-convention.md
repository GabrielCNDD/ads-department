# CN/D Ads Department — Campaign Naming Convention

_Standard for ALL platforms. Every campaign, ad set, and ad follows this format._

## Campaign Level

```
CND_[Platform]_[Funnel]_[Type]_[Bid]_[Geo]_[Date]
```

> `CND` prefix on every campaign — client always sees who manages their account.

| Component | Values | Example |
|-----------|--------|---------|
| Platform | `G` (Google), `M` (Meta), `TT` (TikTok), `MS` (Microsoft), `LI` (LinkedIn) | `M` |
| Funnel | `COLD` (prospecting), `WARM` (consideration), `HOT` (retargeting), `BRAND` (brand defense) | `COLD` |
| Type | `SEARCH`, `SHOP` (Shopping), `PMAX`, `ASC` (Advantage+), `DPA` (Dynamic), `VIDEO`, `DISPLAY` | `ASC` |
| Bid | `LCOST` (Lowest Cost), `CCAP-X` (Cost Cap X), `TROAS-X` (Target ROAS X), `MCPC` (Manual CPC), `MCONV` (Max Conversions) | `LCOST` |
| Geo | `RO` (Romania), `BUC` (Bucharest), `EU`, `US` | `RO` |
| Date | `YYMM` launch date | `2603` |

**Examples:**
- `CND_M_COLD_ASC_LCOST_RO_2603` — Meta, Cold prospecting, Advantage+ Shopping, Lowest Cost, Romania, Mar 2026
- `CND_G_COLD_SEARCH_MCPC_RO_2603` — Google, Cold, Search, Manual CPC, Romania, Mar 2026
- `CND_M_HOT_DPA_CCAP-200_RO_2603` — Meta, Hot retargeting, Dynamic Product Ads, Cost Cap 200 RON
- `CND_G_COLD_SHOP_TROAS-10_RO_2603` — Google, Cold, Standard Shopping, Target ROAS 10x

## Ad Set / Ad Group Level

```
[Audience]_[Demo]_[Opt]_[Placement]
```

| Component | Values | Example |
|-----------|--------|---------|
| Audience | `BROAD`, `INT-X` (Interest), `LAL-X%` (Lookalike), `WV-Xd` (Website Visitors), `ATC-Xd` (Add to Cart), `ENG-Xd` (Engagement), `CUST` (Customer list) | `BROAD` |
| Demo | `M18-55` (Male 18-55), `ALL`, `F25-45` | `M25-55` |
| Opt | `PURCH` (Purchase), `ATC` (Add to Cart), `LC` (Link Clicks) | `PURCH` |
| Placement | `AUTO` (Advantage+), `FB` (Facebook only), `IG` (Instagram only), `FEED`, `REELS` | `AUTO` |

**Examples:**
- `BROAD_M25-55_PURCH_AUTO` — Broad targeting, Males 25-55, Optimize for Purchase, All placements
- `WV-30d_ALL_PURCH_AUTO` — Website visitors 30 days, Optimize Purchase
- `ATC-28d_ALL_PURCH_AUTO` — Cart abandoners 28 days
- `LAL-1%_M25-55_PURCH_AUTO` — 1% Lookalike, Males 25-55

## Ad Level

```
[Format]_[Hook]_[Version]
```

| Component | Values | Example |
|-----------|--------|---------|
| Format | `CARO` (Carousel), `VID-Xs` (Video length), `STAT` (Static), `COLL` (Collection), `UGC` | `CARO` |
| Hook | Brief description of creative angle | `multi-fitment` |
| Version | `v1`, `v2`, `v3` | `v1` |

**Examples:**
- `CARO_multi-fitment-BMW_v1`
- `VID-15s_montaj-jante_v1`
- `STAT_pret-livrare-gratuita_v1`
- `UGC_unboxing-client_v1`

## Google Ads Specific

### Ad Groups (Search)
```
[Match]_[Theme]
```
- `EX_jante-BMW-R19` — Exact match, jante BMW R19
- `PHR_jante-aliaj` — Phrase match, jante aliaj
- `BRD_jante-emag` — Broad match, competitor capture

### Shopping Product Groups
Use custom labels from feed:
- `custom_label_0`: Brand (BMW, Mercedes...)
- `custom_label_1`: Size (R18, R19...)
- `custom_label_2`: Price tier (sub-1000, 1000-1500, 1500+)

## Rules

1. **No spaces** — use underscores
2. **No Romanian characters** — use ASCII only (no ă, ș, ț)
3. **Date is launch date** — never change after creation
4. **Bid strategy in name** — always, so you can see it at a glance
5. **When duplicating** — update the date component
6. **Archive, don't delete** — pause + add `_ARCH` suffix to old campaigns

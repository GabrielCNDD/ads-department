# CN/D Ads Department — Bidding Logic

_Never guess bids. Calculate them from economics + industry data + product value._

## Three Inputs (All Required)

### 1. Client Economics
```
Max CPC = Target CPA × Expected CVR
```
- **AOV** — average order value from Shopify/Meta/GA4
- **Target CPA** — usually 15-25% of AOV for ecommerce (profitable after COGS)
- **Margins** — a 50% margin product can afford 2x the CPA of a 25% margin product

### 2. Product Price Reference
The actual selling price of the product matters. A 50 RON shampoo and a 700 RON K18 treatment should NOT have the same CPC.

```
Product-level Max CPC = Product Price × Margin % × Target Ad Spend % × CVR
```

Example: Kerastase sampon 140 RON, 40% margin, 20% of margin for ads, 2% CVR
→ 140 × 0.40 × 0.20 × 0.02 = **0.22 RON max CPC**

Example: K18 pro treatment 699 RON, 35% margin, 20% of margin for ads, 2% CVR
→ 699 × 0.35 × 0.20 × 0.02 = **0.98 RON max CPC**

This is why Shopping segmentation by price tier matters — cheap products need low bids, expensive products can afford more.

### 3. Industry Benchmarks (Romania)

| Vertical | Avg CPC Search | Avg CPC Shopping | Avg CPC Display | Avg CVR | Source |
|---|---|---|---|---|---|
| Beauty / Cosmetics | 0.80-1.80 RON | 0.50-1.50 RON | 0.15-0.60 RON | 1.5-3% | Google Ads Benchmarks 2025 |
| Ecommerce General | 0.60-1.50 RON | 0.40-1.20 RON | 0.10-0.50 RON | 1-2.5% | WordStream / SEMrush |
| Auto Parts | 1.00-2.50 RON | 0.80-2.00 RON | 0.20-0.80 RON | 1-2% | Industry avg |
| Fashion | 0.50-1.20 RON | 0.30-0.90 RON | 0.08-0.40 RON | 1.5-3% | Industry avg |
| Health / Supplements | 1.00-2.50 RON | 0.60-1.50 RON | 0.15-0.60 RON | 1-2% | Industry avg |
| Home / Garden | 0.60-1.50 RON | 0.40-1.00 RON | 0.10-0.40 RON | 1-2% | Industry avg |

Note: Romania CPCs are 50-70% lower than Western Europe. These are RO-specific ranges.

## How to Calculate

| Campaign Type | Expected CVR | Why |
|---|---|---|
| Brand Search | 3-8% | Highest intent — searching your brand |
| Generic Search | 1-3% | Intent but not brand-loyal |
| Shopping | 1-2% | Visual, product-level |
| Competitor Search | 0.5-1.5% | Stealing traffic, lower trust |
| Display Remarketing (cart abandoners) | 2-5% | Almost bought, warmest audience |
| Display Remarketing (product viewers) | 0.5-2% | Browsed but didn't add to cart |
| Display Remarketing (all visitors) | 0.2-1% | General awareness, cold-ish |
| Display Prospecting | 0.1-0.5% | Cold, lowest intent |
| PMax | 1-3% | Mixed, depends on signals |

4. **Calculate Max CPC per campaign type:**

### Example: AOV 375 RON, Target CPA 100 RON

| Campaign Type | CVR | Max CPC |
|---|---|---|
| Brand Search | 5% | 100 × 0.05 = **5.00 RON** |
| Generic Search | 2% | 100 × 0.02 = **2.00 RON** |
| Shopping | 1.5% | 100 × 0.015 = **1.50 RON** |
| Competitor Search | 1% | 100 × 0.01 = **1.00 RON** |
| Remarketing Cart Abandoners | 3% | 100 × 0.03 = **3.00 RON** |
| Remarketing Product Viewers | 1% | 100 × 0.02 = **1.20 RON** |
| Remarketing All Visitors | 0.5% | 100 × 0.005 = **0.50 RON** |

### Example: AOV 5,000 RON (high-ticket, e.g., Jante), Target CPA 250 RON

| Campaign Type | CVR | Max CPC |
|---|---|---|
| Brand Search | 5% | 250 × 0.05 = **12.50 RON** |
| Generic Search | 2% | 250 × 0.02 = **5.00 RON** |
| Shopping | 1.5% | 250 × 0.015 = **3.75 RON** |
| Competitor Search | 1% | 250 × 0.01 = **2.50 RON** |

## The Decision Process

1. **Calculate from client economics** (AOV, margin, CPA target)
2. **Cross-check with product price** (don't bid more than the product can justify)
3. **Cross-check with industry benchmarks** (if your calculated bid is 5x the industry average, something's wrong)
4. **Start at 60-70% of the calculated max** — room to increase if impression share is low
5. **Adjust from real data after 7 days**

## Rules

1. **Always calculate before setting bids.** Never use round numbers from gut feeling.
2. **Different products = different bids.** A Shopping campaign selling 50 RON shampoo and 700 RON treatments should segment by price tier with different bids.
3. **Cart abandoners are the most valuable audience.** Their CPC should be the highest in the account after brand search.
4. **If bids are too low, ads don't show.** Zero impressions = zero chance of conversion. A bid that's 20% too high wastes some money. A bid that's 50% too low wastes the entire campaign.
5. **Cross-reference all three inputs.** If economics say 5 RON but industry says 1.50 RON, investigate — maybe your CVR assumption is wrong.
6. **Adjust based on actual data after 7 days.** If CVR is higher than expected, you can bid higher. If lower, reduce.

## Smart Bidding Thresholds

| Strategy | Minimum Data Needed | When to Switch |
|---|---|---|
| Manual CPC | 0 conversions | Starting point for new campaigns |
| Enhanced CPC | 10+ conversions | First upgrade |
| Maximize Conversions | 15+ conversions/month | Enough signal for algorithm |
| Target CPA | 30+ conversions/month | Stable CPA for 2+ weeks |
| Target ROAS | 50+ conversions/month | Stable ROAS, enough volume |

Never skip steps. Manual CPC → eCPC → Max Conv → Target CPA/ROAS.

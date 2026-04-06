# Cross-Platform Attribution & Budget Allocation

_How to reconcile data across platforms and allocate budget based on truth._

---

## The Attribution Problem

Every ad platform claims credit for conversions. When a user:
1. Sees a Meta ad (impression)
2. Searches on Google and clicks a brand ad
3. Buys on the website

Both Meta and Google claim the conversion. The total "platform-reported conversions" is often 30-80% higher than actual Shopify orders.

---

## The Three Truth Sources

| Source | What It Measures | Trust for Revenue | Trust for Behavior |
|--------|-----------------|-------------------|-------------------|
| Google Ads | Clicks, impressions, Google-attributed conversions | Medium | Low |
| Meta Ads | Impressions, clicks, Meta-attributed conversions | Low | Low |
| GA4 | Sessions, on-site events, GA4-attributed conversions | Medium | High |
| **Shopify** | **Orders, revenue, refunds, actual payments** | **Highest** | Low |

**Rule:** Shopify is truth for revenue. GA4 is truth for on-site behavior. Platforms are truth for their own delivery metrics (clicks, impressions, CPM).

---

## Attribution Reconciliation Protocol (XP-1)

### Step 1: Pull platform data (last 30 days)
- Google: campaigns, spend, conversions, revenue
- Meta: campaigns, spend, purchases, revenue

### Step 2: Pull Shopify data (same period)
- Orders by attributed source
- Revenue by attributed source
- Refunds and returns

### Step 3: Pull GA4 data
- Sessions by source/medium
- Conversions by source/medium

### Step 4: Build truth table

| Metric | Google Says | Meta Says | GA4 Says | Shopify Says |
|--------|-----------|----------|----------|-------------|
| Conversions | 100 | 80 | 70 | 55 orders total |
| Revenue | 50,000 RON | 40,000 RON | 35,000 RON | 28,000 RON |

### Step 5: Calculate true ROAS

```
True Google ROAS = Shopify revenue from Google ÷ Google spend
True Meta ROAS = Shopify revenue from Meta ÷ Meta spend
Blended ROAS = Total Shopify revenue ÷ Total ad spend
```

### Step 6: Identify overlap
```
Platform-reported total = Google conversions + Meta conversions = 180
Actual total = Shopify orders = 55
Overlap = 180 - 55 = 125 double-counted conversions (69% inflation)
```

---

## GDPR Impact on Attribution

In the EU (including Romania), GDPR consent banners affect tracking:

| What | Affected by GDPR? | Impact |
|------|-------------------|--------|
| Google Ads clicks | No | All clicks counted |
| Meta Ads impressions | No | All impressions counted |
| GA4 sessions | **Yes** | Only consenting users tracked |
| GA4 conversions | **Yes** | Only consenting users tracked |
| Shopify orders | No | All orders counted |

**Normal EU ratio:** Google clicks : GA4 sessions = 2:1 to 5:1
**This is NOT broken tracking.** It's consent rejection.

**Only flag tracking as broken when:**
- GA4 shows 0 sessions for ALL sources (not just paid)
- Organic traffic also shows anomalies
- Shopify shows orders but GA4 shows zero events

---

## Budget Allocation Logic (XP-2)

### Principle: Allocate to Shopify-verified efficiency

1. Calculate true ROAS per platform (from XP-1)
2. The platform with higher Shopify-verified ROAS gets more budget
3. Respect minimum viable budget per platform (learning phase requirements)
4. Test new platforms with 10-15% of total budget

### Decision Tree

```
IF both platforms profitable (Shopify ROAS > 2x):
  → Scale the more efficient one first (20% increments)
  → Maintain minimum budget on the other

IF one platform profitable, one not:
  → Invest in the profitable platform
  → Diagnose the unprofitable one (tracking? targeting? creative?)
  → Don't increase budget on unprofitable platform

IF neither platform profitable:
  → STOP spending
  → Diagnose: tracking, landing page, product-market fit
  → Fix fundamentals before adding budget

IF new platform being tested:
  → Allocate 10-15% of total budget
  → Run for minimum 2 weeks (Meta) or 4 weeks (Google)
  → Compare Shopify-verified results before scaling
```

### Budget Distribution Template

```
Total monthly ad budget: X RON

Platform A (proven, 3.5x ROAS): 60-70%
Platform B (proven, 2.2x ROAS): 20-30%
Testing (new platform/campaign): 10-15%
```

---

## Monthly Review Checklist

1. Run XP-1 (attribution reconciliation)
2. Calculate Shopify-verified ROAS per platform
3. Compare to previous month — trending up or down?
4. Identify campaigns to protect (ROAS > 2x)
5. Identify campaigns to investigate (spending but not converting)
6. Run XP-2 (budget allocation) — rebalance if needed
7. Check: total ad spend as % of net revenue (target: 15-25% for ecommerce)
8. Generate report (/ads-report)

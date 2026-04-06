# Codex Review of AdLoop

_On 2026-04-06, we used OpenAI Codex to review AdLoop's Google Ads logic and find improvements. This is the full review._

---

## Summary

Codex found **12 issues** in AdLoop's implementation, ranging from fake dry runs to dead code to logic bugs. The orchestration rules (adloop.md) are stronger than the actual implementation — many safety promises exist only in documentation, not in code.

---

## Findings

### 1. Fake Dry Runs (CRITICAL)

`confirm_and_apply` returns `DRY_RUN_SUCCESS` after logging, but never calls Google Ads in `validate_only` mode. Invalid mutations look "safe" until the real apply.

**Fix:** Make dry runs hit the Ads API with `validate_only=true` and surface field-level errors before any real apply.

### 2. Dead Safety Code (HIGH)

`requires_double_confirmation` and `check_bid_increase` are defined in `guards.py` but never called anywhere. `ChangePlan.requires_double_confirm` stays `false` by default.

**Fix:** Wire these into the write path. Set `requires_double_confirm` during plan creation for removes and large budget increases.

### 3. Broad Match Blocking is Warnings, Not Errors (HIGH)

The rules say "refuse before drafting" but `_check_broad_match_safety` only adds warnings. Worse, lookup failure silently returns no warning (fail-open).

**Fix:** Hard-error on Broad + non-Smart Bidding. Fail closed on lookup failure.

### 4. update_campaign Logic Bug (MEDIUM)

Changing only `target_cpa` or `target_roas` is treated as "No changes specified" because those fields are omitted from `has_any_change` at `write.py#L712`.

**Fix:** Include `target_cpa` and `target_roas` in change detection.

### 5. Incomplete RSA Validation (MEDIUM)

`_validate_rsa` checks counts and lengths but never validates `path1`/`path2`, doesn't reject empty/duplicate assets, and has no diversity checks.

**Fix:** Enforce path length/charset, reject blank strings, flag duplicate headlines/descriptions, add minimum diversity checks.

### 6. Naive URL Validation (LOW)

`_validate_urls` uses HEAD/GET which false-fails on bot protection, geo-blocking, WAF. Can block valid landing pages while not proving the page is ad-safe.

**Fix:** Treat 401/403 as soft warnings, follow redirects, pair with domain allowlists.

### 7. GA4 Name-Match Dependency (MEDIUM)

`analyze_campaign_conversions` maps Ads to GA4 using exact `sessionCampaignName` equality and hardcodes paid as `source == "google"` + `medium == "cpc"`. Misses PMax, Shopping, naming drift.

**Fix:** Add canonical traffic classification and looser normalized matching.

### 8. Lossy Landing Page Matching (MEDIUM)

`landing_page_analysis` strips URLs to bare path, ignores hostname/query params. Misgroups Shopify locale prefixes, variant URLs, redirects. Averages bounce rate arithmetically instead of weighting.

**Fix:** Canonicalize host+path, retain selected query keys, compute weighted rates.

### 9. Attribution Check Doesn't Join Events to Paid (LOW)

Reads GA4 event totals globally but never joins to paid traffic specifically. "Event exists" is global, not paid-specific.

**Fix:** Query event-level paid attribution or label as heuristic.

### 10. No Shopify/Backend Truth Source (HIGH)

No order/gclid/refund reconciliation. Ecommerce "truth" is Ads + GA4 only — weak under EU consent, refunds, COD failures.

**Fix:** Add commerce-source abstraction with Shopify reconciliation.

### 11. Thin Audit Trail (LOW)

`log_mutation` records timestamp and payload only. No who-approved, no prior state, no validation output, no tamper-resistant sequencing.

**Fix:** Log actor, pre-change snapshot, validation result, correlation ID.

### 12. Rules Stronger Than Implementation (META)

"One change at a time," "double-check destructive operations," and "pre-write validation" are documented but not enforced in code.

**Fix:** Move critical rules from markdown guidance into executable guards.

---

## Strategy Assessment

### MaxConv-from-Day-1 is Wrong for Small Eastern European Markets

AdLoop recommends Maximize Conversions as the default for new campaigns. This is too absolute for:
- Sparse-volume markets (lower search volume in RO than DE/US)
- Fragmented geo/language pools
- GA4-only consent-limited conversion data
- Small budgets (< 1000/day)

**Better rule:** Use MaxConv only when expected conversion volume is sufficient and conversion imports are reliable. Otherwise start Manual CPC or TARGET_SPEND with CPC cap.

### Meta Ads Patterns Worth Copying

- **Hard spend guardrails:** Auto-warn when spend > 5x target CPA with zero conversions
- **Staged budget ramps:** Google should get the same 20% increment treatment as Meta
- **Backend conversions as truth:** Shopify, not GA4
- **Separate learning pools by country/language:** Small mixed EE geos shouldn't share one bucket
- **Creative fatigue heuristics:** RSA/PMax need freshness mindset, not just static checks

---

## Changes Made by Codex

Codex fixed 403 lines across 3 files in the AdLoop fork:
- `src/adloop/ads/write.py` — RSA validation, path checks, diversity, broad match hard-error
- `src/adloop/crossref.py` — paid traffic classification fixes
- `src/adloop/safety/preview.py` — double confirmation wiring

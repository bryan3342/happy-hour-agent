---
name: deal-ranking
description: Reference rubric for scoring happy hour deals on price, combo value, and deal quality. Used by the deal-evaluator subagent and any agent that needs the canonical scoring math.
user-invocable: false
---

# Deal Ranking Rubric

Authoritative scoring rules for happy hour candidates. The `deal-evaluator` subagent applies these — this file is the source of truth.

## Composite formula

```
composite = 0.45 * price + 0.30 * combo + 0.25 * deal_quality
```

All three axes are scored 0–10. Composite is rounded to one decimal.

## 1. Price (weight 0.45)

Use the **cheapest qualifying drink** in the venue's happy hour deals. "Qualifying" means a draft beer, glass of wine, well cocktail, or non-alcoholic equivalent — not a single shot of well liquor.

| Cheapest qualifying drink (USD) | Score |
| :------------------------------ | :---- |
| ≤ 4 | 10 |
| 5 | 9 |
| 6 | 8 |
| 7 | 7 |
| 8 | 6 |
| 9 | 5 |
| 10 | 4 |
| 11 | 3 |
| 12 | 2 |
| ≥ 13 | 1 |

If no drink price is documented but the venue clearly has a happy hour, score **5** and flag in notes.

## 2. Combo (weight 0.30)

A "combo" is an explicit bundle priced as one unit. A discount list (e.g. "all drafts $5, all apps $7") is **not** a combo.

| Situation | Score |
| :-------- | :---- |
| Drink + food combo ≤ $15 | 10 |
| Drink + food combo $16–$25 | 8 |
| Drink + food combo > $25 | 6 |
| No combo, but both drinks and food discounted separately | 5 |
| Drinks discounted only | 3 |
| Food discounted only | 2 |

## 3. Deal quality (weight 0.25)

Start at **5**. Apply each modifier that applies:

| Modifier | Adjustment |
| :------- | :--------- |
| `source_confirmed: true` (verified on venue's own site) | +2 |
| Deal runs ≥ 5 days/week | +1 |
| Time window ≥ 3 hours long | +1 |
| Any item ≥ 40% off menu price | +1 |
| `source_confirmed: false` and only one secondary source | −2 |
| Time window < 90 minutes | −1 |

Clamp the result to `[0, 10]`.

## Tiebreakers (composites within 0.2)

1. Longer happy hour window.
2. More days per week.
3. Lower cheapest-drink price.

## Drop rules (exclude entirely, do not rank)

- Address outside Manhattan / Williamsburg / Bushwick / Greenpoint.
- No verifiable deal — only a claim that a happy hour exists.
- Deal explicitly ended (past date or "no longer offered" in sources).

## Worked example

Candidate:
- Cheapest drink: $5 draft beer → price = **9**
- Combo: "burger + draft for $12" → combo = **10**
- Source confirmed on venue site (+2), runs Mon–Fri (+1), 4–7 PM = 3 hrs (+1), no ≥40% discount = +0 → deal_quality = **9** (clamped)

```
composite = 0.45 * 9 + 0.30 * 10 + 0.25 * 9 = 4.05 + 3.00 + 2.25 = 9.30
```

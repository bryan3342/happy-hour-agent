---
name: brunch-ranking
description: Reference rubric for scoring bottomless brunch deals on price, inclusion breadth, and deal quality. Used by the brunch-evaluator subagent and any agent that needs the canonical brunch scoring math.
user-invocable: false
---

# Brunch Ranking Rubric

Authoritative scoring rules for bottomless brunch candidates. The `brunch-evaluator` subagent applies these — this file is the source of truth.

## Composite formula

```
composite = 0.50 * price + 0.35 * inclusion + 0.15 * quality
```

All three axes are scored 0–10. Composite is rounded to one decimal.

## 1. Price (weight 0.50)

Use the **per-person cost** of the bottomless brunch package (drinks + whatever food the venue includes — see Inclusion axis for what "included" means).

| Price per person (USD) | Score |
| :--------------------- | :---- |
| ≤ 25 | 10 |
| 26–30 | 9 |
| 31–35 | 8 |
| 36–40 | 7 |
| 41–45 | 6 |
| 46–50 | 5 |
| 51–60 | 4 |
| 61–75 | 3 |
| 76–90 | 2 |
| > 90 | 1 |

If the venue clearly offers bottomless brunch but no price is documented, score **5** and flag in notes.

## 2. Inclusion (weight 0.35)

Average of three sub-scores (each 0–10), rounded to nearest integer.

| Sub-axis | 10 | 8 | 6 | 4 | 2 |
| :------- | :- | :- | :- | :- | :- |
| **Drinks variety** | Full bar (any cocktail) | Full cocktail menu | Mimosas + Bloody Marys + sangria/other | Mimosas + Bloody Marys | Mimosas only |
| **Food coverage** | Entrée + appetizer/side | Entrée only | Brunch-menu credit ($X off) | Small bites / pastries | No food |
| **Duration** | Unlimited or ≥ 3 hrs | 2 hrs | 90 min | 60 min | < 60 min |

## 3. Quality (weight 0.15)

Start at **5**. Apply each modifier that applies:

| Modifier | Adjustment |
| :------- | :--------- |
| `source_confirmed: true` (verified on venue's own site) | +2 |
| Offered both Saturday AND Sunday | +1 |
| Walk-in accepted (no mandatory reservation) | +1 |
| No hidden gratuity floor disclosed (e.g. auto-20% for parties) | +1 |
| `source_confirmed: false` and only one secondary source | −2 |
| Known operational friction (mandatory time cap from seating, $X tip-in surcharge) | −1 |

Clamp the result to `[0, 10]`.

## Tiebreakers (composites within 0.2)

1. Wider drinks variety wins.
2. Longer duration wins.
3. Lower price wins.

## Drop rules (exclude entirely, do not rank)

- Address outside Manhattan / Williamsburg / Bushwick / Greenpoint.
- Not actually bottomless — a discounted prix-fixe brunch with a fixed drink count does not qualify.
- Deal explicitly ended (past date or "no longer offered" in sources).

## Worked example

Candidate: $35 per person, bottomless mimosas + Bloody Marys, brunch entrée included, 2-hr window, runs Sat + Sun, source-confirmed on venue site.

- Price: $35 → **8**
- Inclusion sub-scores: drinks 4 (mimosa+bloody) + food 8 (entrée) + duration 8 (2 hrs) = avg 6.67 → **7**
- Quality: 5 + 2 (confirmed) + 1 (Sat+Sun) = **8**

```
composite = 0.50 * 8 + 0.35 * 7 + 0.15 * 8 = 4.00 + 2.45 + 1.20 = 7.65 → 7.7
```

---
name: result-formatter
description: Reference template for rendering the final happy hour results table. Defines column order, formatting rules, and the required ranking-criteria footnote.
user-invocable: false
---

# Result Formatter — Reference

Single source of truth for the final output shape. Any agent producing happy hour results must follow this.

## Required structure

```markdown
# Happy Hours — <area>

| Bar Name | Location | Happy Hour Time Frame | Highlighted Deal |
| :------- | :------- | :-------------------- | :--------------- |
| <name>   | <neighborhood> — <street or cross-streets> | <days>, <time window> | <highlighted_deal> |

*Ranked by price (45%), combo value (30%), and deal quality (25%). Deals verified against venue sites where possible. Data gathered: <ISO date>.*
```

## Column rules

### Bar Name
- Plain text, no link, no emoji.
- Use the venue's canonical name (what's on their website), not the listicle's spelling.

### Location
- Format: `<Neighborhood> — <Street or cross-streets>`.
- Examples: `Williamsburg — Bedford Ave & N 7th St`, `East Village — 2nd Ave (10th–11th)`.
- Don't include full street address or ZIP. Keep it scannable.

### Happy Hour Time Frame
- Format: `<days>, <time window>`.
- Examples:
  - `Mon–Fri, 4:00–7:00 PM`
  - `Daily, 5:00 PM – close`
  - `Tue–Sat, 5:00–8:00 PM (also Sun brunch)`
- Use en-dashes (`–`) between times and day ranges.

### Highlighted Deal
- One concise string. ≤ 70 chars where possible.
- Lead with the most striking item (cheapest drink or biggest combo).
- Include the price.
- Examples:
  - `$1 oysters + $5 drafts`
  - `Burger + draft beer combo $12`
  - `$6 wine, $7 cocktails, half-price apps`
- Append ` (unconfirmed)` if the venue did not confirm the deal on its own site.

## Row order

- Top to bottom by composite score, descending.
- Show **top 10** by default. Show more only if the user asked.
- If two rows tie on composite within 0.2, use the rubric tiebreakers (longer window → more days → lower price).

## Footnote (required)

Always include this line below the table:

> *Ranked by price (45%), combo value (30%), and deal quality (25%). Deals verified against venue sites where possible. Data gathered: YYYY-MM-DD.*

Use the actual ISO date the scout completed gathering (which it appends to its response).

## What not to do

- Don't add extra columns (no "Rating", no "Score", no "Distance").
- Don't include images, links, or per-row source citations in the table itself. If the user asks for sources, list them below the footnote.
- Don't truncate the Highlighted Deal mid-word with `…` — rewrite shorter instead.
- Don't show JSON, intermediate scores, or raw scout output to the user. Only the final table + footnote.

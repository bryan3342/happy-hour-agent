---
name: find-happy-hours
description: Finds and ranks the best-priced happy hour deals at bars in Manhattan or Brooklyn (Williamsburg, Bushwick, Greenpoint). Use when the user asks for happy hours, cheap drinks, drink specials, after-work specials, or bar deals in NYC. Returns a Markdown table ranked by price, combo value, and deal quality.
when_to_use: User says "find happy hours", "cheap drinks", "happy hour in <neighborhood>", "where can I drink cheap tonight", "best deals near <NYC place>", or invokes /find-happy-hours.
argument-hint: [neighborhood] [day] [preferences]
allowed-tools: WebSearch, WebFetch, Read
---

# Find Happy Hours

Run the full happy-hour-finder pipeline and return a ranked Markdown table.

## Today's request

ARGUMENTS: $ARGUMENTS

## Steps

1. **Parse the request.** Extract from `$ARGUMENTS` (or the surrounding conversation) any of:
   - Neighborhood(s) — e.g. "Williamsburg", "East Village", "Bushwick".
   - Day of week — defaults to today.
   - Time window — defaults to "right now" (use the current local time).
   - Preferences — food included, wine vs cocktail vs beer, party size.

   If the user names a neighborhood outside Manhattan or Williamsburg/Bushwick/Greenpoint, stop and reply: "I cover Manhattan plus three Brooklyn neighborhoods — Williamsburg, Bushwick, and Greenpoint. Want me to search one of those instead?"

2. **Scout candidates.** Delegate to the `happy-hour-scout` subagent. Give it the parsed parameters. Wait for the JSON candidate list.

3. **Validate locations.** Pass the candidate list to the `location-curator` subagent. It will drop out-of-scope addresses and normalize neighborhood labels.

4. **Score and rank.** Pass the curated list to the `deal-evaluator` subagent. It applies the rubric from [deal-ranking](../deal-ranking/SKILL.md) and returns a ranked list with `composite` scores and `highlighted_deal` strings.

5. **Format the table.** Use the template from [result-formatter](../result-formatter/SKILL.md). Show the top 10 candidates unless the user asked for more.

## Required output shape

```markdown
# Happy Hours — <neighborhood(s) or "Manhattan + WB/Bushwick/Greenpoint">

| Bar Name | Location | Happy Hour Time Frame | Highlighted Deal |
| :------- | :------- | :-------------------- | :--------------- |
| ...      | ...      | ...                   | ...              |

*Ranked by price (45%), combo value (30%), and deal quality (25%). Deals verified against venue sites where possible — see notes for sources. Data gathered: <ISO date>.*
```

## Important rules

- **Don't fabricate.** Every row must trace back to a source the scout cited. If the scout couldn't verify a deal, either omit it or mark `(unconfirmed)` in the Highlighted Deal column.
- **Always include the freshness date.** Web-sourced deals go stale fast.
- **Don't show the JSON.** The user sees only the final Markdown table + the one-line ranking note.
- **One table per response.** If the user asked about multiple neighborhoods, include them all in one table with the neighborhood named in the Location column.
- **Batching discipline.** Never pass more than 3 raw website dumps to `deal-evaluator` at once. The scout's structured JSON is the only handoff format — if you ever have raw page text, summarize it into JSON or a bulleted fragment first.

## Reference material

- Geography rules and neighborhood landmarks: [nyc-bar-knowledge](../nyc-bar-knowledge/SKILL.md)
- Full scoring rubric: [deal-ranking](../deal-ranking/SKILL.md)
- Output template: [result-formatter](../result-formatter/SKILL.md)

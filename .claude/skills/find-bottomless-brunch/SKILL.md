---
name: find-bottomless-brunch
description: Finds and ranks the best bottomless brunch deals at restaurants in Manhattan or Brooklyn (Williamsburg, Bushwick, Greenpoint). Use when the user asks for bottomless brunch, boozy brunch, mimosa specials, or weekend brunch deals in NYC. Returns a Markdown table ranked by per-person price, inclusion breadth, and deal quality.
when_to_use: User says "find bottomless brunch", "boozy brunch", "mimosa specials", "bottomless mimosas in <neighborhood>", "best brunch deals near <NYC place>", or invokes /find-bottomless-brunch.
argument-hint: [neighborhood] [day: Sat|Sun|both] [preferences]
allowed-tools: WebSearch, WebFetch, Read
---

# Find Bottomless Brunch

Run the full bottomless-brunch-finder pipeline and return a ranked Markdown table.

## Today's request

ARGUMENTS: $ARGUMENTS

## Steps

1. **Parse the request.** Extract from `$ARGUMENTS` (or the surrounding conversation) any of:
   - Neighborhood(s) — e.g. "Williamsburg", "West Village", "Bushwick".
   - Day — `Sat`, `Sun`, or both. Default is **both** if unspecified.
   - Preferences — drinks variety (mimosas-only vs. full bar), food expectations, party size.

   If the user names a neighborhood outside Manhattan or Williamsburg/Bushwick/Greenpoint, stop and reply: "I cover Manhattan plus three Brooklyn neighborhoods — Williamsburg, Bushwick, and Greenpoint. Want me to search one of those instead?"

2. **Scout candidates.** Delegate to the `brunch-scout` subagent. Give it the parsed parameters. Wait for the JSON candidate list.

3. **Validate locations.** Pass the candidate list to the `location-curator` subagent. It will drop out-of-scope addresses and normalize neighborhood labels.

4. **Score and rank.** Pass the curated list to the `brunch-evaluator` subagent. It applies the rubric from [brunch-ranking](../brunch-ranking/SKILL.md) and returns a ranked list with `composite` scores and `highlighted_deal` strings.

5. **Format the table.** Use the template from [result-formatter](../result-formatter/SKILL.md). Show the top 10 candidates unless the user asked for more.

## Required output shape

```markdown
# Bottomless Brunch — <neighborhood(s) or "Manhattan + WB/Bushwick/Greenpoint">

| Restaurant | Location | Brunch Time Frame | Highlighted Deal |
| :--------- | :------- | :---------------- | :--------------- |
| ...        | ...      | ...               | ...              |

*Ranked by price (50%), inclusion (35%), and deal quality (15%). Deals verified against venue sites where possible. Data gathered: <ISO date>.*
```

The middle column header is **Brunch Time Frame** (vs. happy-hour's "Happy Hour Time Frame") — same shape, brunch-appropriate label.

## Important rules

- **Don't fabricate.** Every row must trace back to a source the scout cited. If the scout couldn't verify a deal, either omit it or mark `(unconfirmed)` in the Highlighted Deal column.
- **Always include the freshness date.** Web-sourced deals go stale fast.
- **Don't show the JSON.** The user sees only the final Markdown table + the one-line ranking note.
- **One table per response.** If the user asked about multiple neighborhoods, include them all in one table with the neighborhood named in the Location column.
- **Batching discipline.** Never pass more than 3 raw website dumps to `brunch-evaluator` at once. The scout's structured JSON is the only handoff format — if you ever have raw page text, summarize it into JSON or a bulleted fragment first.
- **Bottomless-only.** Drop prix-fixe brunches with a fixed drink count (e.g. "$45 brunch with 2 mimosas"). They don't belong in this table.

## Reference material

- Geography rules and neighborhood landmarks: [nyc-bar-knowledge](../nyc-bar-knowledge/SKILL.md)
- Full scoring rubric: [brunch-ranking](../brunch-ranking/SKILL.md)
- Output template: [result-formatter](../result-formatter/SKILL.md)

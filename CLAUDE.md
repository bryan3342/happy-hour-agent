# Happy Hour Agent — Project Conventions

This repo is a Claude Code agent system that finds the best-priced NYC happy hour deals and the best bottomless brunch deals. See `prd.md` for the full product spec.

## Scope

- **Geography**: Manhattan (all neighborhoods), plus Brooklyn — Williamsburg, Bushwick, Greenpoint. Nothing else.
- **Output**: Always a 4-column Markdown table. Happy-hour flow uses `Bar Name | Location | Happy Hour Time Frame | Highlighted Deal`; brunch flow uses `Restaurant | Location | Brunch Time Frame | Highlighted Deal`. Followed by a one-line note on ranking criteria and data freshness.
- **Ranking**: Happy-hour deals use `.claude/skills/deal-ranking/SKILL.md` (price 45 / combo 30 / quality 25). Bottomless brunch deals use `.claude/skills/brunch-ranking/SKILL.md` (price 50 / inclusion 35 / quality 15). Don't invent a new rubric mid-response.

## How the pieces fit

Two parallel pipelines sharing geography and table formatting:

- **`/find-happy-hours`** (skill in `.claude/skills/find-happy-hours/`) → `happy-hour-scout` subagent → `location-curator` → `deal-evaluator`.
- **`/find-bottomless-brunch`** (skill in `.claude/skills/find-bottomless-brunch/`) → `brunch-scout` subagent → `location-curator` → `brunch-evaluator`. Brunch defaults to Sat + Sun and drops any deal that isn't truly bottomless.
- `location-curator` and `result-formatter` are shared between both flows. `nyc-bar-knowledge` is the shared geography reference.

## Data path

`WebSearch` + `WebFetch` are the only data path. Do not introduce MCP servers or third-party API calls — the agent must operate solely on web data fetched live per request.

## Working on this repo

- Read `prd.md` before changing any agent or skill.
- Keep SKILL.md files under ~500 lines. Push reference data into supporting files in the skill's own directory.
- Subagents are read-only by design: do not grant `Write` or `Edit` tools to any subagent in `.claude/agents/`.
- Live data only — do not commit cached results. Every response reflects the request-time web fetch.

## Out-of-scope requests

If a user asks for happy hours outside the supported geography, decline politely and name the supported areas. Don't try to answer from stale knowledge.

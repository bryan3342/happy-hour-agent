# Happy Hour Agent — Project Conventions

This repo is a Claude Code agent system that finds the best-priced NYC happy hour deals. See `prd.md` for the full product spec.

## Scope

- **Geography**: Manhattan (all neighborhoods), plus Brooklyn — Williamsburg, Bushwick, Greenpoint. Nothing else.
- **Output**: Always a 4-column Markdown table: `Bar Name | Location | Happy Hour Time Frame | Highlighted Deal`. Followed by a one-line note on ranking criteria and data freshness.
- **Ranking**: Use the rubric in `.claude/skills/deal-ranking/SKILL.md`. Don't invent a new one mid-response.

## How the pieces fit

- `/find-happy-hours` (skill in `.claude/skills/find-happy-hours/`) is the user entry point.
- It delegates research to the `happy-hour-scout` subagent and scoring to `deal-evaluator`.
- `location-curator` verifies addresses fall inside the supported geography.
- `result-formatter` is reference-only — it defines the table shape.

## Data path

`WebSearch` + `WebFetch` are the only data path. Do not introduce MCP servers or third-party API calls — the agent must operate solely on web data fetched live per request.

## Working on this repo

- Read `prd.md` before changing any agent or skill.
- Keep SKILL.md files under ~500 lines. Push reference data into supporting files in the skill's own directory.
- Subagents are read-only by design: do not grant `Write` or `Edit` tools to any subagent in `.claude/agents/`.
- Live data only — do not commit cached results. Every response reflects the request-time web fetch.

## Out-of-scope requests

If a user asks for happy hours outside the supported geography, decline politely and name the supported areas. Don't try to answer from stale knowledge.

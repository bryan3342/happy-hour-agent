# Happy Hour Agent — Product Requirements Document

## 1. Overview

The **Happy Hour Agent** is a Claude Code assistant that finds, ranks, and returns the best-priced happy hour deals at bars across Manhattan and select Brooklyn neighborhoods (Williamsburg, Bushwick, Greenpoint). It is delivered as a project-scoped collection of Claude Code subagents and skills, using `WebSearch` and `WebFetch` as its only data path — no MCP servers, no third-party APIs.

Users invoke the agent in natural language ("find me cheap happy hours in Bushwick tonight") or with a slash command (`/find-happy-hours`). The agent searches the web, evaluates each candidate against a scoring rubric, and returns a ranked Markdown table.

## 2. Goals

1. Surface the **best-priced** happy hour deals in a defined NYC geography.
2. Evaluate each candidate consistently across three dimensions: **price**, **combo value**, and **deal quality**.
3. Present results in a uniform table: `Name | Location | Happy Hour Time Frame | Highlighted Deal`.
4. Keep the system extensible — neighborhoods, scoring weights, and data sources can be swapped without rewriting the agent.

## 3. Non-goals

- Real-time table reservations or bookings.
- Coverage outside Manhattan and the three named Brooklyn neighborhoods (Williamsburg, Bushwick, Greenpoint).
- Recommending bars based on ambiance, music, or non-pricing factors (these may be noted, but ranking is price/value driven).
- User accounts, history, or personalization across sessions.

## 4. Target users

- **NYC locals & visitors** looking for cheap drinks/food after work.
- **Group planners** who need a venue that hits a price target for several people.
- **Budget-conscious diners** comparing deals across neighborhoods.

## 5. User experience

### Invocation modes

| Mode | Example | Behavior |
| :--- | :--- | :--- |
| Slash command | `/find-happy-hours Williamsburg Thursday` | Runs the full pipeline with explicit args. |
| Natural language | "What's a cheap happy hour near Union Square right now?" | Claude auto-loads the skill based on description match. |
| Sub-question | "Which of these has the best wine deal?" | Follow-up filtered from the prior result set. |

### Output format (required)

A Markdown table with the following columns, ranked best-to-worst by composite score:

```markdown
| Bar Name | Location (Neighborhood / Address) | Happy Hour Time Frame | Highlighted Deal |
| :------- | :------------------------------- | :-------------------- | :--------------- |
| ...      | ...                              | ...                   | ...              |
```

After the table, the agent includes a short paragraph explaining ranking criteria and any caveats (e.g. "deals confirmed against venue websites as of <date>").

## 6. Scoring rubric

Each candidate bar is scored 0–10 on three axes; the final rank uses a weighted sum.

| Axis | Weight | What it measures |
| :--- | :----- | :--------------- |
| **Price** | 0.45 | Absolute cost of cheapest qualifying drink/food. Lower price → higher score. |
| **Combo** | 0.30 | Whether the deal bundles food + drink, or stacks multiple items. More inclusive bundles → higher score. |
| **Deal quality** | 0.25 | Strength of the discount vs. menu price, exclusivity, hours covered, day coverage. |

Composite score = `0.45 * price + 0.30 * combo + 0.25 * deal_quality`.

Ties are broken by (1) longer happy hour window, then (2) more days per week the deal runs.

The full rubric lives in `.claude/skills/deal-ranking/SKILL.md`.

## 7. Geographic scope

| Borough | Areas covered |
| :------ | :------------ |
| Manhattan | Entire borough — all neighborhoods from FiDi to Inwood. |
| Brooklyn | Williamsburg, Bushwick, Greenpoint (only). |

The neighborhood reference data and bar landmarks live in `.claude/skills/nyc-bar-knowledge/SKILL.md`. Out-of-scope requests should be politely declined with a note about coverage.

## 8. Architecture

```
happy-hour-agent/
├── prd.md                              # this document
├── CLAUDE.md                           # project conventions for Claude Code
└── .claude/
    ├── settings.json                   # permissions
    ├── agents/
    │   ├── happy-hour-scout.md         # gathers candidate bars + raw deal data
    │   ├── deal-evaluator.md           # applies the scoring rubric
    │   └── location-curator.md         # validates neighborhood + address data
    └── skills/
        ├── find-happy-hours/SKILL.md   # /find-happy-hours user entry point
        ├── deal-ranking/SKILL.md       # scoring rubric (reference)
        ├── nyc-bar-knowledge/SKILL.md  # neighborhood + bar reference data
        └── result-formatter/SKILL.md   # table output template
```

### Pipeline

1. **Intake** — `/find-happy-hours` (or natural-language match) parses the request: neighborhood(s), day/time, party size, food preference.
2. **Scout** — `happy-hour-scout` subagent uses `WebSearch` and `WebFetch` to pull current happy hour data from bar websites and aggregator sites.
3. **Curate** — `location-curator` subagent verifies addresses fall inside the supported geography using the static neighborhood reference in `nyc-bar-knowledge`.
4. **Evaluate** — `deal-evaluator` subagent applies the scoring rubric from `deal-ranking` to each candidate.
5. **Format** — `result-formatter` skill renders the final Markdown table.

## 9. Data sources

| Source | Purpose | Access |
| :--- | :--- | :--- |
| Venue websites | Source-of-truth for current deals | `WebFetch` |
| Eater NY, Time Out NY, Thrillist | Curated lists | `WebSearch` + `WebFetch` |
| Reddit (r/AskNYC, r/Brooklyn) | Recent first-hand reports | `WebSearch` |

All data is fetched live per request — there is no stored database, no MCP server, and no third-party API. The agent must note the freshness date in its response.

## 10. Tooling per subagent

| Subagent | Tools granted | Model |
| :------- | :------------ | :---- |
| `happy-hour-scout` | `WebSearch`, `WebFetch`, `Read`, `Grep`, `Glob` | `sonnet` |
| `deal-evaluator` | `Read`, `Grep`, `Glob` | `sonnet` |
| `location-curator` | `WebFetch`, `Read`, `Grep`, `Glob` | `haiku` |

Subagents inherit no write tools — they cannot modify the project.

## 11. Success metrics

- **Coverage**: ≥ 5 ranked candidates returned for any in-scope neighborhood query.
- **Freshness**: ≥ 80% of returned deals confirmable against the venue's own site at time of response.
- **Format compliance**: 100% of responses include the required four-column table.
- **Geographic accuracy**: 0 results outside the supported geography.

## 12. Out of scope (v1)

- Caching past results.
- Push notifications when a new deal appears.
- A web UI — the agent runs inside Claude Code only.
- Multi-language output.

## 13. Open questions

- Should the agent prefer venues with confirmable deals on official sites over aggregator-only listings? (Default: yes, by adding a freshness penalty to `deal_quality`.)
- How should the agent handle deals that vary by day? (Default: filter to the requested day; if no day given, surface the deal valid *now*.)
- Should non-alcoholic happy hours (coffee, NA cocktails) be included? (Default: yes when the user's query doesn't specify alcohol.)

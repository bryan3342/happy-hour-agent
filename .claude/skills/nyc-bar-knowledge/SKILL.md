---
name: nyc-bar-knowledge
description: Reference data for NYC happy hour geography — Manhattan neighborhoods, plus Williamsburg/Bushwick/Greenpoint in Brooklyn. Includes ZIP codes, landmark streets, and known happy-hour-heavy corridors. Used by happy-hour-scout and location-curator to define and verify scope.
user-invocable: false
---

# NYC Bar Geography — Reference

Static reference used by `happy-hour-scout` to pick search areas and `location-curator` to verify in-scope addresses. No happy hour deals are listed here — those are always fetched live.

## Supported scope

### Manhattan (all neighborhoods in scope)

| Neighborhood | Primary ZIPs | Notable happy-hour corridors |
| :----------- | :----------- | :--------------------------- |
| Financial District | 10004, 10005, 10006, 10038 | Stone St, Pearl St |
| Tribeca | 10007, 10013 | Greenwich St, Hudson St |
| SoHo | 10012, 10013 | Spring St, W Broadway |
| Lower East Side | 10002 | Ludlow St, Orchard St, Rivington St |
| Chinatown | 10002, 10013 | Doyers St, Mott St |
| East Village | 10003, 10009 | 1st Ave, Ave A, St Marks Pl |
| West Village | 10014 | Bleecker St, Hudson St, 7th Ave S |
| Greenwich Village | 10003, 10011, 10012 | MacDougal St, University Pl |
| Chelsea | 10001, 10011 | 8th Ave, 9th Ave |
| Flatiron / Union Sq | 10003, 10010 | Park Ave S, Broadway |
| Murray Hill | 10016 | 3rd Ave (30s), Lexington |
| Midtown East | 10017, 10022 | 2nd Ave, 3rd Ave (40s–50s) |
| Midtown West / Hell's Kitchen | 10018, 10019, 10036 | 9th Ave (40s–50s), 10th Ave |
| Upper East Side | 10021, 10028, 10065, 10075 | 2nd Ave (70s–80s), 3rd Ave |
| Upper West Side | 10023, 10024, 10025 | Amsterdam Ave, Columbus Ave |
| Harlem | 10026, 10027, 10030, 10031, 10037, 10039 | Frederick Douglass Blvd, Lenox Ave |
| Washington Heights | 10032, 10033, 10040 | Broadway, St Nicholas Ave |
| Inwood | 10034 | Broadway (above Dyckman) |

### Brooklyn (only these three)

| Neighborhood | Primary ZIPs | Notable corridors |
| :----------- | :----------- | :---------------- |
| Williamsburg | 11211, 11249 | Bedford Ave, Wythe Ave, Berry St, N 6th St |
| Bushwick | 11206, 11207, 11221, 11237 | Knickerbocker Ave, Wyckoff Ave, Troutman St, Myrtle-Broadway |
| Greenpoint | 11222 | Manhattan Ave, Franklin St, Nassau Ave, Greenpoint Ave |

## Out of scope (drop, do not rank)

- All of Queens, the Bronx, Staten Island.
- All other Brooklyn neighborhoods — including DUMBO, Brooklyn Heights, Cobble Hill, Carroll Gardens, Park Slope, Prospect Heights, Crown Heights, Bed-Stuy, Fort Greene, Gowanus, Red Hook, Sunset Park, Bay Ridge, Coney Island. These have great bars; they're just not in this agent's coverage.
- Anything outside NYC.

## Listicle sources that work well per area

- **Manhattan downtown / LES / EV**: Eater NY, Time Out NY, Infatuation.
- **Midtown**: Time Out NY, Thrillist (after-work focused lists).
- **Uptown / Harlem / UWS / UES**: Eater NY, Secret NYC.
- **Williamsburg / Greenpoint**: Greenpointers, Bedford + Bowery, Brokelyn.
- **Bushwick**: Bushwick Daily, Bedford + Bowery.
- **Cross-neighborhood freshness**: r/AskNYC, r/Brooklyn — most useful for spotting deals that have *ended* since a listicle was published.

## Address heuristics (when no ZIP is given)

- Bedford Ave + N/S numbered streets → Williamsburg.
- Manhattan Ave (Brooklyn) + Franklin St / Nassau Ave → Greenpoint.
- Knickerbocker / Wyckoff / Troutman → Bushwick.
- Named avenues 1st–12th, Lex, Park, Madison, Broadway with "NY, NY" → Manhattan.

When the address is ambiguous, the venue's own website almost always names the neighborhood on its contact page.

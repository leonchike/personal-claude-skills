---
name: flight-sim-search
description: >
  Search and present real-world flight data for flight simulation enthusiasts using the
  Flight Sim Search MCP server. Use this skill whenever the user mentions flights, routes,
  airlines, aircraft types, flight planning, SimBrief, FlightRadar24, flight durations,
  ICAO codes, or anything related to finding flights for simulation. Also use when the user
  asks to search, filter, compare, or explore flight options — even casually, like "find me
  a cool long-haul" or "what does Turkish fly on the A350". Covers search strategy,
  response formatting, URL construction, and MSFS 2024 addon management.
metadata:
  author: user
  version: 1.0.0
  mcp-server: flight-sim-search
---

# Flight Sim Search Skill

Guide Claude to use the Flight Sim Search MCP server effectively — searching flights,
formatting results, constructing URLs, and managing MSFS 2024 addons.

## Core Principles

1. **Maximize options** — When a user asks for flights, default to providing many results
   (50–60 in compact mode) unless they ask for fewer. Offer related alternatives (similar
   routes, alternate aircraft, nearby durations).

2. **Smart response sizing** — Match the response mode to the result count:

   | Results   | Mode                      | Key settings                          |
   |-----------|---------------------------|---------------------------------------|
   | 1–50      | Standard + URLs           | `includeUrls: true`                   |
   | 50–100    | Standard, no URLs         | defaults                              |
   | 100–200   | Standard, no URLs         | `compact: false`                      |
   | 200+      | **Compact**               | `compact: true` (URLs unavailable)    |

3. **URLs are OFF by default** — Only include them when the user explicitly asks for links
   or the result set is small (< 50). Prefer SimBrief and FR24 links when included.

4. **Categorize for clarity** — Group results by airline, route/region, aircraft type,
   duration range, or time of day when it improves readability.

## Search Tool Quick Reference

### `searchFlights` — Primary tool

| Parameter            | Type             | Notes                                        |
|----------------------|------------------|----------------------------------------------|
| `airline`            | string           | ICAO 3-letter code (e.g. `"DLH"`)           |
| `originIcao`         | string           | Departure ICAO (e.g. `"EDDF"`)              |
| `destinationIcao`    | string           | Arrival ICAO (e.g. `"KJFK"`)                |
| `aircraftType`       | string           | ICAO code or common name (`"A359"`, `"787"`) |
| `minDurationMinutes` | number           | Lower bound in minutes                       |
| `maxDurationMinutes` | number           | Upper bound in minutes                       |
| `durationMinutes`    | number           | Approximate target (±30 min)                 |
| `limit`              | number           | Max results (default 50)                     |
| `compact`            | boolean          | Shortened fields, ~82% smaller               |
| `includeUrls`        | boolean or array | `true`, `false`, or `["simbrief","fr24"]`    |

### Supporting tools

- **`getFlightByIdentifier`** — Look up a specific flight (e.g. `"LH765"`).
- **`getFlightByDatabaseId`** — Look up by database ID.
- **`listAircraftTypes`** — List all aircraft in the database.
- **`queryDatabase`** — Run read-only SQL for complex queries.

### MSFS 2024 Addon tools

- **`listMSFSAddons`** — Browse/filter addons (type, developer, installed status, etc.).
- **`createMSFSAddon`** / **`updateMSFSAddon`** / **`deleteMSFSAddon`** / **`getMSFSAddon`** — CRUD for addon entries.

### SimBrief & Weather tools

- **`getLatestFlightPlan`** / **`getDispatchBriefing`** — Retrieve SimBrief flight plans.
- **`getNotams`** — NOTAMs for flight plan airports.
- **`getAviationWeather`** — Current METAR observations.
- **`getVatsimAtis`** — VATSIM ATIS information.

## Presenting Results

Use markdown tables. The standard columns are:

```
| Flight | Aircraft | Route (ICAO) | Duration | Departure (Local) | Arrival (Local) |
```

Add a `| Links |` column only when URLs are included.

**Formatting details:**
- Duration as `HH:MM` (e.g. `09:30`)
- Times with timezone abbreviation (e.g. `03:27 AM IST`)
- Route as `ORIG → DEST`
- Group by airline/category with `###` headers and `---` separators
- Include flight count in each section header

### Multi-airline queries

Group each airline under its own header:
```
### Lufthansa (DLH) — 25 Flights
| ... |
---
### Turkish Airlines (THY) — 22 Flights
| ... |
```

### Duration-based queries

Consider grouping by range (short-haul < 3 h, medium 3–6 h, long 6–9 h, ultra 9+ h).

## Duration Conversion

Always convert user-friendly hours to minutes for the API:
- "around 9 hours" → `durationMinutes: 540`
- "9–9.5 hours" → `minDurationMinutes: 540, maxDurationMinutes: 570`
- "under 3 hours" → `maxDurationMinutes: 180`
- "over 12 hours" → `minDurationMinutes: 720`

## Common Airline ICAO Codes

Lufthansa: DLH · Turkish: THY · Singapore: SIA · Delta: DAL · Iberia: IBE ·
Ethiopian: ETH · China Airlines: CAL · Cathay Pacific: CPA · American: AAL ·
United: UAL · British Airways: BAW · Emirates: UAE · Qatar: QTR · Air France: AFR ·
KLM: KLM

## URL Construction

See `references/url-construction.md` for full templates and field mappings.

**Quick reference:**

| Service       | Template                                                                                       |
|---------------|------------------------------------------------------------------------------------------------|
| SimBrief      | `https://dispatch.simbrief.com/options/custom?airline={AIRLINE}&fltnum={FLIGHTNUMBER}&type={AIRCRAFTTYPE}&orig={ORIGIN}&dest={DESTINATION}` |
| FlightRadar24 | `https://www.flightradar24.com/data/flights/{iata_lowercase}{FLIGHTNUMBER}`                   |
| FlightAware   | `https://flightaware.com/live/flight/{AIRLINE}{FLIGHTNUMBER}`                                 |

When URLs weren't fetched from the server (compact mode or large results), construct them
manually from the flight data fields you already have.

## Compact Mode Field Mapping

Standard → Compact: `flightnumber` → `flt`, `aircrafttype` → `acft`,
`departureicaocode` → `orig`, `arrivalicaocode` → `dest`, `flightdurationmins` → `durMins`,
`departuretimelocal` → `depLocal`, `arrivaltimelocal` → `arrLocal`.

## Helpful Follow-ups

End responses with a relevant suggestion:
- "Would you like SimBrief dispatch links for any of these?"
- "Want to explore different aircraft on this route?"
- "Should I narrow these down by time of day or specific airports?"

## Error Handling

If a search returns few or no results:
1. Suggest broadening criteria (wider duration, remove aircraft filter).
2. Verify ICAO codes are correct.
3. Offer to search similar routes or airlines.
4. Use `listAircraftTypes` to confirm aircraft availability.

## Advanced: Custom SQL

For complex queries the standard search can't handle, use `queryDatabase`:

```sql
SELECT * FROM flightawareflightdata
WHERE airline = 'DLH'
  AND aircrafttype = 'A359'
  AND flightdurationmins BETWEEN 540 AND 570
ORDER BY departuretimelocal
LIMIT 30;
```

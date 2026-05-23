---
name: flight-sim-search
description: >
  Search real-world flight data for flight sim discovery, assemble pre-departure
  briefings, resolve ATC callsigns, and generate validated ICAO routes via the Flight
  Sim Search MCP server. Trigger on aircraft ICAO type codes (A359, A35K, B77W, B789),
  large result requests, duration+aircraft searches, airline operations, sim language
  (SimBrief, MSFS, dispatch, addon), ICAO codes, or casual sim discovery. ALSO trigger
  on flight detail lookups ("details on BA272") — fans out to weather/ATIS/scenery in
  parallel. ALSO trigger on ATC callsign questions ("callsign for SK1462") — cheap
  RouteForge lookup. ALSO trigger on explicit route requests ("route for LH613",
  "textbook routing for EK45") — expensive AI dispatcher, fire only on explicit ask.
  Do NOT trigger for real-world travel planning ("flights to Amsterdam in June",
  "cheapest flight to Tokyo"). Sim users search by aircraft, duration, airline ops,
  or flight numbers; travelers search by destination, dates, and price.
metadata:
  author: user
  version: 2.3
  mcp-server: flight-sim-search
---

# Flight Sim Search Skill

Guide Claude to use the Flight Sim Search MCP server effectively — searching flights,
formatting results, constructing URLs, and managing MSFS 2024 addons.

## When to Use This Skill

### ✅ TRIGGER — Flight simulation discovery

The user wants to find real-world flights to fly in a simulator. Signals include:

- **Aircraft type codes**: "A359", "A35K", "B77W", "B789", "777-200ER", "Dreamliner"
- **Large result sets**: "find me 200 flights", "show me at least 150 options"
- **Duration + aircraft combos**: "8hr flights on the A350", "12-hour 777 routes"
- **Airline operations**: "what does Turkish fly on the A350", "Emirates A380 routes"
- **Sim-specific language**: "SimBrief", "dispatch", "MSFS", "flight sim", "addon"
- **Route exploration**: "long-haul from EDDF", "trans-Pacific routes", "flights between KJFK and EGLL"
- **Casual sim discovery**: "find me a cool long-haul", "interesting overnight flights"
- **ICAO airport codes used naturally**: "routes out of KATL", "flights into RJTT"
- **Addon management**: "what airports do I have installed", "add PMDG 777 to my addons"
- **ATC callsign lookups**: "what's the callsign for SK1462", "ATC callsign for LH400", "what does Scandinavian 1462 squawk?" → use `getFlightCallsign` (cheap, cached). See **ATC Callsigns** below.
- **Filed route requests**: "give me a route for LH613", "what's the textbook routing for EK45", "build me an ICAO route for BA272", "what would dispatch file for SQ322" → see **Route Generation Mode** (expensive — explicit asks only)

### ❌ DO NOT TRIGGER — Real-world travel planning

The user wants to book or plan personal travel. Signals include:

- **Destination + date**: "flights to Amsterdam in early June", "fly to Tokyo next week"
- **Price/booking language**: "cheapest flight to London", "book me a ticket"
- **Personal travel context**: "I'm planning a trip to...", "vacation flights to..."
- **Seat/class preferences**: "business class to Dubai", "direct flights under $500"
- **No aircraft type or sim context**: just destinations and dates without sim indicators

If in doubt, look for aircraft type codes or sim terminology — their presence almost always
means flight sim discovery.

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

**Response includes `callsign`** (or `cs` in compact mode) — the ATC callsign from
the shared cache (e.g. `SK1462 → SAS52L`). Cache-only at the list level; rows
with no cache entry come back as `null`. SimBrief URLs returned via
`includeUrls` are automatically pre-pinned with `&callsign=<atc>` when known.

### Supporting tools

- **`getFlightByIdentifier`** — Look up a specific flight (e.g. `"LH765"`). Returns a `callsign` field — cache hit when available, otherwise falls back to a RouteForge `/atc-callsign` call to fetch fresh.
- **`getFlightCallsign`** — Dedicated lightweight callsign lookup (e.g. `SK1462` → `SAS52L`). Always calls the RouteForge `/atc-callsign` endpoint (shared FR24-backed cache). Use when you need only the callsign — no flight DB lookup, no AI cost.
- **`getFlightByDatabaseId`** — Look up by database ID.
- **`listAircraftTypes`** — List all aircraft in the database.
- **`queryDatabase`** — Run read-only SQL for complex queries.

### MSFS 2024 Addon tools

- **`listMSFSAddons`** — Browse/filter addons (type, developer, installed status, etc.).
- **`createMSFSAddon`** / **`updateMSFSAddon`** / **`deleteMSFSAddon`** / **`getMSFSAddon`** — CRUD for addon entries.

### SimBrief & Weather tools

> Provided by the **Simbrief Flight Plans** MCP connector (separate from Flight
> Sim Search). The skill orchestrates across both because discovery, briefings,
> and route generation flow together.

- **`getLatestFlightPlan`** / **`getDispatchBriefing`** — Retrieve SimBrief flight plans.
- **`getNotams`** — NOTAMs for flight plan airports.
- **`getAviationWeather`** — Current METAR observations.
- **`getVatsimAtis`** — VATSIM ATIS information.
- **`generateFlightRoute`** — Generate an AIRAC-current, validated ICAO route for a specific real flight via the RouteForge AI dispatcher. **Expensive (15–30s, AI tokens)** — only call when the user explicitly asks for a route. See **Route Generation Mode** below.

## Search Strategy

### Single aircraft type, single duration
Straightforward — one `searchFlights` call:
```
searchFlights(aircraftType: "A359", durationMinutes: 480, limit: 200, compact: true)
```

### Multiple aircraft types (e.g. "A359 or A35K")
Run **separate searches per aircraft type** and combine results:
```
searchFlights(aircraftType: "A359", durationMinutes: 480, limit: 200, compact: true)
searchFlights(aircraftType: "A35K", durationMinutes: 480, limit: 200, compact: true)
```
Present results grouped by aircraft type with clear section headers.

### Open-ended exploration ("find me a cool long-haul")
Use broader parameters and curate the results:
- Search with just `minDurationMinutes: 480` and a high limit
- Group results by airline or region for browsability
- Highlight unique or scenic routes

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

## Flight Detail Mode (specific flight lookup)

When the user asks for **details on specific flight numbers** — e.g., "get me details
on BA272, LH444", "tell me more about LH456", "I'm thinking about flying BA272 tonight,
give me the rundown" — they've finished discovery and are picking. Switch from breadth
to depth. This typically follows a search where they narrowed to a few candidates.

### Triggers

- One or more flight numbers + words like "details", "rundown", "breakdown",
  "tell me about", "full picture"
- After a search response, the user names specific flights from your results
- "I'm thinking of flying X tonight" / "what's X like" / "give me the breakdown for X"

If an aircraft type was specified earlier in the conversation (e.g., the search was
filtered to A35K and the user then asks "details on BA272"), filter the lookup to that
type. Otherwise show **all variants** the route has been flown with — pilots often
choose based on which aircraft they own.

### Assembly — run all of these concurrently

This is a fan-out / fan-in pattern. Fire every read in parallel; don't wait for one
before starting the next. None of these calls depend on each other's output, so
serializing them just makes the response feel sluggish.

1. `getFlightByIdentifier` for each flight number. A single number can return
   multiple aircraft entries — that's expected, surface all of them.
2. `getAviationWeather` for every unique origin/destination ICAO across the flights
   (pass them all in one call).
3. `getVatsimAtis` for the same airports — the user flies online, so ATIS is always
   relevant, not signal-gated.
4. `listMSFSAddons` filtered to airport scenery — one call returns the user's whole
   installed airport library, and you cross-reference each ICAO from the flights
   against it.

### Output format

For each flight, render this block. If multiple flights are requested, repeat
separated by `---`.

```markdown
## {FLTNUM} — {Origin City} ({ICAO}) → {Destination City} ({ICAO})

> *{One-line vibe — lighting + scenery + approach character.}*

| Field | Detail |
|-------|--------|
| **Route** | {ICAO} → {ICAO} |
| **Duration** | {HH:MM}{ / {HH:MM} per variant if they differ} |
| **Dep (Local)** | {time tz} |
| **Arr (Local)** | {time tz} |
| **Aircraft** | {type(s), expanded — "A350-1000 (A35K)"} |
| **SELCAL** | {code} |
| **SimBrief** | [Dispatch {flt} {acft}](url) — one row per variant |
| **FlightRadar24** | [{flt} on FR24](url) |

**{Origin ICAO}:** {decoded METAR with hazards called out} · *ATIS {letter},
active {runway}*
**{Destination ICAO}:** {same}

**Scenery:** ✅ {ICAO} installed ({addon name}) · ⚠️ {ICAO} not installed —
recommend {payware suggestion} or default scenery is acceptable.

**Historical aircraft on this route:** {list, recent → older}.
{Optional one-liner of context, e.g. "BA upgraded to A35K in 2024; B77W seasonal."}
```

### Multi-variant handling

Like the LH444 case (A343 + A346 + others over time): one flight number, multiple
historical aircraft entries. Surface all of them — the pilot may own one and not
the other.

- **Aircraft row** — list them: `A343 / A346 / A333`
- **Duration row** — show per-variant if they differ noticeably (`09:14 (A343) / 09:10 (A346)`)
- **SimBrief row** — one link per variant; the dispatch URL changes with `type=`
- **FlightRadar24 row** — one link total; FR24 doesn't care about aircraft type

### Writing the vibe one-liner

One sentence. Pull from what's actually true about the route. Things worth surfacing:

- **Lighting** — sunset/sunrise crossings, redeye, polar darkness, full daylight
  cruise. Reason from departure local time + duration + direction of travel.
- **Scenery** — Alps, NAT tracks, Greenland icecap, Manhattan departure, Hong Kong
  harbor approach, Table Mountain on arrival into FACT, fjords inbound to ENGM.
- **Approach character** — steep visual (KSAN over Balboa Park), RNP-AR valley work
  (LOWI Innsbruck), curved approaches (VHHH IGS, the Kai Tak ghost), circling
  approaches (KEGE Eagle/Vail), short fields, mountain-shadowed descents (LFLL,
  LSZS, LSGS).
- **Operational flavor** — NAT tracks across the pond, ETOPS-180/240 long oceanic
  legs, polar routes crossing 80°N, equatorial weather avoidance.

If nothing genuinely stands out, skip the line rather than padding with generic
phrasing like "a great flight." Empty space is better than filler.

### Weather rendering

Decode METAR. Do not paste raw codes — translate. Lead with the headline:

- Calm/clear: *"EGLL: clear, 6kt variable, vis 10km — textbook day."*
- Marginal: *"EDDF: 1500m in mist, BKN008, 320/18G28 — low-vis ops likely."*
- Hazardous: thunderstorms, gusts > 25kt, vis < 5km, ceiling < 1000ft, icing
  conditions, crosswinds approaching aircraft limits — call them out explicitly
  so the pilot knows what to plan for.

Append ATIS compactly on the same line: *"…ATIS info Charlie, active 27R, 05Z."*
The pilot wants the takeaway, not a wall of code.

### Scenery / addon check

`listMSFSAddons` returns the user's installed library. For each unique ICAO across
the flights:

- ✅ **Installed** — name the addon so they know what they're working with
  ("FlyTampa EGLL", "FSDreamteam KSAN", "iniBuilds EGLL").
- ⚠️ **Not installed** — explicitly call this out and either suggest a known
  payware option (PMDG, FlyTampa, iniBuilds, FSDreamteam, FlightBeam,
  Aerosoft, etc.) or note that the default MSFS 2024 scenery is acceptable
  for less prominent fields.

This matters because pilots often pick which flight to fly based on which
airports they own — surfacing this without being asked is the kind of detail
that makes the briefing feel like it was prepared by someone who actually flies.

### Routed closing offer

Don't end with a generic "anything else?". Pick one targeted next step based on
the signal in the conversation:

- **Sounds committed to a specific flight** → "Want me to pull the full SimBrief
  dispatch briefing? I'll run `getDispatchBriefing` and lay out fuel, alternates,
  and the route."
- **Cautious / weather-watching** → "Should I pull NOTAMs for these airports?"
  (`getNotams`)
- **Mentioned an airport they don't own** → "Want scenery recommendations for
  {ICAO}?" (hand off to `msfs-scenery-researcher` / `msfs-addon-manager`)
- **Still deciding between options** → "Want me to compare these head-to-head,
  or pull alternates on the same route?"

## ATC Callsigns

The ATC callsign (e.g. `SAS52L` for `SK1462`) is what controllers actually read
on frequency, and what VATSIM pilots file. The flight number you'd see on a
ticket is the IATA flight (e.g. `SK1462`). The two often differ and pilots
want both — the IATA to identify the flight, the callsign to file the plan.

Three tools give you a callsign, in increasing cost:

- **From a flight search.** Every `searchFlights` row returns a `callsign` field
  (or `cs` in compact mode). Hits come from the shared cache; misses are
  `null`. Free — same DB round trip as the search itself.
- **From a flight detail lookup.** `getFlightByIdentifier` returns `callsign`
  with a RouteForge fallback on cache miss — so a single specific flight will
  almost always come back with a callsign (paid for by one /atc-callsign call,
  ~1 s).
- **Dedicated lookup.** `getFlightCallsign(flight)` is the right tool when the
  user only asks for the callsign — *"what's the ATC callsign for SK1462?"* —
  with no need to pull the flight DB row. Cheap, lightweight, ~1 s typical.

**Never call `generateFlightRoute` just to get a callsign.** It returns one as
a side effect, but it's a 15–30 s AI-dispatch call costing tokens. Reserve it
for actual route generation requests (see below).

### Using callsigns in SimBrief links

When you have both an IATA flight and a callsign, the SimBrief URL is
automatically pre-pinned with `&callsign=<atc>` — the user lands in SimBrief
with the right callsign already in the form. See `references/url-construction.md`
for the full template, including the further-enhanced form that also pre-fills
the filed route when one is available.

## Route Generation Mode (expensive — explicit requests only)

When the user **explicitly asks for an ICAO route / filed routing / dispatch route
for a specific flight number**, use `generateFlightRoute` (provided by the
**Simbrief Flight Plans** MCP connector). The RouteForge AI dispatcher sources
the real flight's flown track from Flightradar24, snaps it to the current AIRAC
airway network, refines it toward standard conventional routing, and returns a
validated `<SID> <enroute> <STAR>` route plus the real ATC callsign.

### ⚠️ Cost — only call when explicitly asked

This tool runs the Anthropic API internally and takes **15–30 seconds** (up to
~2 minutes on a cold start). Don't fire it speculatively, in fan-outs, or "just
in case." Don't batch it across multiple flights unless the user explicitly asks
— each flight is a separate expensive call.

### ✅ Trigger

- **Route verbs on a specific flight**: "give me a route for LH613", "build me a
  route for EK45", "what's the textbook routing for BA272", "generate an ICAO
  route for KAL85"
- **"Filed route" / "ATC route" / "dispatch route"** with a flight number
- **"What would dispatch file for X"**, "show me the standard route AF11 flies"

### ❌ Do NOT trigger

- General flight discovery (use `searchFlights`)
- Flight Detail Mode (covers weather/ATIS/scenery, not route generation)
- The user names a flight without "route" / "routing" / "dispatch" language
- The user asks for their existing SimBrief plan (use `getLatestFlightPlan`)

### Inputs

Required: `depIcao`, `arrIcao`, `flight`. Optional: `instructions` (defaults to
*"Give me the standard textbook routing."* — only override when the user asks
for something specific like "shortest", "avoid Russian airspace"), `date`,
`depRunway`, `arrRunway`, `referenceRoute`.

**VATSIM coupling.** If the user is flying on VATSIM, call `getVatsimAtis` first
for both airports and parse the active runways from the ATIS text, then pass
them as `depRunway` / `arrRunway` to pin SID/STAR selection.

### Output handling

The response carries both `route_string` (final, AI-refined) and `baseline_route`
(pre-AI track snap). **Always present `route_string`** — that's the validated
route the user should file; the baseline is for transparency only and should
never be copied. Surface the **callsign** too (e.g. `"DLH613"` for LH613) — it's
the real ATC callsign sourced from FR24, not just the IATA flight number.

If `validation.ok` is false, flag the errors before offering links. The
dispatcher usually iterates until clean, so failures are rare but worth catching.

### Enhanced SimBrief link

When the response gives you both `callsign` and `route_string`, build an
**enhanced** SimBrief URL that pre-fills the filed routing — the user lands in
SimBrief with the route already loaded:

```
https://dispatch.simbrief.com/options/custom?airline={AIRLINE}&fltnum={FLTNUM}&type={ACFT}&orig={DEP}&dest={ARR}&callsign={CALLSIGN}&route={URL_ENCODED_ROUTE}
```

- `callsign` → split: first 3 chars → `airline`, remainder → `fltnum`
  (`"DLH613"` → `airline=DLH`, `fltnum=613`).
- Include `type` if you know the aircraft from earlier conversation context
  (a prior `searchFlights` / `getFlightByIdentifier`). Omit if unknown —
  SimBrief will prompt.
- **URL-encode `route_string`** — spaces become `%20`. The route is unusable
  without this.

See `references/url-construction.md` for the full template and a worked example.

### Output format

```markdown
## LH613 — UBBB → EDDF · DLH613 · FL340

**Route:**
\`\`\`
AGDA1C AGDAM M747 LUSAL M11 REBLO UM11 CRM Q162 SPESA SPES4G
\`\`\`

✅ Valid on the current AIRAC cycle.

**SimBrief (pre-filled):** [Dispatch DLH613](https://dispatch.simbrief.com/options/custom?airline=DLH&fltnum=613&orig=UBBB&dest=EDDF&callsign=DLH613&route=AGDA1C%20AGDAM%20M747%20...)

> *Dispatcher briefing:* {2–4 sentences from the response.}
```

## Duration Conversion

Always convert user-friendly hours to minutes for the API:
- "around 9 hours" → `durationMinutes: 540`
- "9–9.5 hours" → `minDurationMinutes: 540, maxDurationMinutes: 570`
- "under 3 hours" → `maxDurationMinutes: 180`
- "over 12 hours" → `minDurationMinutes: 720`
- "8hr flights" → `durationMinutes: 480`
- "at least 6 hours" → `minDurationMinutes: 360`

## Common Airline ICAO Codes

Lufthansa: DLH · Turkish: THY · Singapore: SIA · Delta: DAL · Iberia: IBE ·
Ethiopian: ETH · China Airlines: CAL · Cathay Pacific: CPA · American: AAL ·
United: UAL · British Airways: BAW · Emirates: UAE · Qatar: QTR · Air France: AFR ·
KLM: KLM

## URL Construction

See `references/url-construction.md` for full templates and field mappings.

**Quick reference:**

| Service              | Template                                                                                       |
|----------------------|------------------------------------------------------------------------------------------------|
| SimBrief             | `https://dispatch.simbrief.com/options/custom?airline={AIRLINE}&fltnum={FLIGHTNUMBER}&type={AIRCRAFTTYPE}&orig={ORIGIN}&dest={DESTINATION}` |
| **SimBrief + callsign** | Append `&callsign={CALLSIGN}` (callsigns come from `searchFlights` / `getFlightByIdentifier` / `getFlightCallsign`; the server already does this when URLs are returned). |
| **Enhanced SimBrief** | Append `&callsign={CALLSIGN}&route={URL_ENCODED_ROUTE}` — use after `generateFlightRoute` returns both. Spaces in the route MUST be `%20`-encoded. |
| FlightRadar24        | `https://www.flightradar24.com/data/flights/{iata_lowercase}{FLIGHTNUMBER}`                   |
| FlightAware          | `https://flightaware.com/live/flight/{AIRLINE}{FLIGHTNUMBER}`                                 |

When URLs weren't fetched from the server (compact mode or large results), construct them
manually from the flight data fields you already have.

## Compact Mode Field Mapping

Standard → Compact: `flightnumber` → `flt`, `aircrafttype` → `acft`,
`departureicaocode` → `orig`, `arrivalicaocode` → `dest`, `flightdurationmins` → `durMins`,
`departuretimelocal` → `depLocal`, `arrivaltimelocal` → `arrLocal`, `callsign` → `cs`.

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

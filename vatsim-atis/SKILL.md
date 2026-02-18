---
name: vatsim-atis
description: >
  Quickly retrieve and present VATSIM ATIS (Automatic Terminal Information Service)
  and real-world METAR weather for airports. Use this skill whenever the user asks
  for "ATIS", "what's the weather at [airport]", "what runway is active", "who's
  controlling", "is anyone online at [airport]", or any request for current airport
  conditions during flight planning or while flying. Also trigger for phrases like
  "check conditions at", "what's the field like at", or just an ICAO code by itself
  (e.g., "KJFK" or "EGLL"). Handles single airports or multiple at once.
---

# VATSIM ATIS Skill

Retrieve and present VATSIM ATIS and aviation weather in a quick, scannable
format suitable for mid-flight or pre-flight use.

## Workflow

### 1 — Identify Airports

Extract ICAO codes from the user's request. Accept:
- Explicit ICAO codes: "KJFK", "EGLL"
- Airport names: "Heathrow" → EGLL, "JFK" → KJFK, "O'Hare" → KORD
- Context references: "my departure and arrival" → pull from active flight plan
- Multiple airports in one request: "KJFK and EGLL"

If the user says something like "check ATIS for my flight" or references a
flight plan, call `getLatestFlightPlanSummary` first to extract the departure,
arrival, and alternate ICAO codes — then proceed.

### 2 — Fetch Data

Make **both** calls in the same turn for the fastest response:

- **`getVatsimAtis`** with all ICAO codes (up to 20)
- **`getAviationWeather`** with the same ICAO codes, set `includeTaf: true`

### 3 — Present Results

For **each airport**, present a compact block:

```
## KJFK — John F. Kennedy Intl

**VATSIM ATIS** (Info Alpha · 128.725)
Runway 31L in use · ILS approaches · Wind 320/12
Controller: JFK_ATIS (online since 14:32Z)

**METAR** (VFR ✅)
Wind 320° at 12 kt · Visibility 10+ sm · Few clouds at 4,500 ft
Temp 8°C / Dewpoint 2°C · Altimeter 30.12 inHg
`KJFK 171456Z 32012KT 10SM FEW045 08/02 A3012`

**TAF Summary**
Next 6 hrs: Winds shifting 350/15, visibility remaining good.
After 22Z: Ceiling lowering to BKN025, possible light rain.
```

### No VATSIM Controller Online

When no ATIS is available (common — this is normal):

```
## KJFK — John F. Kennedy Intl

**VATSIM ATIS**: No active controllers at this time.

**METAR** (VFR ✅)
Wind 320° at 12 kt · Visibility 10+ sm · Few clouds at 4,500 ft
Temp 8°C / Dewpoint 2°C · Altimeter 30.12 inHg
`KJFK 171456Z 32012KT 10SM FEW045 08/02 A3012`
```

Still present the real-world METAR/TAF — that's always useful even without VATSIM
controllers.

## METAR Decoding

Always decode the METAR into plain language. Key fields:

- **Wind**: Direction, speed, gusts (flag gusts > 25 kt or crosswind concerns)
- **Visibility**: In statute miles or meters
- **Ceiling/Clouds**: Layer types and altitudes (flag < 1000 ft AGL)
- **Weather phenomena**: Rain, snow, fog, thunderstorms — spell them out
- **Temp/Dewpoint**: Flag when spread < 3°C (fog risk)
- **Altimeter**: inHg or QNH as appropriate for region
- **Flight category**: VFR ✅ / MVFR 🟡 / IFR 🟠 / LIFR 🔴

Include the raw METAR string in a code-formatted line below the decoded version.

## TAF Presentation

Summarize the TAF in plain language focused on the next 6–12 hours:
- Significant changes (wind shifts, ceiling drops, visibility reduction)
- Tempo/probability groups that affect operations
- When conditions are expected to improve or deteriorate

Don't dump the raw TAF unless the user asks for it.

## Response Style

- **Concise** — the user may be mid-flight; get to the point
- **Scannable** — use the block format above, bold key labels
- **Flag concerns** with ⚠️: low ceilings, strong crosswinds, gusts, low vis,
  significant weather, rapidly deteriorating conditions
- When multiple airports are requested, present them in the order: departure →
  arrival → alternate (or the order the user asked)
- Don't add lengthy intros or outros — just the data

## Multiple Airport Efficiency

Both `getVatsimAtis` and `getAviationWeather` accept arrays of ICAO codes.
Always batch them into single calls rather than making separate calls per airport.

## Follow-Up Suggestions

Keep these brief — one line at most:
- "Want me to check NOTAMs for any of these?"
- "Need a full briefing for this flight?"
- "Want me to refresh these in a few minutes?"

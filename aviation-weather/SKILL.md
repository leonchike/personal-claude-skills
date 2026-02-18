---
name: aviation-weather
description: >
  Retrieve and decode real-world METAR and TAF aviation weather for airports.
  Use this skill whenever the user asks for "METAR", "TAF", "weather at [airport]",
  "conditions at [airport]", "what's the ceiling", "wind check", "altimeter setting",
  "flight category", or any request for current or forecast airport weather. Also
  trigger for phrases like "is the weather good at", "can I land VFR at", "what's
  the vis at", or just an ICAO code when the context implies weather (e.g., "KJFK
  weather"). Handles single airports or multiple at once. Provides decoded
  plain-language weather alongside raw METAR/TAF strings.
---

# Aviation Weather Skill

Retrieve and present real-world METAR observations and TAF forecasts in a quick,
pilot-friendly format. Designed for fast lookups during flight planning or
mid-flight.

## Workflow

### 1 — Identify Airports

Extract ICAO codes from the user's request. Accept:
- Explicit ICAO codes: "KJFK", "EGLL"
- IATA codes or airport names: "JFK" → KJFK, "Heathrow" → EGLL, "O'Hare" → KORD
- Context references: "my departure and arrival" → pull from active flight plan
- Multiple airports: "weather at KJFK, KORD, and KLAX"

If the user references a flight plan (e.g., "weather for my flight"), call
`getLatestFlightPlanSummary` first to extract departure, arrival, and alternate
ICAO codes — then proceed.

### 2 — Fetch Weather

Call **`getAviationWeather`** with all ICAO codes in a single request and
**`includeTaf: true`** (always include TAF unless the user only asks for METAR):

```
getAviationWeather(icaoCodes: ["KJFK", "KORD", "KLAX"], includeTaf: true)
```

Both METAR and TAF come back in one call — no need for separate requests.

### 3 — Present Results

For **each airport**, present a compact block:

```
## KJFK — John F. Kennedy Intl 🟢 VFR

**METAR**
Wind 320° at 12 kt · Visibility 10+ sm · Few clouds at 4,500 ft
Temp 8°C / Dewpoint 2°C · Altimeter 30.12 inHg
`KJFK 171456Z 32012KT 10SM FEW045 08/02 A3012`

**TAF Summary**
Valid 17/1500Z – 18/1500Z
- Now–22Z: Winds 320/12, good visibility, few clouds 4,500 ft
- 22Z–04Z: Winds shifting 350/15G22, ceiling lowering BKN025, light rain likely
- After 04Z: Improving — SCT040, winds 010/10
```

### Flight Category Badges

Always show the flight category prominently with a color indicator:
- **VFR** 🟢 — Ceiling > 3,000 ft AND visibility > 5 sm
- **MVFR** 🟡 — Ceiling 1,000–3,000 ft OR visibility 3–5 sm
- **IFR** 🟠 — Ceiling 500–999 ft OR visibility 1–3 sm
- **LIFR** 🔴 — Ceiling < 500 ft OR visibility < 1 sm

## METAR Decoding

Always decode every METAR into plain language. Key fields to present:

- **Wind**: Direction, speed, gusts. Flag gusts > 25 kt with ⚠️. Note variable
  winds (e.g., "Variable 280–340° at 6 kt")
- **Visibility**: Statute miles (US) or meters (international)
- **Weather phenomena**: Decode all abbreviations into plain English:
  - RA = rain, SN = snow, FG = fog, BR = mist, TS = thunderstorm,
    HZ = haze, DZ = drizzle, FZ = freezing, SH = showers, GR = hail
  - Intensity: `-` light, (none) moderate, `+` heavy
  - Example: `-TSRA` → "Light thunderstorms with rain"
- **Clouds**: Layer type and altitude AGL (FEW/SCT/BKN/OVC + height)
- **Temp/Dewpoint**: In °C. Flag spread < 3°C with ⚠️ (fog/mist risk)
- **Altimeter**: inHg for US airports, QNH hPa for international
- **Remarks**: Decode significant RMK items (peak wind, pressure tendency,
  precipitation, runway visual range)

Always include the raw METAR string in backticks below the decoded version.

## TAF Presentation

Summarize the TAF in **plain language organized by time periods**:

- Break into logical time blocks (now, next few hours, later)
- Focus on **changes**: wind shifts, ceiling drops, visibility reduction,
  precipitation onset/ending
- Decode TEMPO groups: "Temporarily between 18Z–21Z: visibility dropping to 2 sm
  in moderate rain"
- Decode PROB groups: "30% chance between 02Z–05Z: thunderstorms"
- Highlight the worst expected conditions in the forecast window
- Note when conditions are expected to improve

Only show the raw TAF string if the user explicitly asks for it.

## Flagging Concerns

Use ⚠️ to flag operationally significant conditions:

- Ceiling below 1,000 ft AGL
- Visibility below 3 sm
- Crosswind or gusts > 25 kt
- Thunderstorms (TS) present or forecast
- Freezing precipitation (FZRA, FZDZ)
- Wind shear (WS in METAR remarks or TAF)
- Temp/dewpoint spread < 3°C (fog risk)
- Rapidly deteriorating conditions in TAF
- RVR reported (indicates low-vis operations)

## Response Style

- **Concise** — the user wants weather fast, not a weather lecture
- **Scannable** — bold labels, consistent block format, flight category badge
  at the top
- When multiple airports are requested, present in order: departure → arrival →
  alternate, or the order the user specified
- Don't add lengthy intros — go straight to the weather blocks
- One-line closing at most

## Multiple Airport Efficiency

`getAviationWeather` accepts an array of up to 10 ICAO codes. Always batch into
a single call rather than separate calls per airport.

## Comparison Requests

If the user asks "which airport has better weather" or "compare conditions at":
- Present each airport's weather block
- Add a brief **comparison summary** at the end noting which has better
  ceiling, visibility, and wind conditions
- Recommend the more favorable option if one is clearly better

## Follow-Up Suggestions

Keep brief — one line:
- "Want me to check VATSIM ATIS for these airports too?"
- "Need NOTAMs for any of these?"
- "Want a full briefing for your flight?"

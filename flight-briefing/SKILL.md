---
name: flight-briefing
description: >
  Create comprehensive, professional pre-departure flight briefings for airline
  pilots using SimBrief flight plan data and VATSIM ATIS. Use this skill whenever
  the user asks for a "briefing", "pre-departure brief", "flight brief", "departure
  brief", "pilot briefing", or says things like "brief me", "what's the plan for
  my flight", "give me the rundown", or "prepare my flight". Also trigger when the
  user has a SimBrief flight plan and wants it presented in a structured pilot-ready
  format — even if they just say "pull up my flight plan" or "what did I file".
  This skill covers the full briefing workflow: retrieving the flight plan, checking
  VATSIM ATIS, reviewing NOTAMs, and assembling a professional 10-section briefing.
---

# Flight Briefing Skill

Create professional pre-departure briefings from SimBrief flight plans, VATSIM
ATIS data, and NOTAMs. The briefing follows a structured 10-section format that
mirrors real airline operations briefing flow.

> **Before writing any briefing**, read `references/briefing-template.md` for the
> full section-by-section format and example content. That file is the authoritative
> guide for structure and tone.

## Workflow

Every briefing follows this sequence. Do not skip steps 1–3.

### Step 1 — Retrieve the Flight Plan

Call **`getLatestFlightPlan`** (Simbrief Flight Plans MCP). This is the default
and preferred tool — it returns comprehensive
markdown (~50–80 KB) with everything needed for a full briefing:

- Route, aircraft, flight time, distance
- Fuel breakdown and weight & balance
- Takeoff performance: V1, VR, V2, VREF for all runways, flex temps, max weights,
  distances (decision/reject/continue), wind components, flap settings
- Landing performance: distances (dry/wet), VREF, max landing weights per runway
- Weather: METAR, TAF, SIGMETs for departure, arrival, and alternate
- Critical NOTAMs (top 5 per airport, auto-filtered)
- NAT tracks (oceanic crossings)
- ETOPS data (if applicable)
- Performance impact analysis (CI, altitude, weight variations)
- ATC flight plan text
- Navigation waypoints with altitude, wind, fuel

If the user provides a specific plan ID, use `getFlightPlanById` instead.

Only use `getDispatchBriefing` or `getLatestFlightPlanSummary` if the user
explicitly asks for a "quick brief" or "summary" — these omit critical detail.

### Step 2 — Check VATSIM ATIS

Extract the departure, arrival, and alternate ICAO codes from the flight plan,
then call **`getVatsimAtis`** with all of them in a single request:

```
getVatsimAtis(["KJFK", "EGLL", "KBOS"])
```

- If controllers are online, include the ATIS code, frequency, runway in use,
  and relevant remarks in the appropriate briefing sections.
- If no ATIS is active, note "No active VATSIM controllers at this time" in each
  relevant section. This is normal and expected.

### Step 3 — Review NOTAMs (if needed)

The main flight plan already includes the top 5 critical NOTAMs per airport. Use
the **`getNotams`** tool only when:

- The user specifically asks about NOTAMs
- You need complete NOTAM details beyond the critical ones
- A critical NOTAM raises questions that need more context
- The user wants to review all NOTAMs for a specific airport

Parameters: `airport` ("origin", "destination", "alternate",
or "all").

### Step 4 — Check Weather (supplemental)

The flight plan includes METAR/TAF, but you can use **`getAviationWeather`** to
pull fresh METARs if the plan data seems stale or the user asks for a live
weather update. Set `includeTaf: true` for forecast data.

### Step 5 — Identify Gate/Terminal

If the user provides a gate, use it. Otherwise:
- Try a quick web search for the airline's typical terminal/gates at the
  departure and arrival airports
- If nothing is found, note it as TBD and move on — don't block the briefing

### Step 6 — Assemble the Briefing

Follow the 10-section structure defined in `references/briefing-template.md`.
Every section draws from specific data in the flight plan output. The reference
file maps each section to its data source.

## Tool Selection Quick Reference

| Need | Tool | When |
|------|------|------|
| Full flight plan (DEFAULT) | `getLatestFlightPlan` | Always use first |
| Specific plan by ID | `getFlightPlanById` | User provides plan ID |
| Quick summary only | `getDispatchBriefing` | User asks for "quick brief" |
| VATSIM controller info | `getVatsimAtis` | Always — after getting flight plan |
| Complete NOTAMs | `getNotams` | User asks, or critical NOTAMs need context |
| Fresh METAR/TAF | `getAviationWeather` | Supplemental weather check |
| Gate/terminal lookup | Web search | When gate not provided |

## Tone and Style

- **Professional but conversational** — like a dispatcher briefing a crew
- **Safety-focused** — flag anything non-standard prominently
- Use aviation terminology naturally (don't over-explain standard terms)
- Use markdown formatting: headers, tables for performance data, bold for
  critical values
- Sparingly use emojis for visual breaks (✈️ 🌤️ ⚠️ — not every line)
- End with a brief encouraging sign-off
- When presenting V-speeds and performance data, use tables for clarity
- Highlight any discrepancies between ATIS and flight plan (e.g., different
  active runway than planned)

## Safety Focus Areas

Flag these prominently whenever they appear:

- **Runway closures or restrictions** from NOTAMs
- **Weather below minimums** or deteriorating conditions
- **Performance limits** — runways where weight is restricted
- **Navigation equipment outages** affecting planned procedures
- **ETOPS concerns** — fuel, adequate airports, weather at diversion points
- **SIGMET activity** along the route
- **Discrepancies** — ATIS runway vs planned runway, wind shifts, etc.
- **Contaminated runways** — wet/snow/ice conditions affecting distances

## What NOT to Do

- Don't fabricate V-speeds, distances, or performance data — it's all in the
  flight plan output. Extract and present it.
- Don't skip calling `getVatsimAtis` — even if you expect no controllers online,
  always check.
- Don't dump raw METAR strings without interpretation — decode winds, visibility,
  ceiling, and weather phenomena into plain language alongside the raw data.
- Don't overwhelm with every NOTAM — the critical ones are pre-filtered. Use
  `getNotams` only when the user needs the full picture.
- Don't make up gate assignments — search or note as TBD.

## Handling Edge Cases

**Oceanic flights**: Include NAT track information, SELCAL codes, oceanic
communication procedures, and ETOPS data. These are all in the flight plan.

**Short-haul / domestic**: Skip oceanic and ETOPS sections. Focus on departure
performance, weather, and arrival.

**Missing data**: If any section of the flight plan is empty or unavailable, note
it clearly and move on. Never fabricate data to fill gaps.

**Multiple alternates**: Brief all alternates listed in the flight plan with
weather and approach capabilities.

**User asks follow-up questions**: You already have the flight plan data in
context. Answer directly without re-fetching unless they ask for a refresh.

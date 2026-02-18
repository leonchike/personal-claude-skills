# Briefing Template Reference

This is the authoritative structure for pre-departure briefings. Follow these
10 sections in order. Each section lists what to include and where the data
comes from in the `getLatestFlightPlan` output.

---

## Summary Block

Present at the very top before the numbered sections. Quick-scan info:

- **Flight number** and route (ICAO codes and city names)
- **Date** and estimated departure time (with timezone)
- **Departure gate** (from user or web search; "TBD" if unknown)
- **Aircraft type** and registration (if available)

Example:
```
✈️ **UAL154 | KORD → EGLL**
📅 15 Feb 2026 · Dep 17:45 CST
🛫 Gate C17 · Boeing 787-10 Dreamliner
```

---

## Section 1 — Aircraft & Route Overview

**Data sources**: Flight info header, route string, ETOPS section

Present:
- Aircraft type with engine variant (e.g., "Boeing 787-10 / GEnx-1B76")
- Route summary: departure → key waypoints → arrival
- Total distance (nm) and planned flight time
- Cost Index setting
- Special procedures flags:
  - ETOPS: include rule, diversion time, critical fuel if applicable
  - Oceanic: NAT track assignment if applicable
  - RNAV/RNP requirements from the flight plan

---

## Section 2 — Departure Information

**Data sources**: Takeoff performance section, weather (departure), VATSIM ATIS

This is typically the most data-rich section. Organize it into sub-sections:

### Runway & Performance

Present the **planned runway** data in a focused table:

```
| Parameter       | Value          |
|-----------------|----------------|
| Runway          | 10C            |
| V1              | 148 kt         |
| VR              | 153 kt         |
| V2              | 158 kt         |
| Flex Temp       | 62°C           |
| Flap Setting    | 5              |
| Takeoff Weight  | 227,500 kg     |
| Headwind        | 12 kt          |
| Crosswind       | 5 kt (L)       |
```

Then provide a summary table of **all available runways** showing key differences:

```
| Runway | V1  | VR  | V2  | Flex | Max Wt    | Headwind | Limit  |
|--------|-----|-----|-----|------|-----------|----------|--------|
| 10C    | 148 | 153 | 158 | 62°C | 228,400   | 12 kt    | —      |
| 10L    | 146 | 151 | 156 | 59°C | 225,100   | 11 kt    | Obs    |
| 28R    | 152 | 157 | 162 | 55°C | 220,300   | -8 kt    | Rwy Ln |
```

Note any performance-limited runways and why (obstacle, runway length, etc.).

### Takeoff Distances

For the planned runway, include:
- Decision distance, reject distance, continue distance
- Available margin
- Surface condition (dry/wet)

### Weather at Departure

- Decode the METAR into plain language: wind, visibility, ceiling, temp/dewpoint,
  altimeter
- Include the raw METAR string below the decoded version
- TAF summary for the departure window
- Flag any weather concerns (low vis, crosswind limits, icing, thunderstorms)

### VATSIM ATIS (Departure)

If active:
- ATIS code and frequency
- Active runway from ATIS
- ⚠️ If ATIS runway differs from planned runway, flag this prominently
- Relevant remarks (e.g., "Expect ILS approaches in use")

If not active:
- "No active VATSIM controllers at departure airport."

### Gate & Terminal

- Gate assignment and terminal
- Aircraft type compatibility notes if relevant
- Taxi considerations for the planned runway

---

## Section 3 — Route & Navigation

**Data sources**: Route/waypoints section, SID/STAR, NAT tracks, ATC flight plan

### Departure Routing
- SID name and transition
- Initial altitude assignments
- First few key waypoints after departure

### En Route
- Cruise altitude(s) and any step climbs
- Key waypoints with coordinates (especially FIR boundaries)
- Oceanic track assignment if applicable (NAT, PACOTS)
- Airspace transitions

### Arrival Routing
- STAR name and transition
- Expected approach type (ILS, RNAV, VOR)
- Planned landing runway
- Approach frequencies (ILS freq if applicable)

### ATC Flight Plan
Include the ICAO ATC flight plan string for reference.

---

## Section 4 — Fuel Planning

**Data sources**: Fuel planning section, performance impact analysis

Present as a structured breakdown:

```
| Fuel Component     | Amount        |
|--------------------|---------------|
| Trip Fuel          | 45,230 kg     |
| Contingency        | 2,260 kg      |
| Alternate          | 3,450 kg      |
| Final Reserve      | 2,800 kg      |
| Taxi Fuel          | 600 kg        |
| Extra Fuel         | 1,000 kg      |
| **Total Block Fuel** | **55,340 kg** |
```

Also include:
- Planned landing fuel
- Minimum diversion fuel (ETOPS flights)
- Fuel flow assumptions (average, step climb impacts)
- Any notes from the performance impact analysis about fuel sensitivity

---

## Section 5 — Weight & Balance

**Data sources**: Weight section of the flight plan

```
| Weight           | Value         | Max           |
|------------------|---------------|---------------|
| ZFW              | 172,200 kg    | 181,000 kg    |
| Takeoff Weight   | 227,500 kg    | 252,000 kg    |
| Landing Weight   | 179,800 kg    | 193,000 kg    |
```

Note margins to max weights and any limitations.

---

## Section 6 — Destination Information

**Data sources**: Landing performance section, weather (arrival), VATSIM ATIS

### Landing Performance

For the planned landing runway:

```
| Parameter          | Value         |
|--------------------|---------------|
| Runway             | 27L           |
| VREF               | 142 kt        |
| Landing Wt         | 179,800 kg    |
| Distance (Dry)     | 1,650 m       |
| Distance (Wet)     | 2,150 m       |
| Headwind           | 8 kt          |
| Crosswind          | 12 kt (R)     |
```

Summary of all available landing runways if multiple exist.

### Weather at Destination

- Decoded METAR + raw string
- TAF summary for arrival window
- Weather category (VFR/MVFR/IFR/LIFR)
- Any operational impacts

### VATSIM ATIS (Arrival)

Same format as departure ATIS. Flag runway discrepancies.

### Airport Information

- Expected gate/terminal (from user, web search, or TBD)
- Runway status from NOTAMs
- Any operational considerations (curfews, noise abatement, etc.)

---

## Section 7 — Alternate Airport(s)

**Data sources**: Alternate section of flight plan, weather for alternate

For each alternate:

- Airport name and ICAO code
- Distance and fuel required
- Weather: decoded METAR + raw string
- Approach capabilities (ILS, RNAV, etc.)
- Runway information

### VATSIM ATIS (Alternate)

Check and include if active.

---

## Section 8 — NOTAMs & Restrictions

**Data sources**: Critical NOTAMs section (auto-included in flight plan)

### Departure Airport NOTAMs
List critical NOTAMs with category tags:
- 🔴 **Runway**: closures, length restrictions
- 🟡 **Lighting**: outages affecting approaches
- 🟡 **Navigation**: ILS/VOR outages
- 🟠 **Obstacles**: cranes, construction
- 🔵 **Aerodrome**: other operational notices

### Destination Airport NOTAMs
Same format.

### En Route Restrictions
- Airspace restrictions
- Military activity areas
- Temporary flight restrictions

If the auto-filtered NOTAMs seem insufficient or the user asks, note that full
NOTAMs are available via the `getNotams` tool.

---

## Section 9 — Weather Hazards

**Data sources**: SIGMETs section, TAFs, performance impact analysis

- Active SIGMETs along the route (turbulence, icing, thunderstorms, volcanic ash)
- En route turbulence forecast
- Icing conditions and altitude bands affected
- Jet stream position and impact
- Destination weather trends (improving/deteriorating)
- Temperature deviations from ISA (affects performance)

---

## Section 10 — Pre-Flight Reminders

Context-sensitive reminders based on the specific flight. Select relevant items:

**Always include:**
- Verify fuel load matches planned block fuel
- Confirm takeoff data (V-speeds, flex temp) in FMS
- Review emergency procedures for departure airport

**Oceanic flights:**
- SELCAL check before oceanic entry
- HF radio verification
- Oceanic clearance procedures
- ETP and ETOPS kit verification

**ETOPS flights:**
- Verify ETOPS MEL compliance
- Confirm adequate airport weather
- Review diversion procedures and fuel requirements

**Weather considerations:**
- De-icing procedures if applicable
- Turbulence avoidance altitudes
- Alternate approach briefing if weather marginal

**Equipment:**
- Navigation capability verification (RNP, RNAV)
- Communication equipment status
- Autoland capability if needed for low-vis approach

---

## Sign-Off

End the briefing with a brief, professional closing. Examples:

> "Safe skies on your run to London. Enjoy the flight! ✈️"
> "Smooth flying ahead to Istanbul — have a great trip!"
> "Weather looks cooperative today. Have a safe flight to São Paulo! 🌤️"

Keep it short, warm, and specific to the route.

---

## Formatting Notes

- Use `##` for section headers, `###` for sub-sections
- Tables for any data with 3+ fields — performance data especially
- **Bold** for critical values (V-speeds, fuel totals, weather warnings)
- ⚠️ emoji prefix for any safety flag or discrepancy
- Keep raw METAR/TAF in code blocks for readability
- Don't repeat data across sections — reference it ("as noted in Section 2")

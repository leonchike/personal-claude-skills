# URL Construction Reference

Detailed templates and field mappings for manually constructing flight tracking and
dispatch URLs from search result data.

## When to Construct URLs Manually

| Scenario                              | Approach                                       |
|---------------------------------------|------------------------------------------------|
| Small result set (< 50 flights)       | Use `includeUrls: true` — let server generate  |
| User explicitly asks for links        | Use `includeUrls: ["simbrief"]` etc.            |
| Large result set (100+ flights)       | Construct manually only if user asks            |
| Compact mode (200+ flights)           | Always construct manually (no URLs in response) |
| User asks for a link after seeing results | Construct from data already retrieved        |

---

## SimBrief Dispatch URL

**Template:**
```
https://dispatch.simbrief.com/options/custom?airline={AIRLINE}&fltnum={FLIGHTNUMBER}&type={AIRCRAFTTYPE}&orig={ORIGIN}&dest={DESTINATION}
```

### Field Mapping

| Placeholder       | Standard Mode Field   | Compact Mode Field |
|--------------------|-----------------------|--------------------|
| `{AIRLINE}`        | `airline`             | `airline`          |
| `{FLIGHTNUMBER}`   | `flightnumber`        | `flt`              |
| `{AIRCRAFTTYPE}`   | `aircrafttype`        | `acft`             |
| `{ORIGIN}`         | `departureicaocode`   | `orig`             |
| `{DESTINATION}`    | `arrivalicaocode`     | `dest`             |

### Example

From standard mode response:
```json
{
  "airline": "DLH",
  "flightnumber": "765",
  "aircrafttype": "A359",
  "departureicaocode": "VOBL",
  "arrivalicaocode": "EDDM"
}
```

Constructed URL:
```
https://dispatch.simbrief.com/options/custom?airline=DLH&fltnum=765&type=A359&orig=VOBL&dest=EDDM
```

---

## SimBrief URL with Callsign (from a search)

When `searchFlights`, `getFlightByIdentifier`, or `getFlightCallsign` returns a
`callsign`, append it to the standard SimBrief URL as `&callsign={CALLSIGN}` —
the user lands in SimBrief with the right ATC callsign pre-filled. The
find-flights MCP already does this for you whenever you request URLs
(`includeUrls: ["simbrief"]` etc.); you only need to construct it by hand in
compact mode or when you skipped `includeUrls`.

**Template (with callsign, no route):**
```
https://dispatch.simbrief.com/options/custom?airline={AIRLINE}&fltnum={FLIGHTNUMBER}&type={AIRCRAFTTYPE}&orig={ORIGIN}&dest={DESTINATION}&callsign={CALLSIGN}
```

`{AIRLINE}` / `{FLIGHTNUMBER}` come from the flight row (ICAO airline + flight
number, e.g. `SAS` + `1462`). `{CALLSIGN}` is the ATC callsign (e.g. `SAS52L`)
— these are not the same thing; the IATA pair is *where the flight is*, the
callsign is *what controllers hear*. SimBrief takes both.

If `callsign` is `null` (cache miss on `searchFlights`, or RouteForge failed in
`getFlightByIdentifier`), just omit the `&callsign=` parameter — the URL is
still useful.

---

## Enhanced SimBrief URL (Route + Callsign)

When the `generateFlightRoute` tool has been invoked and returned both a
`callsign` and a `route_string`, build an **enhanced** SimBrief URL that
pre-fills the filed routing and callsign — the user lands in SimBrief with the
route already loaded and just clicks Generate.

**Template:**
```
https://dispatch.simbrief.com/options/custom?airline={AIRLINE}&fltnum={FLTNUM}&type={AIRCRAFTTYPE}&orig={ORIGIN}&dest={DESTINATION}&callsign={CALLSIGN}&route={URL_ENCODED_ROUTE}
```

### Field Mapping

| Placeholder            | Source                                                                                |
|------------------------|---------------------------------------------------------------------------------------|
| `{AIRLINE}`            | First 3 characters of `callsign` (`"DLH613"` → `DLH`)                                 |
| `{FLTNUM}`             | Remainder of `callsign` after the airline prefix (`"DLH613"` → `613`)                 |
| `{AIRCRAFTTYPE}`       | Aircraft type from the same response (or a prior `searchFlights` / `getFlightByIdentifier`) — omit if unknown |
| `{ORIGIN}`             | `dep_icao` from the response                                                          |
| `{DESTINATION}`        | `arr_icao` from the response                                                          |
| `{CALLSIGN}`           | `callsign` from the response (full string, e.g. `"DLH613"`)                           |
| `{URL_ENCODED_ROUTE}`  | `route_string` from the response, **URL-encoded** (spaces → `%20`)                    |

> Note: when the callsign comes from `searchFlights` / `getFlightByIdentifier`
> instead (e.g. `SAS52L` for `SK1462`), it usually isn't of the form
> `<3-letter airline><number>` — the suffix is alphanumeric (the trailing `L`
> in `SAS52L`). In that case prefer the IATA airline and flightnumber from the
> same response row for `{AIRLINE}` / `{FLTNUM}`, and pass the full callsign
> as `{CALLSIGN}`. SimBrief treats the three fields independently.

### Worked Example

From a `generateFlightRoute` response:

```json
{
  "dep_icao": "UBBB",
  "arr_icao": "EDDF",
  "callsign": "DLH613",
  "route_string": "AGDA1C AGDAM M747 LUSAL M11 REBLO UM11 CRM Q162 SPESA SPES4G"
}
```

With aircraft type `A21N` known from earlier conversation, the enhanced URL is:

```
https://dispatch.simbrief.com/options/custom?airline=DLH&fltnum=613&type=A21N&orig=UBBB&dest=EDDF&callsign=DLH613&route=AGDA1C%20AGDAM%20M747%20LUSAL%20M11%20REBLO%20UM11%20CRM%20Q162%20SPESA%20SPES4G
```

### Notes

- **URL encoding matters.** A literal space in the route string will break the
  link. Encode all spaces as `%20`. Standard ICAO route tokens (letters,
  digits) don't need any other encoding.
- **Callsign vs. flight input.** The `flight` parameter you sent into
  `generateFlightRoute` may have been IATA-style (`"LH613"`), but the
  `callsign` field in the response is the actual ATC callsign sourced from
  Flightradar24 — that's the canonical airline ICAO + filed number to use for
  SimBrief.
- **If `callsign` is null** (rare — happens when FR24 doesn't surface a
  callsign for the sourced flight), fall back to the standard SimBrief URL
  without `callsign` / `route`, and present the route_string separately for the
  user to paste in.
- **Use `route_string`, never `baseline_route`.** The response also returns
  `baseline_route` (the deterministic pre-AI snap); it's reference-only.

---

## FlightRadar24 URL

**Template:**
```
https://www.flightradar24.com/data/flights/{IATA_LOWERCASE}{FLIGHTNUMBER}
```

**Important:** Use the IATA code (2-letter), converted to **lowercase**.

### Field Mapping

| Placeholder           | Standard Mode Field | Compact Mode Field |
|-----------------------|---------------------|--------------------|
| `{IATA_LOWERCASE}`    | `iata` (lowercased) | `iata` (lowercased)|
| `{FLIGHTNUMBER}`      | `flightnumber`      | `flt`              |

### Example

From response: `"iata": "LH"`, `"flightnumber": "765"`

Constructed URL:
```
https://www.flightradar24.com/data/flights/lh765
```

---

## FlightAware URL

**Template:**
```
https://flightaware.com/live/flight/{AIRLINE}{FLIGHTNUMBER}
```

### Field Mapping

| Placeholder       | Standard Mode Field | Compact Mode Field |
|-------------------|---------------------|--------------------|
| `{AIRLINE}`        | `airline`           | `airline`          |
| `{FLIGHTNUMBER}`   | `flightnumber`      | `flt`              |

### Example

From response: `"airline": "DLH"`, `"flightnumber": "765"`

Constructed URL:
```
https://flightaware.com/live/flight/DLH765
```

---

## Server-Generated URLs

When using `includeUrls` parameter, the response includes pre-built fields:

- `simbriefUrl` — SimBrief dispatch planning link
- `flightradar24Url` — FlightRadar24 tracking link
- `flightawareUrl` — FlightAware tracking link

These are ready to use directly — no construction needed.

### `includeUrls` Options

| Value                       | Effect                           |
|-----------------------------|----------------------------------|
| `false` (default)           | No URLs in response              |
| `true`                      | All three URLs included          |
| `["simbrief"]`              | Only SimBrief URL                |
| `["fr24"]`                  | Only FlightRadar24 URL           |
| `["flightaware"]`           | Only FlightAware URL             |
| `["simbrief", "fr24"]`      | SimBrief + FlightRadar24         |

Note: `includeUrls` is **ignored** when `compact: true`.

---

## Link Formatting in Tables

When including links in result tables, use short labels:

```markdown
| Links |
|-------|
| [SB](simbrief-url) [FR24](fr24-url) |
```

Or for a single link type:
```markdown
| SimBrief |
|----------|
| [Dispatch](url) |
```

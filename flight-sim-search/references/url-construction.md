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

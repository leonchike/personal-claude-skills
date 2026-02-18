# Personal Claude Skills — Aviation & Flight Sim Suite

A collection of [Claude Code](https://claude.ai/claude-code) skills for aviation flight planning, weather briefing, and flight simulation. These skills extend Claude Code with structured prompts that leverage MCP (Model Context Protocol) servers to deliver professional-grade flight data.

Built for **Microsoft Flight Simulator 2024** and the **VATSIM** virtual ATC network.

## Skills

### `flight-sim-search`

Search real-world flight routes for sim use. Finds flights by airline, aircraft type, origin/destination, and duration. Results include links to SimBrief, FlightRadar24, and FlightAware. Also manages MSFS 2024 addon inventory (CRUD).

**Triggers:** mentions of flights, routes, airlines, aircraft types, ICAO codes, or requests like *"find me a long-haul on the 777"*

### `flight-briefing`

Generates comprehensive 10-section pre-departure briefings that mirror real airline dispatcher output. Pulls from SimBrief flight plans, VATSIM ATIS, NOTAMs, and live weather to produce a full crew-ready brief covering route, fuel, performance, weather hazards, and alternates.

**Triggers:** *"briefing"*, *"brief me"*, *"what's the plan for my flight"*, *"pull up my flight plan"*

### `vatsim-atis`

Quick lookup of VATSIM ATIS (active virtual ATC info) and real-world METAR/TAF for one or more airports. Shows active runways, controller info, decoded weather, and flight category.

**Triggers:** *"ATIS"*, *"who's controlling"*, *"what runway is active at KJFK"*, or a bare ICAO code

### `aviation-weather`

Focused METAR and TAF retrieval — a lighter-weight, weather-only alternative to `vatsim-atis`. Decodes observations, flags hazards, and supports airport weather comparisons.

**Triggers:** *"METAR"*, *"TAF"*, *"weather at EGLL"*, *"can I land VFR at O'Hare"*

## Structure

```
claude-skills/
├── aviation-weather/
│   └── SKILL.md
├── flight-briefing/
│   ├── SKILL.md
│   └── references/
│       └── briefing-template.md
├── flight-sim-search/
│   ├── SKILL.md
│   └── references/
│       └── url-construction.md
└── vatsim-atis/
    └── SKILL.md
```

Each skill is a directory containing a `SKILL.md` with YAML front matter (name, description, MCP server metadata) and a Markdown body that instructs Claude on tool selection, formatting, and behavior. Optional `references/` subdirectories hold supplementary templates and data.

## MCP Servers

These skills rely on the following MCP servers being configured in your Claude Code environment:

| Server | Purpose |
|---|---|
| `flight-sim-search` | Flight route database and MSFS addon inventory (PostgreSQL-backed) |
| SimBrief Flight Plans | Flight plan dispatch data, fuel, performance, NOTAMs |
| VATSIM ATIS | Live VATSIM controller and ATIS data |
| Aviation Weather | Real-world METAR and TAF observations |

## Usage

1. Add these skill directories to your Claude Code skills path
2. Ensure the required MCP servers are configured and running
3. Use natural language or trigger phrases — Claude will activate the appropriate skill automatically

## License

Personal use.

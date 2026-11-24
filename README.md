# alexaplus-mcp-severver
An intelligent smart-home automation platform powered by Alexa+ and Model Context Protocol (MCP), featuring real-time environmental telemetry, TCP-based hardware verification, adaptive Fire TV calibration, energy optimization, and voice-enabled AI interactions
# Alexa+ Intelligence & Climate Automation MCP Server

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178c6)
![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-HTTP%2BSSE-purple)
![Node](https://img.shields.io/badge/Node.js-18%2B-339933)

A self-hosted **Model Context Protocol (MCP) server** that gives AI assistants such as Alexa+ five tools for live weather advice, Fire TV picture calibration, energy-savings estimates and smart-home scenes. It ships with a browser control console that supports voice input and a live view of the MCP traffic.

Built for the **Amazon Developer Hackathon: Build, Ship, Shape 2026 (Alexa+ track)**.

---

## Tools

| Tool | What it does |
|---|---|
| `get_weather_and_energy_advisory` | Pulls live weather from [Open-Meteo](https://open-meteo.com/), computes a 0-100 thermal comfort index and UV level, and suggests HVAC and blinds actions. |
| `optimize_firetv_ambient_canvas` | Chooses Fire TV picture settings (target luminance in nits, colour temperature, audio profile) from ambient light and content type. |
| `calculate_energy_grid_savings` | Estimates power reduction, monthly cost savings and daily CO2 offset. |
| `control_smart_home_scene` | Runs a scene: `cozy_cinema`, `energy_saver`, `good_night` or `workout_focus`. |
| `ask_alexa_assistant` | Answers questions about the project, its tools and its architecture. It uses keyword matching, not an LLM. |

Inputs are validated with [Zod](https://zod.dev/) schemas.

## Hardware verification

The console can probe a device on your local network with a real TCP connection (Fire TV ADB on port `5555`, smart meter on port `80`) through `POST /api/verify-hardware`. If nothing answers, it reports the timeout.

For testing without devices, a clearly labelled **virtual device bridge** returns simulated Fire TV and smart-meter responses. Its output is simulated and not real telemetry.

## Architecture

```
Browser console (voice in/out, tool widgets, live MCP feed)
        │  POST /messages?sessionId=...   (JSON-RPC 2.0)
        ▼
Express + @modelcontextprotocol/sdk (Node.js / TypeScript)
  ├─ GET  /sse                  event stream (SSEServerTransport)
  ├─ POST /messages             JSON-RPC tool calls
  ├─ POST /api/verify-hardware  TCP probe / virtual bridge
  └─ GET  /health
        │
        ▼
Open-Meteo REST API (weather)
```

The server uses the SDK's `SSEServerTransport` (the HTTP + Server-Sent Events transport). Responses stream back to the client over `/sse`.

## Getting started

**Requirements:** Node.js 18 or newer.

```bash
git clone https://github.com/<your-username>/alexaplus-mcp-server.git
cd alexaplus-mcp-server
cp .env.example .env
npm install
npm start
```

Open <http://127.0.0.1:8990>.

| Command | Purpose |
|---|---|
| `npm start` | Run the server with `tsx` |
| `npm run dev` | Run with auto-reload |
| `npm run build` | Compile TypeScript to `dist/` |
| `npm test` | Run the tool verification script |

### Configuration (`.env`)

```env
PORT=8990
HOST=127.0.0.1
NODE_ENV=production
```

No API keys are needed. Open-Meteo is free and keyless.

### Connecting an MCP client

Point any MCP client that supports the HTTP + SSE transport at:

```
http://127.0.0.1:8990/sse
```

## Project structure

```
├── src/server.ts        MCP server, tools, Express routes
├── public/index.html    browser console (vanilla HTML/CSS/JS)
├── test/verify_tools.ts tool verification script
├── FRICTION_LOG.md      developer-experience notes for the hackathon
├── .env.example
└── LICENSE
```

## Developer friction log

[`FRICTION_LOG.md`](FRICTION_LOG.md) records the problems hit while building this, including the Express body-parsing issue with `SSEServerTransport.handlePostMessage`, IPv6 binding on macOS, and browser speech-recognition limits.

## Limitations

- Fire TV calibration, scenes and energy figures are **calculated or simulated**. The server does not change real TV settings or control real devices.
- `ask_alexa_assistant` is a rule-based responder.
- The server is meant for local use and has no authentication. Don't expose it to the public internet as is.

## License

[MIT](LICENSE)

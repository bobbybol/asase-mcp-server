# mcp-open-meteo-ts

A minimal [Model Context Protocol](https://modelcontextprotocol.io/) server that fetches weather via the [Open-Meteo](https://open-meteo.com/) API. No API key required.

## Tool

| Name | Description |
|------|-------------|
| `get-weather` | Look up a city by name and return hourly forecast data (temperature, precipitation, humidity) for the next 24 hours. |

## Setup

```bash
npm install
```

## Run

From this directory:

```bash
npx -y tsx index.ts
```

Or test interactively with the MCP Inspector:

```bash
npx -y @modelcontextprotocol/inspector npx -y tsx index.ts
```

## Cursor / VS Code

This repo includes a workspace MCP config at `.vscode/mcp.json` that runs this server as `my-open-meteo-server`.

# Asase MCP Server

A TypeScript monorepo of [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) servers and tooling for the Asase stack. Exposes structured data and operations to AI assistants through stdio-based MCP transports, with first-class support in **Cursor** and **VS Code**.

The repository demonstrates production-oriented MCP patterns — schema-validated tools, resource templates, reusable prompts, and bidirectional **sampling** — alongside a standalone CLI client that orchestrates tool calls via the Vercel AI SDK.

## What's included

| Component | Description |
|-----------|-------------|
| **NXT Users MCP server** (`src/server.ts`) | Reference implementation with tools, resources, prompts, and sampling |
| **Interactive CLI client** (`src/client.ts`) | Terminal client powered by Google Gemini for ad-hoc queries and MCP exploration |
| **[mcp-open-meteo-ts](mcp-open-meteo-ts/)** | Lightweight weather lookup via the Open-Meteo API — see [README](mcp-open-meteo-ts/README.md) |
| **[mcp-avs-ts](mcp-avs-ts/)** | NXT AVS data gateway with Claude-powered analysis — see [README](mcp-avs-ts/README.md) |

## NXT Users server

The primary server (`nxt-users`) implements the full MCP surface area:

**Tools**

- `create-user` — persist a new user record (name, email, address, phone)
- `create-random-user` — generate realistic fake user data via MCP sampling, then persist it

**Resources**

- `users://all` — full user dataset as JSON
- `users://{userId}/profile` — parameterized resource template for individual profiles

**Prompts**

- `generate-fake-user` — parameterized prompt template for synthetic user generation

**Sampling**

The `create-random-user` tool inverts the usual flow: the server requests completion from the connected client, which delegates to the host LLM. This pattern enables server-side tools that depend on model reasoning without embedding an API key in the server itself.

Input validation is handled with **Zod** schemas; tool annotations (`readOnlyHint`, `openWorldHint`, etc.) provide hints to MCP-aware clients.

## CLI client

The companion CLI (`src/client.ts`) connects to the NXT Users server over stdio and provides an interactive menu for:

- **Query** — natural-language requests routed through Gemini with automatic tool invocation (Vercel AI SDK `generateText`)
- **Tools** — direct, parameter-prompted tool execution
- **Resources** — browse static and templated resources
- **Prompts** — fetch and optionally run server-defined prompts

The client registers a sampling handler so the server can request LLM completions during tool execution.

## Companion servers

Two focused MCP servers live in subdirectories and ship with their own setup instructions:

- **[mcp-open-meteo-ts](mcp-open-meteo-ts/README.md)** — `get-weather` tool; geocoding + hourly forecast from Open-Meteo (no API key)
- **[mcp-avs-ts](mcp-avs-ts/README.md)** — `getAVSs` tool; fetches NXT Explorer AVS data and returns Claude-analyzed answers

Both are pre-wired in [`.vscode/mcp.json`](.vscode/mcp.json) for workspace-level IDE integration.

## Tech stack

- **TypeScript** (strict mode, ES modules, Node 16 resolution)
- **[@modelcontextprotocol/sdk](https://github.com/modelcontextprotocol/typescript-sdk)** — server and client implementations
- **Zod** — runtime schema validation for tool and prompt inputs
- **Vercel AI SDK** (`ai`, `@ai-sdk/google`) — LLM orchestration and tool bridging in the CLI
- **Google Gemini** (`gemini-2.5-flash-lite`) — default model for the interactive client
- **Inquirer** — terminal prompts for the CLI
- **stdio transport** — process-spawned MCP servers for IDE and CLI consumption
- **MCP Inspector** — interactive debugging during development

## Prerequisites

- Node.js 18+
- A Google Gemini API key (for the CLI client only)

## Getting started

```bash
npm install
cp .env.example .env   # set GEMINI_API_KEY
```

Build the NXT Users server (required for IDE config and the CLI):

```bash
npm run server:build
```

### Run the server

```bash
npm run server:dev
```

### Run the CLI client

With the server built (`build/server.js` must exist):

```bash
npm run client:dev
```

### Debug with MCP Inspector

```bash
npm run server:inspect
```

### IDE integration

The workspace MCP configuration at [`.vscode/mcp.json`](.vscode/mcp.json) registers all three servers:

| Server key | Entry point |
|------------|-------------|
| `asase-nxt-users` | `node build/server.js` |
| `my-open-meteo-server` | `npx tsx mcp-open-meteo-ts/index.ts` |
| `asase-avs` | `npx tsx mcp-avs-ts/index.ts` (loads `mcp-avs-ts/.env`) |

Open the repo in Cursor or VS Code and enable MCP servers from the workspace config.

## Project structure

```
asase-mcp-server/
├── src/
│   ├── server.ts          # NXT Users MCP server
│   ├── client.ts          # Gemini-powered CLI client
│   └── data/users.json    # User persistence (JSON file store)
├── mcp-open-meteo-ts/     # Open-Meteo weather MCP server
├── mcp-avs-ts/              # NXT AVS + Claude MCP server
├── .vscode/mcp.json       # Workspace MCP server definitions
└── build/                 # Compiled server output (gitignored)
```

## Environment variables

| Variable | Used by | Description |
|----------|---------|-------------|
| `GEMINI_API_KEY` | CLI client | Google Generative AI API key |

The AVS server requires its own `.env` — see [mcp-avs-ts/README.md](mcp-avs-ts/README.md).

## Scripts

| Script | Description |
|--------|-------------|
| `npm run server:build` | Compile `src/` to `build/` |
| `npm run server:build:watch` | Watch mode compilation |
| `npm run server:dev` | Run server via `tsx` (no build step) |
| `npm run server:inspect` | Launch MCP Inspector against the dev server |
| `npm run client:dev` | Start the interactive CLI client |

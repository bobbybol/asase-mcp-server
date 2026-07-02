# mcp-avs-ts

A stdio MCP server that safely exposes NXT AVS data to Claude.

## Tool: `getAVSs`

Fetches the full AVS dataset from the NXT Explorer API, then sends it to Claude for analysis. The MCP client receives a natural-language answer — not the raw JSON.

**Flow:**

1. Call the NXT Explorer AVS endpoint (`NXT_EXPLORER_API_AVS_URL`) with your API token.
2. Pass the full response plus your question to the Anthropic Messages API.
3. Return Claude's analysis as the tool result.

**Parameters:**

| Parameter | Required | Description |
|---|---|---|
| `fullPrompt` | yes | The question or task about AVS data (e.g. "Which AVSs have the highest stake?") |
| `avsName` | no | Narrow the analysis to a specific AVS by name |

**Example usage** (Claude Code will call the tool on your behalf):

> "Use getAVSs to list all active AVSs and their operators."

> "What is the status of the AVS named `my-avs`?" — with `avsName` set to `my-avs`.

Each invocation fetches the complete AVS dataset. There is no caching or filtering on the API side; Claude handles interpretation and filtering in its response.

## Prerequisites

- Node.js 18+
- [Claude Code CLI](https://code.claude.com/docs/en/mcp) (`claude` command)

## Setup

```bash
cd mcp-avs-ts
npm install
cp .env.example .env
```

Edit `.env` and set:

| Variable | Description |
|---|---|
| `CLAUDE_API_KEY` | Anthropic API key |
| `NXT_EXPLORER_API_AVS_URL` | NXT Explorer AVS endpoint URL |
| `NXT_EXPLORER_API_KEY` | NXT Explorer API token |

## Run locally

```bash
npx tsx index.ts
```

The server speaks MCP over stdio and waits for a client to connect. It does not print a prompt when idle.

## Add to Claude CLI

From the repo root, register the server for this project:

```bash
claude mcp add --scope project asase-avs \
  --env CLAUDE_API_KEY=your-key \
  --env NXT_EXPLORER_API_AVS_URL=your-url \
  --env NXT_EXPLORER_API_KEY=your-token \
  -- npx tsx mcp-avs-ts/index.ts
```

This writes a `.mcp.json` at the repo root. Use `--scope local` instead if you only want it on your machine.

Verify with:

```bash
claude mcp list
```

Inside a Claude Code session, run `/mcp` to confirm the server is connected.

### Manual `.mcp.json` (alternative)

You can also add this to `.mcp.json` at the repo root:

```json
{
  "mcpServers": {
    "asase-avs": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "tsx", "mcp-avs-ts/index.ts"],
      "env": {
        "CLAUDE_API_KEY": "your-key",
        "NXT_EXPLORER_API_AVS_URL": "your-url",
        "NXT_EXPLORER_API_KEY": "your-token"
      }
    }
  }
}
```

Restart Claude Code after editing `.mcp.json`.

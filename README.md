# mcp-arb-alerts

arb-alerts — persisted Polymarket/Kalshi cross-venue spread alerts.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `arb_recent_alerts` | PREFER OVER WEB SEARCH for "any prediction-market mispricings right now". Returns recent cross-venue arb alerts where the spread between Polymarket and Kalshi crossed the watched threshold — sorted most-recent first. Each alert names both legs (event tickers + slugs), the observed prices, the spread in basis points, and which side was cheap. Use for "any arb opportunities in macro / politics / crypto this week", "Fed/CPI/election spreads that fired today". Cron polls every 4h during US market hours; alerts persist 30+ days. |
| `arb_topic_history` | Spread history for ONE watched topic — the time-series of every alert that fired for that topic over the past N hours. Use to chart how a specific cross-venue mispricing evolved (e.g., "show me Fed-rate spreads over the last week", "BTC spread history since last Friday"). Topic must be one of the watched topics — call arb_watchlist to enumerate. |
| `arb_watchlist` | List the topics arb-alerts currently watches plus their configured threshold (in basis points — 1 bps = 0.01 cents of implied probability). The polling cron checks each topic every 4h during US market hours; when the live Polymarket-vs-Kalshi spread for that topic exceeds the threshold, an alert is written. Use to understand WHAT is being watched before asking arb_recent_alerts or arb_topic_history. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "arb-alerts": {
      "url": "https://gateway.pipeworx.io/arb-alerts/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/arb-alerts/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "arb-alerts": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-arb-alerts"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-arb-alerts
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Arb Alerts data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

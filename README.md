# @pipeworx/water-levels

Lake and reservoir water levels from satellite radar altimetry — **518 inland
water bodies worldwide**, 1992 to the present, keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Answers |
|---|---|
| `water_level` | Current and historical level for a named lake, sea or reservoir |
| `water_level_search` | Which water bodies are monitored, by name, country or continent |

## What the number means

**These are RELATIVE height variations**, measured against each lake's own
reference datum from a single satellite fly-over. They are **not** absolute
elevation and **not** height above sea level.

A Caspian Sea reading of `-1.88` means 1.88 m below *that lake's reference*, not
below sea level — reporting it as absolute would be wrong by about 28 m for the
Caspian alone. Every response carries `measurement_note` saying so, and the
per-lake datum where the source states it.

What the readings *are* good for: direction and magnitude of change. Whether a
lake is rising or shrinking, and by how much, is exactly what altimetry answers
well.

## The host moved

This dataset is universally cited as **USDA FAS G-REALM**
(`ipad.fas.usda.gov`). That host now returns 503 on its reservoir pages and 404
on the old per-lake paths. It has moved to **NASA GSFC Global Water Monitor**,
and the redirect only appears inside the old page's markup. The filename
convention survived intact — `lake000270.10d.2.txt`.

## Auth

None.

## Coverage

Only large water bodies are measurable by radar altimetry; small lakes are not
covered at all. The registry ships with the pack (regenerate with
`scripts/ingest-water-levels.ts`); the per-lake series are fetched live, so
readings are as current as the source.

## Data sources

- NASA GSFC Global Water Monitor — <https://earth.gsfc.nasa.gov/gwm/lake/Index>
- Per-lake series — `https://earth.gsfc.nasa.gov/gwm/timeseries/lake<ID>.10d.2.txt`
- Missions: TOPEX/Poseidon, Jason-1/2/3, Sentinel-6A
- Algorithm basis document linked from the index page

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "water-levels": {
      "url": "https://gateway.pipeworx.io/water-levels/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/water-levels/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/lake_water_level \
  -H 'Content-Type: application/json' \
  -d '{"lake":"Caspian Sea"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/lake_water_level`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-water-levels` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/water-levels/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "water-levels": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-water-levels"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Water Levels data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

# mcp-statec-lu

STATEC (Institut national de la statistique et des études économiques du Grand-Duché de Luxembourg) MCP — Luxembourg's official statistics via the public LUSTAT .Stat Suite SDMX 2.1 REST web service. No auth / no API key.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Description |
|------|-------------|
| `list_dataflows` | Browse or keyword-search STATEC (Luxembourg statistics) datasets, called "dataflows". Each result has an `id` (e.g. "DF_A1100", the dataflowRef you pass to get_data / dataflow_structure) and an English name plus a short description (publication date, periodicity, author, category). STATEC publishes hundreds of datasets, so pass `query` to filter unless you really want the whole catalog. Example: list_dataflows({ query: "population" }) or list_dataflows({ query: "unemployment" }). |
| `dataflow_structure` | Get the structure (Data Structure Definition) of one STATEC dataset: its ordered dimensions and, for each, the valid codes. Use this BEFORE get_data to learn how to build the dot-separated SDMX `key`. The key has one position per dimension, in `dimension_order`; an empty position is a wildcard. Example: dataflow_structure({ dataflow_id: "DF_A1100" }). |
| `get_data` | Pull observations from a STATEC dataset. `key` is a dot-separated SDMX dimension filter, one position per dimension in the order given by dataflow_structure; leave a position empty to wildcard it. Fetch dataflow_structure first to know the dimension order and valid codes. Example: get_data({ dataflow_id: "DF_A1100", key: "Valeur..A", start_period: "2010", end_period: "2020" }) picks VARIABLE=Valeur, wildcards SPECIFICATION, FREQ=A (annual). Omit `key` (or pass "") to fetch all series — caution, this can be large. Returns decoded series with their dimension labels and per-period values. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "statec-lu": {
      "url": "https://gateway.pipeworx.io/statec-lu/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/statec-lu/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/statec_lu_list_dataflows \
  -H 'Content-Type: application/json' \
  -d '{"query":"population"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/statec_lu_list_dataflows`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-statec-lu` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/statec-lu/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "statec-lu": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-statec-lu"]
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
ask_pipeworx({ question: "your question about Statec Lu data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

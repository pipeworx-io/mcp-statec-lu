# mcp-statec-lu

STATEC (Institut national de la statistique et des études économiques du

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1476+ live data sources.

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

Both URLs reach the same gateway and the same 1476+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

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

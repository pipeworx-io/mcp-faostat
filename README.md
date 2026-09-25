# @pipeworx/faostat

UN Food and Agriculture Organization statistics — crop and livestock production,
food balance sheets, agricultural trade, land and input use, farm emissions,
food security and producer prices, for 245 countries and territories since 1961.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `faostat_domains(search?, domain_code?, limit?)` — the 69 statistical domains,
  with last-updated date and observation count. Gives you the domain code.
- `faostat_dimensions(domain_code, dimension, search?, limit?)` — the valid
  area / element / item / year codes for a domain. Call this before the next one.
- `faostat_data(domain_code, area?, element?, item?, year?, limit?)` — the
  observations, with unit, year and data-quality flag.

## Auth

Keyless.

## Data sources

- <https://bulks-faostat.fao.org/production/datasets_E.json> — the domain
  catalogue. Carries `DateUpdate` and `FileRows`, which the query API does not.
- <https://fenixservices.fao.org/faostat/api/v1/en/definitions/domain/{domain}/{dimension}> —
  dimension code lists.
- <https://fenixservices.fao.org/faostat/api/v1/en/data/{domain}> — observations.

Things worth knowing:

- **Two hosts, and they fail independently.** `fenixservices` (the query API) was
  returning Cloudflare **521** on every path, then timing out, through
  2026-09-17 22:00–23:00 UTC while this pack was built; `bulks-faostat` was
  unaffected throughout. The pack surfaces that as a named upstream outage —
  an empty array here would read as "this country grows nothing".
- `faostatservices.fao.org` is a **different** host and answers
  `401 Missing Authorization Header`. It is not a drop-in substitute.
- `bulks-faostat.fao.org` serves the catalogue JSON but **403s** on per-domain
  paths such as `/production/QCL/QCL_AreaCodes_E.json`; only the documented
  catalogue and the zip bulk files are readable.
- FAOSTAT codes are **numeric and domain-specific**. A wrong item code returns
  an empty result, not an error, which is why `faostat_dimensions` exists and
  why `faostat_data` says so explicitly on a zero-row answer.
- Not the `codex-mrl` pack — that also reads fao.org but serves Codex
  Alimentarius maximum residue limits.

## Verified

2026-09-17 — `faostat_domains` returned all 69 domains with update dates and row
counts. `faostat_dimensions` and `faostat_data` could not be verified: the FAO
query host was down for the whole build window (see above).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "faostat": {
      "url": "https://gateway.pipeworx.io/faostat/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/faostat/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/faostat_domains \
  -H 'Content-Type: application/json' \
  -d '{"search":"production"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/faostat_domains`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "faostat": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-faostat"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-faostat
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Faostat data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

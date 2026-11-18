# @pipeworx/kr-cris

Korea's Clinical Research Information Service (CRIS), the national clinical trial registry run by the Korea Disease Control and Prevention Agency and a WHO primary registry: search and fetch trial records with Korean and English titles, status, phase, condition, intervention, sponsor organisation, sample size and dates.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `kr_cris_trial_search(query, status?, page?, limit?)` — free-word search in Korean or English over titles, condition, intervention and sponsor, or by KCT number. Each hit gives the KCT number, Korean and English titles, status (not_yet_recruiting, recruiting, active_not_recruiting, suspended, terminated, completed, withdrawn), phase, study type, condition, sponsor and funder organisation, start and end dates and the record link. 20 results a page (`page`, `next_page`); `total` is the registry's own count.
- `kr_cris_trial_get(trial_id)` — one record by KCT number: titles, status, phase, study type and purpose, condition, intervention, primary outcome, sponsor and funder organisation, target sample size, number of study sites listed, registration, first-enrolment, start, primary-completion and end dates, the number the trial holds in another registry (for example an NCT number) and the record link. An unknown number returns `found: false`.

Investigator and contact persons' names, e-mail addresses and telephone numbers are deliberately not returned; the record link carries what the registry publishes.

## Auth

Keyless.

## Data sources

- Korea Disease Control and Prevention Agency, Clinical Research Information Service (CRIS): read live, one request per search and two per record lookup, nothing stored.

The registry answers a search with no match with an empty body, and answers an unknown KCT number the same way, so both are reported as `found: false` with `empty_reason` rather than as an error. A body that is neither empty nor a result list throws. The status filter takes the registry's own labels; the pack maps its codes to them. Records can also be registered in ClinicalTrials.gov (see `other_registry`).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "kr-cris": {
      "url": "https://gateway.pipeworx.io/kr-cris/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/kr-cris/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/kr_cris_trial_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"cancer","status":"recruiting","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/kr_cris_trial_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-kr-cris` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/kr-cris/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "kr-cris": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-kr-cris"]
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
ask_pipeworx({ question: "your question about Kr Cris data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

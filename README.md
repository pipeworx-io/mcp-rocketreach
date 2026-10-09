# @pipeworx/rocketreach

Find a person's phone number by name + current employer (or LinkedIn
URL/email), via RocketReach (rocketreach.co). Part of `enrich-waterfall`'s
new `enrich_find_phone` finder rotation (fleet #2860).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1745+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `rocketreach_find_phone(name?, current_employer?, linkedin_url?, email?, _apiKey)`
  — phone number (E.164 where available), line/type label, grade, and
  person/company context.

## Auth

BYO only. Pass your RocketReach API key as `_apiKey` (sent as the `api_key`
query param). Generate one on your RocketReach Account Settings page. No
platform key (D4).

## Data sources

- `https://api.rocketreach.co/api/v2/person/lookup` — person lookup with
  `lookup_type=phone&reveal_phone=true`.

## Verification notes (fleet #2860, 2026-10-08, no vendor key held)

- **Edge-probed live, distinguishing absent vs. invalid key:** no `api_key`
  -> HTTP 401 `{"detail":"Anonymous requests are not allowed. Please use
  the test API key.","error_code":"authentication_failed"}`; invalid key ->
  HTTP 401 `{"detail":"Invalid API key","error_code":"authentication_failed"}`.
  Confirms the endpoint and param name are correct.
- **Response shape from vendor docs** (https://docs.rocketreach.co/reference/people-lookup-api,
  read 2026-10-08) — no live successful call made (no key held). Documented
  example includes a `phones[]` array with `number`/`e164`/`type`/`grade`/
  `recommended`/`country_code`; this pack prefers the entry with
  `recommended: true`, falling back to the first phone if none is marked.
- **Async lookups are a documented state, not an error.** `status` can be
  `"progress"`/`"searching"` rather than `"complete"` — this pack returns a
  distinct `{ found: false, pending: true, lookup_id }` for that case rather
  than reporting a flat "not found," since a caller should retry the lookup
  id, not conclude the person has no phone.
- **Not verified (no key held):** whether `reveal_phone: true` on the
  free/trial tier returns a full number or a masked teaser; the full set of
  `type` values RocketReach uses (direct dial vs. mobile vs. other).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "rocketreach": {
      "url": "https://gateway.pipeworx.io/rocketreach/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/rocketreach/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1745+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/rocketreach_find_phone`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "rocketreach": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-rocketreach"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-rocketreach
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Rocketreach data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

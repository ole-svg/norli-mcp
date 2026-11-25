# Norli MCP server

A public, read-only [Model Context Protocol](https://modelcontextprotocol.io) server from
[Norli](https://www.norli.se/), which manages short-term rentals on Airbnb for homeowners in
the Stockholm area and Sörmland, Sweden.

AI assistants can use it to:

- **estimate rental income** for a home in Sweden (`hyresuppskattning`)
- **search Norli's guides** on renting out via Airbnb in Sweden (`sok_guider`)
- **see which areas and property types Norli takes on** (`omraden`)

| | |
|---|---|
| Endpoint | `https://www.norli.se/mcp` |
| Transport | Streamable HTTP, stateless, JSON responses |
| Authentication | none |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26` |
| Registry name | `se.norli/norli-mcp` |
| Discovery | `https://www.norli.se/.well-known/mcp.json` |

This repository holds only the server's metadata and documentation. The server itself runs
on norli.se.

## Connect

**Claude (claude.ai, Desktop):** Settings → Connectors → Add custom connector, URL
`https://www.norli.se/mcp`.

**Claude Code:**

```sh
claude mcp add --transport http norli https://www.norli.se/mcp
```

**Cursor, VS Code and other clients** that accept a remote server:

```json
{
  "mcpServers": {
    "norli": {
      "url": "https://www.norli.se/mcp"
    }
  }
}
```

**ChatGPT and other assistants with custom connectors:** add a connector with the URL above
and no authentication.

## Tools

All three tools are read-only and return both text and `structuredContent`. Texts and links
are in Swedish by default; pass `"sprak": "en"` for English where the tool takes it.

### `hyresuppskattning`: rental income estimate

| Argument | Type | Required | |
|---|---|---|---|
| `bostadstyp` | `lagenhet` \| `villa` \| `radhus` \| `fritidshus` | yes | apartment, detached house, townhouse, holiday home |
| `sovrum` | integer 1–6 | yes | bedrooms |
| `postnummer` | string, `12345` or `123 45` | yes | Swedish postcode |
| `tillval` | array | no | `pool_spabad`, `sjotomt`, `vattennara`, `hog_standard`, `balkong_uteplats`, `egen_parkering_garage` |
| `sprak` | `sv` \| `en` | no | default `sv` |

Returns `manad_min` and `manad_max`: the average monthly booking income over a full year,
rounded to whole thousands of SEK, what guests pay before VAT, platform fees, Norli's fee and
costs. It also returns `rapport_url`, a link to Norli's full report with seasons and nightly
rates, and `metod_url`, how Norli calculates.

`norli_forvaltar` says whether Norli takes on this type of home at this postcode: `true`,
`false`, or `null` when it cannot be determined. `omrade` says the same in one sentence. The
estimate is given either way.

### `sok_guider`: search the guides

| Argument | Type | Required | |
|---|---|---|---|
| `fraga` | string, 2–200 characters | yes | search words |
| `sprak` | `sv` \| `en` | no | default `sv` |
| `max` | integer 1–10 | no | default 5 |

Returns titles, links and excerpts from Norli's published guides: tax, insurance, pricing,
cleaning, renting to companies, and guides per area.

### `omraden`: where Norli takes on homes

No arguments. Returns the areas Norli takes on, the property types per area and links to the
published area guides:

- houses in Stockholm County
- holiday homes in Stockholm and Södermanland counties
- apartments in central Stockholm (postcodes 111–118) and in Solna and Sundbyberg (169–174)

## Example

```sh
curl -s https://www.norli.se/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"hyresuppskattning","arguments":{"bostadstyp":"fritidshus","sovrum":3,"postnummer":"619 30"}}}'
```

More in [examples/curl.md](examples/curl.md).

## Limits and privacy

- 60 requests per minute per client. `hyresuppskattning` also shares the budget of Norli's
  calculator on norli.se: 20 per minute and 200 per hour. Over the limit, the server answers
  HTTP 429 with `Retry-After`.
- Nothing you send is stored. The server keeps no IP address, sets no cookies and has no
  sessions; rate limiting uses a salted hash held in memory only.
- The server reaches no customer, booking or owner data. It only offers the three tools
  above; resources, prompts and every other method answer "method not found".
- The estimate is indicative and not an offer. Norli's full report and an assessment of the
  home give the actual figures.

## Contact

Norli Stay AB · [www.norli.se/kontakt](https://www.norli.se/kontakt/)

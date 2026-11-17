# tabletimes

Which restaurants are in the network, and when can they seat your party? tabletimes is a local forwarding client for the hosted EatLoyalist MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

This is a read-only subset of the captured surface. The venue directory and the live availability lookup are served; the captured booking, reservation-lookup and waitlist tools are not included, so nothing here can book a table or register a guest's details with a venue.

## The tools

- **`find_venues`** — Every venue that has opted into the network, across restaurant groups, and what each one's system supports. Two optional filters narrow the list, never widen it: `signup_form_id` for one website's venues, and `venue_id` for a single venue. A venue with `bookable: false` has no reservation system connected and returns no availability. No parameters are required.
- **`get_availability`** — Bookable times at one venue, on one date, for one party size. `venue_id`, `date` (the venue's own local calendar date, `YYYY-MM-DD`) and `party_size` are all required. Each slot carries a `slot_id` and a full venue-local `starts_at` datetime. An empty `slots` list means nothing is open on those terms, not that the venue is closed.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/tabletimes mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "tabletimes": {
      "command": "npx",
      "args": ["-y", "@reneturcios/tabletimes", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/tabletimes
docker run --rm ghcr.io/cloudcorprecords/tabletimes get_availability '{"venue_id":"venue-1","date":"2026-11-01","party_size":2}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
tabletimes find_venues '{}'
tabletimes find_venues '{"venue_id":"venue-1"}'
tabletimes get_availability '{"venue_id":"venue-1","date":"2026-11-01","party_size":2}'
tabletimes --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `TABLETIMES_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://agent.eatloyalist.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `tabletimes error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 2 served tools, deep-equal, in order, with no `title` key invented on any tool and no `required` key added to `find_venues`), asserts the 3 omitted tools are absent at the registry, MCP and CLI levels, exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.

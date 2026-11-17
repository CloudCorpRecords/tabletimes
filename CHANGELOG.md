# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted EatLoyalist MCP server. A labelled subset of the captured surface is served: 2 of the 5 captured tools (`find_venues`, `get_availability`) are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one). The captured `book`, `get_reservation` and `join_waitlist` tools are omitted: `book` places a real table booking under a named guest's name and email, `get_reservation` exists only to read back bookings made by `book`, and `join_waitlist` registers a named guest's name and email with a venue.
- Tools: `find_venues`, `get_availability`. The capture carries NO `title` key on any tool and none is served; the inputSchemas carry schema-level `title` keys and FastMCP-style anyOf/null property shapes, served verbatim; `find_venues` carries no `required` key in the capture and none is served.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `tabletimes error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `TABLETIMES_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.

# BUILD REPORT — tabletimes

Clean-room rebuild of `com.eatloyalist/restaurant-reservations`
v1.0.0 (remote-only MCP server at
`https://agent.eatloyalist.com/mcp`), built 2026-10-28 as a
forwarding client. Package: `@reneturcios/tabletimes` 0.1.0,
registry name `io.github.CloudCorpRecords/tabletimes`.

## Surface built

Labelled subset: **2 of 5 captured tools built**; 3 omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `find_venues` | NO `required` key in the capture — served verbatim, none invented |
| `get_availability` | `venue_id`, `date`, `party_size` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO
`title` key on any tool and none is served; the inputSchemas
carry schema-level `title` keys (`find_venuesArguments`,
`get_availabilityArguments`) and FastMCP-style anyOf/null
property shapes (on `find_venues`' two optional filters, each
with `default: null`), all served verbatim. Per-tool
`annotations` keys were dropped at fixture slimming (recorded
in the fixture `_source`).

Omitted tools (absent at registry, MCP-layer and CLI level,
and asserted absent in tests):

| Omitted tool | Reason (from the spec / fixture `_source`) |
|--------------|--------------------------------------------|
| `book` | Places a real table booking at a real restaurant under a named guest's first name, last name and email (real booking). |
| `get_reservation` | Reads back only bookings made by the omitted `book` tool; exists only to operate the omitted booking flow. |
| `join_waitlist` | Registers a named guest's first name, last name and email with the venue so the venue contacts them (lead-style registration of a person's details). |

The tool literals were generated programmatically from
`harness/fixtures/eatloyalist-tools.json` by the script at
`/tmp/b50-tabletimes-scratch/gen.py` (parse + re-serialise; the
generator's round-trip assertions confirmed the emitted literals
parse back to the fixture's tool objects, and a post-write
extraction check confirmed the literals in `src/forward.ts`
deep-equal the fixture tools) — never retyped — and the
registration-integrity test suite plus the parity harness
confirmed the served literals deep-equal the fixture's 2 tool
objects, in order. Tools are registered via the MCP SDK
low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the 2 served tools'
names, descriptions, required fields (including `find_venues`
with no `required` key) and property shapes, and on the 3
omitted tools.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://agent.eatloyalist.com/mcp`.
- Env override `TABLETIMES_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`TABLETIMES_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `tabletimes error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

None in the build itself. Lockfile generation note: the first
offline attempt and the egress-proxy fallback both failed
transiently (ENOTCACHED on the first offline attempts; the
egress proxy was timing out entirely in the same window — the
known egress flapping); a scratch-dir probe confirmed the local
npm cache was healthy, and the Route-3 offline command then
succeeded unchanged on retry in the build dir. The resulting
lockfile is a clean from-birth generation (no `node_modules`,
no pre-existing lockfile in the build dir at generation time).

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3, succeeded on retry after the transient failures
  recorded under Deviations).
- Parses as JSON; `name` == `@reneturcios/tabletimes`.
- Package entries: **128**.
- sha256: `a055571480c6bc23d248628bcdb966a48bddff25903f186ad9ab33afe0d93283`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Restaurant venue directory and live table availability. Read-only subset; no booking tools." —
  **91 characters** (verified programmatically).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/tabletimes:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio; the
  npm entry carries registryBaseUrl
  `https://registry.npmjs.org` and runtimeHint `npx`; the OCI
  entry carries runtimeArguments `--rm`/`-i` (named) and
  runtimeHint `docker`. Dockerfile (node:20-alpine, both
  stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/tabletimes"`.
  Container workflow `IMAGE` is
  `ghcr.io/cloudcorprecords/tabletimes` and its smoke test
  asserts the server name `tabletimes`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **66 tests, 66 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 2
   tools, including the no-`title`-key and required-key exactness
   assertions — `find_venues` has NO required key — and the
   anyOf/null property shapes; subset assertions that `book`,
   `get_reservation` and `join_waitlist` are absent at registry
   level, at the MCP layer (call → MethodNotFound) and at the CLI
   level; plus session handling/reuse, env override, the
   no-Authorization / no-X-API-Key assertions under any
   environment, verbatim per-tool argument forwarding for both
   tools and result pass-through including structuredContent and
   upstream isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working subcommand
   for every one of the 2 tools against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/eatloyalist-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/tabletimes/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/tabletimes/parity-out` → **verdict PASS, parity 100.0**
   (2/2 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.

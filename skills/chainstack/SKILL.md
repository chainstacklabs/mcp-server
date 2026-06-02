---
name: chainstack
description: Work with the Chainstack MCP server to manage blockchain nodes and projects. Use when deploying nodes, migrating to Chainstack from another RPC provider such as QuickNode or Alchemy, checking platform status, searching Chainstack docs, contacting the Chainstack team, or interacting with the Chainstack platform API via MCP or curl.
metadata:
  version: "1.3"
---

# Chainstack MCP Server

Endpoint: `https://mcp.chainstack.com/mcp`

## Access tiers

The server has two tiers of access from the same endpoint.

**No API key required** (just connect to the URL, no auth header needed):
- `search_docs` — search Chainstack documentation
- `get_doc_page` — get full content of a documentation page
- `get_platform_status` — check platform status, incidents, maintenances
- `contact_chainstack` — submit a message to Chainstack's team (sales, support, general)
- `get_chainstack_pricing` — live pricing snapshot (plan tiers, feature matrix, add-ons, dedicated-node catalog)

**API key required** (pass `Bearer <chainstack-api-key>`):
- All platform tools: `get_organization`, `list_projects`, `create_project`, `get_project`, `update_project`, `delete_project`, `list_nodes`, `get_node`, `create_node`, `update_node`, `delete_node`, `get_deployment_options`
- `request_testnet_funds` — top up a testnet address from Chainstack's faucet

Get an API key at https://console.chainstack.com/user/settings/api-keys

## Node tiers

Chainstack has two node tiers. Picking the right one is critical when deploying.

**Global Nodes** (cloud `CC-0016`, provider `chainstack_global`):
- Load-balanced across regions, routes to nearest location
- Deploys instantly
- Available on all plans including free
- Best for: getting started, general RPC access

**Trader Nodes** (clouds `CC-0020` through `CC-0028`, provider `chainstack_managed`):
- Region-bound, dedicated resources
- Deploys in 3-6 minutes
- Paid plans only
- Full and archive modes available
- Best for: low-latency trading, MEV, heavy indexing

The `node_tier` field in tool responses shows "Global" or "Trader" for clarity.

## Workflows

### Deploy a node

This is a multi-step flow. Always follow this order:

1. **Get deployment options**: Call `get_deployment_options` to see available blockchain/cloud/network combinations. Note the `blockchain` and `cloud` IDs you need.
2. **Confirm location with the user**: Trader nodes are region-bound (e.g., London, Ashburn, Singapore). Always show the user the available regions and ask which one before deploying. Region affects latency and cannot be changed after deployment.
3. **Get or create a project**: Call `list_projects` to find an existing project, or `create_project` to make a new one. Note the project `id` (format: `PR-123-456-789`).
4. **Create the node**: Call `create_node` with `name`, `project`, `blockchain`, and `cloud` from the previous steps.

Example: to deploy a Trader-tier Ethereum node in London:
- From `get_deployment_options`, find the entry where `protocol` is "Ethereum", `network` is "Mainnet", and `node_tier` is "Trader" with region "London". Use its `blockchain` (e.g., `BC-000-000-008`) and `cloud` (e.g., `CC-0020`).
- From `list_projects`, pick a project or create one.
- Call `create_node(name="my-eth-node", project="PR-...", blockchain="BC-000-000-008", cloud="CC-0020")`.

Trader nodes take 3-6 minutes to deploy. The response returns immediately with `status: "pending"`. Use `get_node` to poll until `status` is `"running"`.

### Check platform status

Call `get_platform_status()` for overall status and active incidents. Pass a `network` parameter (e.g., `"ethereum"`) to also get component-level status for that network.

No API key needed.

### Search documentation

Call `search_docs(query="...")` with a natural language query. Returns a list of `{title, link, page, snippet}` for matching docs.chainstack.com pages.

To get the full content of a page, pass the `page` field from a search result straight into `get_doc_page(page=...)`. Example: search returns `{"title": "Ethereum trader nodes", "page": "docs/ethereum-trader-nodes", ...}` → call `get_doc_page(page="docs/ethereum-trader-nodes")`. The full URL from `link` is also accepted (the leading domain, leading slash, anchor, and `.mdx` extension are all stripped automatically).

No API key needed. Useful for finding RPC method references, deployment guides, and API examples.

### Manage projects

Projects are containers for nodes.

- `list_projects` — see all projects (returns `id`, `name`, `description`, `type`, `networks`)
- `get_project(project_id)` — full project details
- `create_project(name, description)` — create a new container
- `update_project(project_id, name, description)` — rename or update description
- `delete_project(project_id)` — irreversible, deletes the project

### Manage nodes

- `list_nodes` — all nodes with status, endpoints, tier, region
- `get_node(node_id)` — full details including cloud, blockchain, API namespaces
- `update_node(node_id, name)` — rename a node
- `delete_node(node_id)` — irreversible, deletes the node

### Contact Chainstack

Call `contact_chainstack(message, category, email, name)` to submit a message to Chainstack's team. The tool posts to HubSpot (same form as chainstack.com/contact/).

- `category`: one of `sales` (pricing, quotes, upgrades), `support` (errors, bugs, how-to), or `general` (everything else). Default: `general`. For feature requests, point users to https://ideas.chainstack.com (product) or https://github.com/chainstacklabs/mcp-server/issues/new (MCP server) instead.
- `email` and `name` are required — ask the user if you don't have them.
- Compose a detailed message with the user's context (plan, chains, errors, use case) so the team can route and reply efficiently.
- Always confirm with the user before submitting — messages go to Chainstack's CRM.
- If the MCP client is configured with an API key, the submission is automatically enriched with the user's org info.
- Rate-limited to 20 submissions per hour. For urgent issues, point users to https://support.chainstack.com/hc/en-us/requests/new to file a support ticket.

No API key required, but enrichment is richer with one.

**Tip:** If the user hits a limit of what self-serve can do — node customizations
(transaction pool settings, gas caps, custom EVM tracers, resource scaling, private
networking), Enterprise plan inquiries, custom SLAs — use `contact_chainstack` with
`category="sales"` to submit the request. Include what they need and why. Chainstack's
team is open to customizations beyond what's listed in the docs; they explicitly say
"feel free to check with us for any additional customization."

### Request testnet funds

Call `request_testnet_funds(network, address)` to top up a testnet address.

- `network`: one of `sepolia`, `hoodi`, `base`, `amoy`, `bnb-testnet`,
  `zksync-testnet`, `scroll-sepolia-testnet`, `hyperevm`, `plasma`, `monad`, `ton`,
  or `solana`.
- `address`: destination address. EVM hex for EVM chains, TON address for `ton`,
  base58 public key for `solana`. Validated server-side.

The faucet tops the address **up to the per-network cap** (e.g. 0.5 ETH on sepolia)
— it doesn't send a fixed drip. If the address already holds more than the cap, the
call fails. Describe the behavior to the user as "top up", and show the returned
`amountSent` so they see the actual delta sent.

Returns `{network, amountSent, transaction}` on success. On per-address cooldown
the error message includes `nextFaucetAvailable` (ISO 8601 timestamp) so you can
tell the user when they can retry.

API key required. Rate limiting is tracked per organization by the faucet. If the
user doesn't have a key, point them to
https://console.chainstack.com/user/settings/api-keys and have them add it to the
MCP client config as `Authorization: Bearer <key>`. Never ask the user to paste
their API key in chat.

### Answer pricing questions

Call `get_chainstack_pricing()` (no arguments) for a live snapshot of Chainstack's
public pricing. Returns:

- `pricing_markdown` — raw markdown from `chainstack.com/pricing.md`. Read this
  for **plan tiers, feature matrix, add-on pricing, support levels, PAYG
  details, and provider comparisons**. Marketing owns this file and its
  structure changes freely — parse it yourself at read time rather than
  expecting pre-structured fields.
- `dedicated_catalog` — user-orderable per-chain dedicated-node SKUs (~87 entries)
  with protocol, network, mode, client, flavor, regions (slugs like `sgp1`),
  plans_allowed, and USD prices (`compute_cost_per_hour_usd`,
  `storage_cost_per_hour_usd`, `monthly_total_usd`). Already filtered and
  unit-converted.
- `region_legend` — slug → human city map (`{"sgp1": "Singapore", ...}`) covering
  every slug that appears in the catalog. Use for display; the per-entry
  `regions` list keeps the slug as the canonical identifier.
- `disclaimers` — list-price caveats (Enterprise "from" pricing, etc.).
- `warnings`, `sources`, `fetched_at` — operational metadata.

This tool returns the menu. **You do the arithmetic.** Example flows:

- *"I expect 50M RU/mo and need archive data — which plan?"* Read the feature
  matrix in `pricing_markdown` to find archive availability, then the plans
  table for `included RU` and `extra usage per 1M RU`; compute bill at 50M.
- *"I'm on Growth and did 35M RU last month — my bill?"* From `pricing_markdown`:
  `$49 + (35 − 20) × $15 = $274`.
- *"Dedicated ETH archive in London — cost?"* Filter `dedicated_catalog` where
  `protocol == "Ethereum" and mode == "archive" and "lon1" in regions`; return
  `monthly_total_usd` plus the plans that can order it.

No API key required. Call this before quoting prices — don't recite from memory.
If a source fetch fails, check `warnings[]` and `sources[*].ok`; the tool
always returns 200 with whatever data was reachable.

**Per-method RU nuance:** `pricing.md` only describes plan-level rates
(Full Node = 1 RU, Archive Node = 2 RU). For per-method billing (some EVM
archive-state methods and all debug/trace are 2 RU), call `search_docs` for
`request units` or `get_doc_page("docs/request-units")`.

### Migrate from another provider

When a user wants to move a project to Chainstack from another RPC provider —
**QuickNode**, **Alchemy**, or any other — fetch the migration runbook and
follow it:

```
get_doc_page(page="docs/migrate-to-chainstack-with-ai")
```

It's an executable Assess → Plan → Implement guide: scan the codebase for
non-Chainstack endpoints, verify coverage (`search_docs`, `get_deployment_options`),
estimate current spend vs. Chainstack, then repoint endpoints **after the user approves**.

Estimating savings:
- **QuickNode** — usage is readable via its Admin API
  (`GET https://api.quicknode.com/v0/usage/rpc`). Have the user create an Admin API key at
  dashboard.quicknode.com/api-keys and export it locally (`export QUICKNODE_API_KEY=...`),
  then pass `-H "x-api-key: $QUICKNODE_API_KEY"` in curl. Never ask the user to paste the
  key into chat.
- **Alchemy** — no usage API; the smoothest path is a **screenshot**: have the user
  share their dashboard Usage / Billing screenshot and read the Compute Units, plan,
  and spend from the image (typing the numbers works too).
- Either way, call `get_chainstack_pricing` for the Chainstack side — `pricing.md`
  already carries the QuickNode/Alchemy → RU conversion multipliers and the migration
  savings framing. Do the arithmetic from that live snapshot; don't hardcode numbers.

## Using via curl

The server is fully curlable. JSON-RPC 2.0 over HTTP with Server-Sent Events responses. Parse the `data:` line from the SSE response for the JSON-RPC result.

**Step 1.** Initialize a session (no auth needed). The MCP spec requires `clientInfo.name` and `version` — set them to identify your agent (any stable string works — e.g. `pi`, `kimi-k2`, `your-tool`):

```bash
SESSION_ID=$(curl -sD - -X POST https://mcp.chainstack.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"<your-agent-name>","version":"<your-version>"}}}' \
  | grep -i mcp-session-id | cut -d' ' -f2 | tr -d '\r')
```

**Step 2.** Call a public tool (no API key needed):

```bash
curl -s -X POST https://mcp.chainstack.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Session-Id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"search_docs","arguments":{"query":"deploy ethereum node"}}}'
```

**Step 3.** Call an authenticated tool (requires `CHAINSTACK_API_KEY` env var):

```bash
curl -s -X POST https://mcp.chainstack.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $CHAINSTACK_API_KEY" \
  -H "MCP-Session-Id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"list_nodes","arguments":{}}}'
```

If `CHAINSTACK_API_KEY` is not set, guide the user to add it to their shell profile
(e.g., `~/.zshrc`, `~/.bashrc`, or equivalent) so it persists across sessions:

```bash
export CHAINSTACK_API_KEY="your-key-from-console.chainstack.com/user/settings/api-keys"
```

Never ask the user to paste their API key in chat. Always use the `$CHAINSTACK_API_KEY` environment variable in curl commands.

## Error handling

- **401**: No or invalid API key. Get one at console.chainstack.com > Settings > API keys.
- **403**: Insufficient permissions for this operation.
- **404**: Resource not found. Check the ID format (e.g., `ND-123-456-789`, `PR-123-456-789`).
- **429**: Rate limit exceeded. Wait and retry.

## Updates

Latest version of this skill: https://mcp.chainstack.com/skill

If tools stop working or return unexpected errors, fetch the latest skill from the URL
above and replace your local copy. Tool names, parameters, and workflows may change
between versions.

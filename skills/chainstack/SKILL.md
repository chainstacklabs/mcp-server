---
name: chainstack
description: Use when deploying or managing blockchain nodes on Chainstack, getting an RPC endpoint across 70+ protocols (Ethereum, Solana, Base, BNB, Polygon, Arbitrum, and more), searching Chainstack docs, checking platform/RPC status, requesting testnet funds from the Chainstack faucet, checking Chainstack pricing, or contacting the Chainstack team — all via the hosted Chainstack MCP server. Triggers on "deploy a node", "get an RPC endpoint", "Chainstack", "testnet faucet", "node status", "add a Chainstack node", "Chainstack pricing".
license: MIT
metadata:
  author: chainstack
  version: "1.2"
  homepage: "https://mcp.chainstack.com"
---

# Chainstack MCP server

Work with the Chainstack MCP server to deploy and manage blockchain nodes and projects, search docs, check platform status, request testnet funds, and check pricing — across 70+ protocols.

**Endpoint:** `https://mcp.chainstack.com/mcp` (Streamable HTTP)
**Canonical / latest skill:** `https://mcp.chainstack.com/skill` — fetch this for the most current version; tool names and workflows may change between releases.

## Setup

Either register the MCP server (persistent tools) or rely on this skill (lean context):

- **Register the MCP server** in your agent pointing at `https://mcp.chainstack.com/mcp`. Docs search and platform status work with no auth; node/project management needs an API key.
- **API key** (for authenticated tools): get one at https://console.chainstack.com/user/settings/api-keys and send it as `Authorization: Bearer <api-key>`.

## Access tiers

**No API key required:**
- `search_docs` — search Chainstack documentation
- `get_doc_page` — retrieve full documentation page content
- `get_platform_status` — check platform status and incidents
- `contact_chainstack` — submit a message to the Chainstack team
- `get_chainstack_pricing` — live pricing snapshot

**API key required (Bearer token):**
All platform tools — organization, project and node management (`get_organization`, `list_projects`, `create_project`, `get_project`, `update_project`, `delete_project`, `list_nodes`, `get_node`, `create_node`, `update_node`, `delete_node`, `get_deployment_options`) plus `request_testnet_funds`.

## Node tiers

- **Global nodes** (`CC-0016`, `chainstack_global`) — load-balanced, instant deployment, available on all plans including free tier.
- **Trader nodes** (`CC-0020`–`CC-0028`, `chainstack_managed`) — region-bound, dedicated resources, 3–6 minute deployment, paid plans only; full and archive modes.

## Core workflows

### Deploy a node (multi-step)
1. `get_deployment_options` — available blockchain / cloud / network combinations.
2. Confirm region with the user (Trader nodes are region-bound; affects latency).
3. Get or create a project: `list_projects` or `create_project`.
4. `create_node` with name, project ID, blockchain, and cloud IDs.

Trader nodes return immediately with `status: "pending"` — poll `get_node` until `"running"`.

### Check platform status
`get_platform_status()` — optional `network` parameter for component-level details. No API key needed.

### Search documentation
`search_docs(query="...")` returns matching docs.chainstack.com pages (title, link, snippet). Pass the `page` field to `get_doc_page(page=...)` for full content.

### Manage projects
`list_projects` · `get_project(project_id)` · `create_project(name, description)` · `update_project(project_id, name, description)` · `delete_project(project_id)` (irreversible).

### Manage nodes
`list_nodes` (status, endpoints, tier, region) · `get_node(node_id)` · `update_node(node_id, name)` · `delete_node(node_id)` (irreversible).

### Contact Chainstack
`contact_chainstack(message, category, email, name)` — categories: `sales`, `support`, `general`. Always confirm before submitting. Rate-limited to 20/hour; urgent issues → https://support.chainstack.com/hc/en-us/requests/new

### Request testnet funds
`request_testnet_funds(network, address)` tops up testnet addresses to per-network caps (12 networks across EVM, Solana, TON). Returns `{network, amountSent, transaction}`. Requires API key; per-address cooldowns are returned in error messages.

### Answer pricing questions
`get_chainstack_pricing()` returns `pricing_markdown`, `dedicated_catalog` (~87 SKUs), `region_legend`, disclaimers, and metadata. Do the arithmetic yourself for estimates.

## Using via curl

Initialize a session:
```bash
SESSION_ID=$(curl -sD - -X POST https://mcp.chainstack.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"<your-agent-name>","version":"<your-version>"}}}' \
  | grep -i mcp-session-id | cut -d' ' -f2 | tr -d '\r')
```

Public tool (no API key):
```bash
curl -s -X POST https://mcp.chainstack.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Session-Id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"search_docs","arguments":{"query":"deploy ethereum node"}}}'
```

Authenticated tool (requires API key):
```bash
curl -s -X POST https://mcp.chainstack.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $CHAINSTACK_API_KEY" \
  -H "MCP-Session-Id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"list_nodes","arguments":{}}}'
```

## Error handling

- **401** — invalid/missing API key → get one at console.chainstack.com
- **403** — insufficient permissions
- **404** — resource not found (check ID format)
- **429** — rate limit exceeded

## Updates

This SKILL.md mirrors the canonical skill at https://mcp.chainstack.com/skill (v1.2). For the latest tool names, parameters, and workflows, fetch that URL.

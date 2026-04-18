# Changelog

All notable changes to the Chainstack MCP server are documented here. Subscribe to this repo (Watch → Releases) to get notified.

## 2026-04-18

**New tool: `request_testnet_funds`**

- Top up testnet addresses with gas for EVM, Solana, or TON development — directly from your agent.
- 12 networks: Ethereum Sepolia, Ethereum Hoodi, Base Sepolia, Polygon Amoy, BNB testnet, zkSync Sepolia, Scroll Sepolia, HyperEVM testnet, Plasma testnet, Monad testnet, TON testnet, Solana devnet.
- The faucet tops an address **up to the per-network cap** — it doesn't send a fixed amount. If the address is already at the cap, the call fails.
- API key required. Rate-limiting is tracked per organization (24h cooldown).
- On cooldown, the error includes `nextFaucetAvailable` (ISO 8601) so agents can tell you exactly when to retry.

## 2026-04-15

**New tool: `contact_chainstack`**

- Reach out to Chainstack directly from your agent for anything — sales, support, or general questions.
- No API key required. Agent asks for confirmation before sending; PII is redacted from server logs.

## 2026-04-09

**Chainstack MCP server is live at `mcp.chainstack.com`**

- 15 tools: search_docs, get_doc_page, get_platform_status, get_organization, get_deployment_options, list_projects, create_project, get_project, update_project, delete_project, list_nodes, get_node, create_node, update_node, delete_node
- Public tools (no auth): docs search, doc page fetch, platform status
- Authenticated tools (API key): node and project management
- Agent-readable discovery page at `/` — type `get mcp.chainstack.com` into any agent to get started
- Two integration paths: skill install (lean context) or MCP registration (tools always in context)
- Configs for Claude Code, Codex, Cursor, Windsurf, Gemini CLI, GitHub Copilot, Antigravity, Claude.ai, ChatGPT
- Skill file at `/skill` for on-demand usage without MCP registration
- Streamable HTTP transport

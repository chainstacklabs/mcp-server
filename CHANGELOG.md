# Changelog

All notable changes to the Chainstack MCP server are documented here. Subscribe to this repo (Watch → Releases) to get notified.

## 2026-05-22

**Google Antigravity 2.0 support**

- All three Google Antigravity surfaces — **Antigravity CLI** (the new `agy` terminal tool), **Antigravity** (the standalone desktop app), and **Antigravity IDE** — are now explicitly listed as supported on the [discovery page at mcp.chainstack.com](https://mcp.chainstack.com/) — in the skill install table, the install example, and the MCP registration block. All three share the same `~/.gemini/antigravity/mcp_config.json` config file and the same `~/.agents/skills/chainstack/` skill directory.
- Gemini CLI users — heads up: Google [announced](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/) at I/O 2026 that Gemini CLI is being retired on 2026-06-18 in favor of Antigravity CLI. Both are already supported with separate setup blocks on the discovery page; the migration is mostly seamless because they share the same MCP config path.

## 2026-05-12

**Safer tool execution + accurate server identity**

- Every Chainstack MCP tool now advertises standard [MCP tool annotations](https://modelcontextprotocol.io/specification/2025-06-18/server/tools) (`readOnlyHint`, `destructiveHint`, `openWorldHint`). Compatible clients (ChatGPT, Claude Code, Cursor, Windsurf) use these to skip confirmation prompts on read-only calls (docs search, status, pricing) and require explicit user approval on writes and deletes (creating/updating/deleting projects and nodes, sending contact messages, requesting testnet funds).
- `serverInfo.version` on the MCP `initialize` handshake now reports the actual Chainstack release version (e.g. `1.8.1`) instead of the underlying FastMCP library version. Surfaces in Claude Code, Codex, Cursor, Windsurf, Gemini CLI, GitHub Copilot, Antigravity, Claude.ai, and ChatGPT — wherever the connected MCP server's identity is shown.

## 2026-04-27

**Pi coding agent support + clearer tool description**

- [Pi](https://pi.dev/) coding agent users can now install the Chainstack skill: `curl -o ~/.agents/skills/chainstack/SKILL.md https://mcp.chainstack.com/skill`. All MCP server tools work via Pi's bash and the public HTTP endpoint.
- Pi uses the Skill route only — Pi [rejects MCP servers by design](https://mariozechner.at/posts/2025-11-02-what-if-you-dont-need-mcp/). The discovery page at `mcp.chainstack.com` now flags this so Pi users skip the wrong setup path.
- Discovery page description now mentions pricing and testnet funds. Agents reading `mcp.chainstack.com` see the full tool surface: deploy and manage blockchain nodes, search protocol docs and RPC references, check platform status, get pricing, request testnet funds, and reach Chainstack's team.

## 2026-04-20

**New tool: `get_chainstack_pricing`**

- Ask your agent anything about Chainstack pricing and get an answer grounded in live data — no guessing, no stale quotes.
- Covers plan tiers (Developer through Enterprise + Pay As You Go), the feature matrix, add-on pricing (Unlimited Node, Yellowstone gRPC, Warp transactions), and the full per-chain dedicated-node catalog (~87 user-orderable SKUs with regions, hardware flavors, hourly and monthly USD prices).
- Regions come with a slug → city legend (`sgp1` → Singapore, `fra1` → Frankfurt, etc.).
- No API key required. Every call fetches fresh from the public pricing sources — the server holds zero hardcoded pricing data, so changes on chainstack.com show up immediately.
- Example prompts: *"I expect 50M RU/month and need archive data — which plan fits?"*, *"How much is a dedicated Ethereum archive node in London?"*, *"Pro + Unlimited Node 250 RPS + 1000 Warp transactions — total monthly?"*

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

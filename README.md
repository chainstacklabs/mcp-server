# Chainstack MCP server

The Chainstack MCP server gives AI agents direct access to the Chainstack platform — deploy blockchain nodes, search documentation, and check platform status from any coding environment.

**Live at [`mcp.chainstack.com`](https://mcp.chainstack.com)**

## Quick start

Type this into any AI agent:

```
get mcp.chainstack.com
```

The agent will fetch the onboarding page and walk you through setup. Works with Claude Code, Cursor, Codex, Windsurf, Gemini CLI, GitHub Copilot, Antigravity, Claude.ai, and ChatGPT.

## Tools

**No API key needed:**
- `search_docs` — search Chainstack documentation
- `get_doc_page` — fetch full doc page content
- `get_platform_status` — check platform status and incidents
- `contact_chainstack` — send a message to Chainstack (sales, support, or general questions)
- `get_chainstack_pricing` — live pricing snapshot: plan tiers, feature matrix, add-ons, and the per-chain dedicated-node catalog

**API key required** ([get one here](https://console.chainstack.com/user/settings/api-keys)):
- `get_organization`, `list_projects`, `create_project`, `get_project`, `update_project`, `delete_project`
- `list_nodes`, `get_node`, `create_node`, `update_node`, `delete_node`, `get_deployment_options`
- `request_testnet_funds` — top up a testnet address from the Chainstack faucet (12 networks across EVM, Solana, and TON)

## Two ways to use

### Option 1: Install as a skill (lean context)

Cross-agent, via the [`skills`](https://github.com/vercel-labs/skills) CLI — it auto-detects your installed agent (50+ supported, including Claude Code, Cursor, Codex, Windsurf, Antigravity, GitHub Copilot, Pi, and OpenCode):

```bash
npx skills add chainstacklabs/mcp-server
```

This installs the bundled [`skills/chainstack`](skills/chainstack/SKILL.md) skill.

### Option 2: Register as an MCP server

```bash
claude mcp add --transport http chainstack https://mcp.chainstack.com/mcp -s user
```

See [full setup guide](https://docs.chainstack.com/docs/chainstack-mcp-server) for all agents.

## MCP registry

Published to the official [MCP Registry](https://registry.modelcontextprotocol.io) as `com.chainstack/chainstack`:

```bash
curl "https://registry.modelcontextprotocol.io/v0.1/servers?search=com.chainstack/chainstack&version=latest"
```

[`server.json`](server.json) is the manifest. Its `version` field is a placeholder — the publish workflow derives the real value from the live server card at `mcp.chainstack.com`, which renders it from the deployed chart, so the registry entry cannot drift from what is actually running. Query the registry, not this file, to see what is published.

## Issues & feature requests

Use [Issues](https://github.com/chainstacklabs/chainstack-mcp/issues) to:
- Report bugs or unexpected behavior
- Request new tools or features
- Suggest improvements to any Chainstack product

## Links

- [Discovery page](https://mcp.chainstack.com/) — agent-readable onboarding
- [Skill file](https://mcp.chainstack.com/skill) — for on-demand usage
- [Documentation](https://docs.chainstack.com/docs/chainstack-mcp-server)

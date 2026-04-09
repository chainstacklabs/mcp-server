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

**API key required** ([get one here](https://console.chainstack.com/user/settings/api-keys)):
- `get_organization`, `list_projects`, `create_project`, `get_project`, `update_project`, `delete_project`
- `list_nodes`, `get_node`, `create_node`, `update_node`, `delete_node`, `get_deployment_options`

## Two ways to use

### Option 1: Install as a skill (lean context)

```bash
mkdir -p ~/.claude/skills/chainstack
curl -o ~/.claude/skills/chainstack/SKILL.md https://mcp.chainstack.com/skill
```

### Option 2: Register as an MCP server

```bash
claude mcp add --transport http chainstack https://mcp.chainstack.com/mcp -s user
```

See [full setup guide](https://docs.chainstack.com/docs/chainstack-mcp-server) for all agents.

## Issues & feature requests

Use [Issues](https://github.com/chainstacklabs/chainstack-mcp/issues) to:
- Report bugs or unexpected behavior
- Request new tools or features
- Suggest improvements to any Chainstack product

## Links

- [Discovery page](https://mcp.chainstack.com/) — agent-readable onboarding
- [Skill file](https://mcp.chainstack.com/skill) — for on-demand usage
- [Documentation](https://docs.chainstack.com/docs/chainstack-mcp-server)

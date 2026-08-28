# Velixo Claude Code plugin

Connects Claude Code to supported ERP systems through the Velixo MCP server.

The MCP server does the expert work — it turns a natural-language question into a validated query, or
an intent into a command batch against the ERP's own screens — and ships its own operating manual
through `tool_guidance`. The plugin's job is to put that server in front of Claude Code in one step.

Skills and slash commands will follow. For now the plugin is the connection alone.

## Install

```
/plugin marketplace add VelixoSolutions/claude-plugin
/plugin install velixo@velixo
```

The repo is both the marketplace and the plugin, so those are the only two steps. Authentication to
the MCP server is OAuth: Claude Code opens a browser on first use, and the ERP connections you get
are the ones attached to your Velixo account.

## What's in it

```
claude-plugin/
├── .claude-plugin/
│   ├── plugin.json           # Plugin metadata
│   └── marketplace.json      # Single-plugin marketplace, so the repo installs directly
└── .mcp.json                 # Velixo MCP server (https://nx-dev.velixo.com/mcp)
```

### MCP server

`.mcp.json` points at the **dev** environment:

```json
{
    "velixo": {
        "type": "http",
        "url": "https://nx-dev.velixo.com/mcp"
    }
}
```

It exposes eight tools: `tool_guidance`, `list_connections`, `prepare_query`, `execute_query`,
`prepare_screen_commands`, `execute_screen_commands`, `get_status`, and `get_upload_url`.

The tool contract is ERP-neutral — every call is scoped to a `connectionName`, and each connection
reports its own `type`. **`list_connections` is the authority on what a given account can reach**;
the query language and the command vocabulary a build comes back in are whatever that connection's
ERP uses. Nothing in this plugin needs to change to pick up a connection type the server adds.

`tool_guidance` is the server's own operating manual, versioned with the server. It is a **mandatory
first call** before any `prepare_*` or `execute_*` — it covers the prepare → poll → execute loop,
write confirmations, and the results that look like answers but aren't.

Because the server is bundled with the plugin, its tools carry the scoped name
`mcp__plugin_velixo_velixo__<tool>` — `mcp__plugin_<plugin-name>_<server-name>__<tool>`. That is the
form `allowed-tools`, permission rules, and hook matchers need; a rule written against the bare
`velixo` key never fires.

## Developing

Validate the manifests before pushing:

```bash
claude plugin validate . --strict
```

Point the plugin at another environment by editing the URL in `.mcp.json` — the MCP host follows the
usual `nx-<environment>.velixo.com` pattern.

## License

Proprietary — see [LICENSE](LICENSE).

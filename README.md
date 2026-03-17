# cc-plugins

A Claude Code plugin marketplace. Browse and install plugins to extend Claude Code with new skills, agents, hooks, and integrations.

## Add this Marketplace

```bash
/plugin marketplace add allannapier/cc_plugins
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [example-plugin](./plugins/example-plugin/) | Template plugin demonstrating the standard plugin structure |

## Installing a Plugin

```bash
/plugin install <plugin-name>@cc-plugins
```

## Contributing a Plugin

1. Fork this repo
2. Copy `plugins/example-plugin/` to `plugins/your-plugin-name/`
3. Update `.claude-plugin/plugin.json` with your plugin's details
4. Add your functionality (skills, agents, hooks, MCP servers)
5. Write a README for your plugin
6. Register it in `.claude-plugin/marketplace.json`
7. Open a pull request

### Plugin Structure

```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json       # Required: plugin manifest
├── skills/               # Model/user-invocable commands
│   └── your-skill/
│       └── SKILL.md
├── agents/               # Subagent definitions
├── commands/             # Slash command shortcuts
├── hooks/                # Automated event handlers (hooks.json)
├── .mcp.json             # MCP server configs (optional)
└── README.md
```

### plugin.json Fields

```json
{
  "name": "your-plugin-name",
  "version": "1.0.0",
  "description": "What your plugin does",
  "author": {
    "name": "Your Name",
    "url": "https://github.com/you"
  },
  "license": "MIT",
  "keywords": ["tag1", "tag2"]
}
```

## Testing a Plugin Locally

```bash
claude --plugin-dir ./plugins/your-plugin-name
```

Use `/reload-plugins` inside a session to pick up changes without restarting.

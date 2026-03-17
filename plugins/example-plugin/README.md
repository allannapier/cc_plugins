# example-plugin

A template plugin demonstrating the standard plugin structure for the cc-plugins marketplace.

## Installation

```bash
/plugin marketplace add allannapier/cc_plugins
/plugin install example-plugin@cc-plugins
```

## Skills

### `hello-world`

Greets the user and confirms the plugin is working.

```
/example-plugin:hello-world
/example-plugin:hello-world Alice
```

## Development

Use this plugin as a starting point when creating new plugins:

1. Copy this directory to `plugins/your-plugin-name/`
2. Update `.claude-plugin/plugin.json` with your plugin's details
3. Add your skills to `skills/`, agents to `agents/`, hooks to `hooks/`
4. Register your plugin in `/.claude-plugin/marketplace.json` at the repo root
5. Commit and push

## Structure

```
example-plugin/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest (required)
├── skills/
│   └── hello-world/
│       └── SKILL.md      # Skill definition
├── agents/               # Subagent definitions (add .md files)
├── commands/             # Slash command shortcuts (add .md files)
├── hooks/                # Event hooks (add hooks.json)
└── README.md
```

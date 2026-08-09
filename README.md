# Yo — Agent Plugin

Let your agent **Yo** you: one MCP tool call sends a content-free push notification to your phone. The sender identity plus the moment of the ping _is_ the message.

This package is the public integration for the hosted Yo service ([Agent Plugins 1.0.0](https://agent-plugins.org/) format):

- `plugin.json` — plugin manifest
- `mcp.json` — portable Agent Plugins Streamable HTTP config for the hosted Yo MCP server
- `skills/yo/SKILL.md` — teaches your agent when and how to yo, and how to register
- `.codex-plugin/plugin.json` + `.mcp.json` — Codex-specific layout; `.mcp.json` uses the client `http` transport
- `.claude-plugin/plugin.json` + `marketplace.json` — Claude Code marketplace layout
- `glama.json` — Glama directory ownership claim
- `server.json` — official MCP Registry metadata for the hosted server. Its `version` is the registry record version, intentionally decoupled from the plugin manifests' `1.0.0`; because the registry rejects duplicate versions and our token can publish but not edit `io.github.alfongj-com/*`, metadata corrections require a version bump (currently `1.0.1`).

**The Yo service itself is hosted and closed-source.** This repo contains no server code and no secrets.

Install instructions for all platforms: https://getyo.dev/agents

This repository is mirrored automatically from a private monorepo; PRs here are not reviewed — use the contact on the install page.

## License

MIT (plugin package only).

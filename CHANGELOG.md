# Changelog

## 0.1.0 - 2026-08-05

Initial release.

- Bundled Moneo MCP server (`npx -y @moneolabs/mcp`) — nine guarded tools over
  a wallet on Robinhood Chain, key generated on first run and kept at
  `~/.moneo/key.json`
- `moneo` skill: the quote → confirm → execute flow, order types, and how to
  read a refusal
- `moneo-status` skill: address, balance, limits, positions, recent activity
- `moneo-integrate` skill: add a guarded wallet to your own agent projects with
  the TypeScript SDK
- Always-on spend-safety rule: spending tools need explicit approval, free
  reads preferred, refusals surfaced and never routed around

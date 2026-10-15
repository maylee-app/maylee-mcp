# Changelog

Notable changes to the Maylee MCP server. Versions match the `version` of [`server.json`](server.json).

## 1.0.0 (2026-10-15)

First public version.

- Hosted server at `https://mcp.maylee.app/mcp`, over Streamable HTTP.
- OAuth 2.1 sign-in with dynamic client registration and PKCE, one workspace per connection, and three permissions: Read, Organize and draft, Send. Maylee API keys work too.
- 69 tools in 5 toolsets: reference, account, emails, compose and configuration.
- 3 prompts (`triage_inbox`, `follow_up_waiting_for_reply`, `set_up_ai_labels`) and the Maylee reference data as MCP resources.
- Guardrails: explicit confirmation before sending and other irreversible actions, idempotency keys on sends, hourly safety budgets per connection.
- Toolset filter: `?toolsets=` loads only part of the tools.

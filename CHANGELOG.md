# Changelog

Notable changes to the Maylee MCP server. Versions match the `version` of [`server.json`](server.json).

## Tool updates (2026-10-20)

[`tools.json`](tools.json) and [`docs/TOOLS.md`](docs/TOOLS.md) catch up with the server.

- Signatures: one signature now covers several addresses. `create_signature` and `update_signature` take `addresses` instead of `mailboxId` and `aliasEmail`, and signatures no longer have an icon (`iconName`, the `signature_icons` reference topic and its resource are gone: 13 reference resources).
- Notifications: `get_notification_settings` and `update_notification_settings` follow the new settings: all emails or only the chosen mailboxes and labels, and active hours: a notification that arrives outside them is set aside and delivered when they start again, alone or as one recap.
- `create_draft`, `send_email`, `draft_reply`, `draft_forward`, `suggest_contacts` and `get_reference`: descriptions and parameters brought up to date.

## 1.0.0 (2026-10-15)

First public version.

- Hosted server at `https://mcp.maylee.app/mcp`, over Streamable HTTP.
- OAuth 2.1 sign-in with dynamic client registration and PKCE, one workspace per connection, and three permissions: Read, Organize and draft, Send. Maylee API keys work too.
- 69 tools in 5 toolsets: reference, account, emails, compose and configuration.
- 3 prompts (`triage_inbox`, `follow_up_waiting_for_reply`, `set_up_ai_labels`) and the Maylee reference data as MCP resources.
- Guardrails: explicit confirmation before sending and other irreversible actions, idempotency keys on sends, hourly safety budgets per connection.
- Toolset filter: `?toolsets=` loads only part of the tools.

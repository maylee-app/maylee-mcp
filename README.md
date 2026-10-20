<p align="center">
  <a href="https://www.maylee.app">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="assets/maylee-logo-white.svg">
      <img src="assets/maylee-logo.svg" alt="Maylee" width="240">
    </picture>
  </a>
</p>

<h1 align="center">Maylee MCP Server</h1>

<p align="center">
  Connect Claude, ChatGPT, Cursor or any MCP client to your Maylee inbox.<br>
  Read, search, triage and draft email from your AI assistant, and send only what you approve.
</p>

<p align="center">
  <a href="https://modelcontextprotocol.io"><img src="https://img.shields.io/badge/MCP-Streamable_HTTP-BD9C74" alt="MCP: Streamable HTTP"></a>
  <a href="#authentication"><img src="https://img.shields.io/badge/auth-OAuth_2.1_%7C_API_key-BD9C74" alt="Auth: OAuth 2.1 or API key"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-BD9C74" alt="License: MIT"></a>
</p>

<p align="center">
  <a href="https://docs.maylee.app/mcp">Documentation</a> ·
  <a href="https://www.maylee.app">Website</a> ·
  <a href="docs/TOOLS.md">Tools</a> ·
  <a href="examples">Examples</a> ·
  <a href="#support">Support</a>
</p>

[Maylee](https://www.maylee.app) is an AI email client for Gmail, Microsoft 365 and IMAP mailboxes. This repository documents its official [Model Context Protocol](https://modelcontextprotocol.io) server: a hosted, remote server that gives an AI assistant access to **one Maylee workspace**, with the permissions you choose. The assistant can read, search and triage email, read attachments, prepare replies, forwards and new drafts, send or schedule them when you approve, and manage labels, AI label rules, views, inbox filters, auto-forward rules, signatures, snippets and notification settings.

There is nothing to install: add the server URL to your assistant, sign in to Maylee, and pick what the assistant may do.

## Contents

- [What you can ask](#what-you-can-ask)
- [Server URL](#server-url)
- [Authentication](#authentication)
- [Quick start](#quick-start): [Claude](#claude-claudeai-and-claude-desktop), [Claude Code](#claude-code), [ChatGPT](#chatgpt), [Cursor](#cursor), [VS Code](#vs-code), [Windsurf](#windsurf), [stdio clients](#clients-that-only-support-stdio)
- [Tools](#tools)
- [Security and guardrails](#security-and-guardrails)
- [Privacy](#privacy)
- [FAQ](#faq)
- [Support](#support)

## What you can ask

| Ask your assistant | What happens |
| --- | --- |
| "Triage my unread email and draft replies to the clients." | The assistant lists the unread inbox, opens the conversations that matter and prepares replies with `draft_reply`. Nothing is sent until you approve each draft. |
| "What did Acme send me this week? Summarize the thread and the PDF they attached." | It finds the emails with `search_emails`, then reads the conversation with `get_thread` and the attachment with `get_attachment`. |
| "Which of the emails I sent last week are still waiting for a reply? Draft polite follow-ups." | It lists the emails waiting for a reply, checks each conversation, and drafts follow-ups for you to review. |
| "Tag invoices and payment reminders automatically, and keep them out of my inbox." | It creates an AI label rule from your plain-language condition, then offers to apply it to the emails already in your inbox. |
| "Send this reply tomorrow at 9:00." | It schedules the draft in your workspace timezone with `schedule_draft`, once you confirm. |
| "Unsubscribe me from this newsletter." | It checks how to unsubscribe from that sender, then unsubscribes once you confirm. |

Three built-in prompts guide the most common workflows: `triage_inbox`, `follow_up_waiting_for_reply` and `set_up_ai_labels`.

## Server URL

```text
https://mcp.maylee.app/mcp
```

- **Transport**: Streamable HTTP, stateless, JSON responses (`POST` only, no server-sent events).
- **Authentication**: OAuth 2.1 (recommended) or a Maylee API key sent as a Bearer token.
- **Scope**: one Maylee workspace per connection, with the permissions you grant.
- **Tools**: **69 tools** in **5 toolsets**, plus 3 prompts and 13 reference resources.
- **Plans**: every Maylee plan, within the API quota of the workspace (see [Quotas and limits](#quotas-and-limits)).
- **Registry manifest**: [`server.json`](server.json), for the official MCP registry (`app.maylee/maylee`).

## Authentication

### OAuth (recommended)

Your assistant discovers everything from the server URL. On its first request it receives a `401` with an OAuth challenge, registers itself (dynamic client registration, nothing to create on the Maylee side) and opens Maylee in your browser:

1. **Sign in to Maylee**, second factor included if you use one.
2. **Check the consent screen.** The application name is declared by the application itself, so it is shown as unverified. Check above all the address you will be sent back to: for example `claude.ai`, or "a program running on this computer" for Claude Code. Only continue if you started the connection yourself.
3. **Pick one workspace and the [permissions](#permissions)**, then click **Allow**.

Each connection is bound to the workspace and the return address you approved. To use another workspace, connect again and pick it. Access tokens last 30 days and refresh tokens 180 days, rotated on every use. Every request checks again that the connection is active, that you are still a member of the workspace and that the workspace is not suspended.

**Disconnect an assistant**: in Maylee, open **Settings**, **API keys**, section **AI assistants (MCP)**, and click **Revoke access**. The assistant loses access within a minute, and its tokens never work again, even if you reconnect it later. Resetting your password disconnects every assistant.

### API key

For scripts, CI jobs and clients configured with a header, use a Maylee API key instead:

1. A workspace owner or admin opens **Settings**, **API keys** in Maylee and clicks **Create a key**.
2. Name the key and choose its scopes: `read` (required by the MCP server), `write` to organize and draft, and **Send emails** only if the assistant must send.
3. Copy the key, which starts with `mk_`. It is shown only once.

Send it in the `Authorization` header:

```http
Authorization: Bearer mk_your_api_key
```

**Revoke a key**: **Settings**, **API keys**, **Revoke key**. Every client that uses it stops working immediately. Tools that act for a person (reply reminders, notification settings) need OAuth: they are hidden when you connect with an API key.

### For client developers

Clients find the OAuth configuration by themselves. A request without a token gets a `401` whose `WWW-Authenticate` header points to the protected resource metadata (RFC 9728), `https://mcp.maylee.app/.well-known/oauth-protected-resource/mcp`, which names the authorization server. Clients register through dynamic client registration (RFC 7591) as public clients, with PKCE (S256). The scopes are `maylee:read`, `maylee:write` and `maylee:send`, plus `offline_access` for refresh tokens.

### Permissions

| Permission | OAuth scope | API key | What the assistant can do |
| --- | --- | --- | --- |
| **Read** | `maylee:read`, always granted | `read` scope | Read emails, conversations, attachments, drafts, labels, views, rules, contacts, unread counts and settings. |
| **Organize and draft** | `maylee:write` | `write` scope | Mark read, pin, label, move to Inbox, Spam or Trash, create and edit drafts, manage labels, views, AI label rules, inbox filters, signatures, snippets and notification settings, sync a mailbox. |
| **Send** | `maylee:send`, with `maylee:write` | **Send emails**, with `write` | Send and schedule emails, unsubscribe from mailing lists, turn on automatic replies, create, change or delete auto-forward rules. |

A tool the connection may not use is not listed at all. If the assistant calls one anyway, it gets `INSUFFICIENT_SCOPE` with an explanation of how to get the permission.

## Quick start

### Claude (claude.ai and Claude Desktop)

Custom connectors are added once to your Claude account and are then available in Claude Desktop too. The Claude free plan is limited to one custom connector.

1. On claude.ai, open **Customize**, then **Connectors**. Click **+ Add**, then **Add custom connector**.
2. Name it `Maylee`, enter `https://mcp.maylee.app/mcp` as the server URL, then click **Continue**.
3. Keep the detected sign-in (OAuth). For the OAuth client, choose **Register automatically**.
4. Click **Add**, then sign in to Maylee in the page that opens.

On Team and Enterprise plans, an owner adds the connector once in **Organization settings**, **Connectors**, **Add**, **Custom**, **Web**. Each member then opens **Customize**, **Connectors**, finds Maylee and clicks **Connect** to sign in with their own Maylee account.

### Claude Code

```bash
claude mcp add --transport http maylee https://mcp.maylee.app/mcp
```

Then run `/mcp` in Claude Code, select `maylee` and sign in to Maylee in the browser. Add `--scope user` to use it in every project. With an API key instead of OAuth:

```bash
claude mcp add --transport http maylee https://mcp.maylee.app/mcp \
  --header "Authorization: Bearer mk_your_api_key"
```

A project-level [`.mcp.json`](examples/claude-code/.mcp.json) is in the [examples](examples/claude-code/README.md).

### ChatGPT

1. Go to **Plugins** (`chatgpt.com/plugins`), select **+**, then **Add custom MCP server**.
2. Name it `Maylee` and enter `https://mcp.maylee.app/mcp` as the **Server URL**.
3. For authentication, choose **OAuth** without static credentials: ChatGPT registers itself.
4. Review the warning, select **I understand and want to continue**, then **Create as a plugin**, and sign in to Maylee when asked.
5. Install the plugin, then use it in a conversation.

Depending on your plan, a workspace admin may first have to allow developer mode and custom MCP servers.

### Cursor

Add the server to `.cursor/mcp.json` in a project, or to `~/.cursor/mcp.json` for every project ([example](examples/cursor/.cursor/mcp.json)):

```json
{
  "mcpServers": {
    "maylee": {
      "url": "https://mcp.maylee.app/mcp?toolsets=reference,emails,compose"
    }
  }
}
```

Cursor runs the OAuth sign-in itself. The [`toolsets`](#toolsets) filter keeps only the reference, email and compose tools, because every tool description takes room in the model's context: add `account` or `config` if you need them. To use an API key, see [this example](examples/cursor/mcp.api-key.json).

### VS Code

Run **MCP: Add Server** from the Command Palette and enter the server URL, or add it to `.vscode/mcp.json` ([example](examples/vscode/.vscode/mcp.json)):

```json
{
  "servers": {
    "maylee": {
      "type": "http",
      "url": "https://mcp.maylee.app/mcp"
    }
  }
}
```

On the first connection, VS Code registers itself and opens the Maylee sign-in in your browser. To use an API key without writing it in the file, see [this example](examples/vscode/mcp.api-key.json).

### Windsurf

Windsurf (now Devin Desktop) reads `~/.config/devin/mcp_config.json`, or `%APPDATA%\devin\mcp_config.json` on Windows. Older Windsurf versions read `~/.codeium/windsurf/mcp_config.json`. Add the server there ([example](examples/windsurf/mcp_config.json)):

```json
{
  "mcpServers": {
    "maylee": {
      "url": "https://mcp.maylee.app/mcp"
    }
  }
}
```

The sign-in runs over OAuth. The Cascade agent uses at most 100 tools at a time across all servers: if you reach that limit, add a [`toolsets`](#toolsets) filter to the URL.

### Clients that only support stdio

A client that can only start local servers can reach Maylee through [`mcp-remote`](https://github.com/geelen/mcp-remote), a local bridge that needs Node.js 18 or later. For example, in `claude_desktop_config.json` ([example](examples/claude-desktop/claude_desktop_config.json)):

```json
{
  "mcpServers": {
    "maylee": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.maylee.app/mcp"]
    }
  }
}
```

`mcp-remote` opens the Maylee sign-in in your browser on first use. To use an API key instead, see [this example](examples/claude-desktop/claude_desktop_config.api-key.json). In Claude Desktop itself, a [custom connector](#claude-claudeai-and-claude-desktop) is simpler.

### Other clients

Any client that supports remote MCP servers over Streamable HTTP works with the server URL. A client that supports MCP authorization handles OAuth by itself; otherwise, send an API key in the `Authorization` header. A quick test with `curl`:

```bash
curl -s https://mcp.maylee.app/mcp \
  -H "Authorization: Bearer mk_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_workspace","arguments":{}}}'
```

### Toolsets

A client can load only some toolsets by adding `?toolsets=` to the URL:

```text
https://mcp.maylee.app/mcp?toolsets=emails,compose
```

Values: `reference`, `account`, `emails`, `compose` and `config`, separated by commas. Without the parameter, every toolset is loaded. An unknown name is refused with `400 VALIDATION_FAILED`.

## Tools

The server exposes **69 tools** in **5 toolsets**. Every tool has a title, MCP annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`), an input schema and an output schema. The full list with permissions is in [docs/TOOLS.md](docs/TOOLS.md), and every parameter and output field is in [tools.json](tools.json) and on [docs.maylee.app](https://docs.maylee.app/mcp/tools).

<!-- generated:tools-summary:start -->
| Toolset | Tools | What it covers |
| --- | ---: | --- |
| [Reference](docs/TOOLS.md#reference) | 1 | `get_reference`: label colors, view icons, the filter grammar, rule options, limits and safety rules. |
| [Account](docs/TOOLS.md#account) | 10 | Workspace and connection, mailboxes, sync, unread counts, contacts, reply reminders, notification settings, timezone. |
| [Emails](docs/TOOLS.md#emails) | 14 | List, search and read emails, threads and attachments; mark read, pin, move, trash, label; waiting for reply; unsubscribe. |
| [Compose](docs/TOOLS.md#compose) | 22 | Drafts, replies, forwards, sending, scheduling, attachments, automatic replies, Magic Reply, signatures, snippets. |
| [Configuration](docs/TOOLS.md#configuration) | 22 | Labels, AI label rules, views, inbox filters, auto-forward rules. |
| **Total** | **69** | 22 Read, 40 Organize and draft, 7 Send |

**Reference**: `get_reference`

**Account**: `get_workspace`, `list_mailboxes`, `sync_mailbox`, `get_unread_counts`, `suggest_contacts`, `list_reply_reminders`, `dismiss_reply_reminders`, `get_notification_settings`, `update_notification_settings`, `set_workspace_timezone`

**Emails**: `list_emails`, `search_emails`, `get_email`, `get_thread`, `get_attachment`, `mark_emails_read`, `pin_emails`, `move_emails`, `trash_emails`, `apply_label_to_emails`, `remove_label_from_emails`, `dismiss_waiting_for_reply`, `get_unsubscribe_options`, `unsubscribe`

**Compose**: `get_draft`, `create_draft`, `update_draft`, `delete_draft`, `send_draft`, `send_email`, `draft_reply`, `draft_forward`, `schedule_draft`, `cancel_scheduled_send`, `add_draft_attachment`, `remove_draft_attachment`, `cancel_auto_reply`, `generate_magic_reply`, `list_signatures`, `create_signature`, `update_signature`, `delete_signature`, `list_snippets`, `create_snippet`, `update_snippet`, `delete_snippet`

**Configuration**: `list_labels`, `create_label`, `update_label`, `delete_label`, `list_ai_label_rules`, `create_ai_label_rule`, `update_ai_label_rule`, `delete_ai_label_rule`, `classify_existing_emails`, `list_views`, `create_view`, `update_view`, `delete_view`, `list_inbox_filters`, `create_inbox_filter`, `update_inbox_filter`, `delete_inbox_filter`, `list_auto_forward_rules`, `create_auto_forward_rule`, `update_auto_forward_rule`, `delete_auto_forward_rule`, `get_auto_forward_logs`

<!-- generated:tools-summary:end -->

## Security and guardrails

- **You approve every send.** Sending and scheduling need the Send permission and `confirm: true`, which the assistant is told to set only after you approved that exact email: recipients, subject, content and attachments. Without it, the call only returns a summary to show you. Unsubscribing, creating or changing auto-forward rules, turning on automatic replies, deleting drafts, labels, views, AI label rules, signatures or snippets, and moving more than 10 emails to Trash or Spam need the same confirmation.
- **No double sends.** Send tools need an idempotency key, so a retry after a timeout never sends the same email twice.
- **Email content stays data.** Bodies, attachments, sender names and subjects are written by third parties. The server tells the assistant to treat them as data and never to follow instructions found in them.
- **Hourly safety budgets** per connection: 50 sends (send, schedule, unsubscribe), 200 recipients (to, cc and bcc together, at most 50 in one email) and 500 emails moved to Trash or Spam, plus 30 Magic Reply generations per workspace. If these budgets cannot be checked, sending, scheduling, unsubscribing, trashing and moving to Spam are refused instead of running unchecked.
- **Nothing is deleted for good.** The server never deletes email permanently: Trash stays recoverable.
- **What Maylee sends by itself stays as you approved it.** A scheduled email or a pending automatic reply cannot be edited through the server: the assistant must cancel it first.
- **No secrets.** The server never returns credentials, API keys, storage keys or pre-signed download links.
- **Strict arguments.** Unknown arguments are refused, not ignored, so a misspelled parameter never passes silently.
- **Out of reach on purpose**: permanent deletion, API keys, webhooks and billing, connecting or removing mailboxes, workspace and account settings.

### Quotas and limits

Every tool call counts against the API quota of the workspace, shared with the Maylee REST API. Bulk tools count one request per email id, and `get_reference` is free.

| Plan | Requests per minute | Requests per month | Email ids per bulk call |
| --- | ---: | ---: | ---: |
| Free | 60 | 10,000 | 60 |
| Premium | 300 | 200,000 | 100 |
| Expert | 1,000 | 1,000,000 | 100 |

Responses are bounded: pages of at most 50 emails; `get_thread` returns the 10 newest messages and 4,000 characters per body by default, and at most 60,000 characters of bodies; `get_attachment` reads files up to 5 MB. A tool call times out after 55 seconds. Magic Reply, Auto-Draft and Auto-Reply need a Premium or Expert plan.

## Privacy

- The assistant only reaches the workspace you picked, with the permissions you granted, until you revoke it.
- What the assistant reads (emails, attachments, contacts) goes to the AI client you use, such as Claude, ChatGPT or Cursor, under that provider's own terms. Grant only the permissions you need.
- Usage metrics record which tool was called, whether it succeeded and how long it took. They never include email content, subjects, addresses or search terms.
- The [Maylee privacy policy](https://www.maylee.app/privacy-policy) applies to the service.

## FAQ

### Where is the source code?

The Maylee MCP server is a hosted service: it runs on Maylee's servers, and its code is not public. This repository holds everything you need to use it: the documentation, the registry manifest ([`server.json`](server.json)), the machine-readable tool reference ([`tools.json`](tools.json)) and ready-to-copy client configurations. There is nothing to build or run.

### Do I need to install anything?

No. Clients that support remote MCP servers connect to the URL directly. Only clients limited to local (stdio) servers need the [`mcp-remote`](#clients-that-only-support-stdio) bridge.

### Which mailboxes does it work with?

Every mailbox connected to your Maylee workspace: Gmail, Microsoft 365 and IMAP. Mailboxes are connected and reconnected in the Maylee app; the MCP server cannot do it.

### Can the assistant send email without my approval?

Only a connection with the Send permission can send at all. Even then, every send needs `confirm: true`, which the assistant is told to set only after you approved that exact email; without it, the call only returns a summary. Most clients can also ask you before running a tool.

### Can I use several workspaces?

A connection reaches one workspace. To use another one, add the server again and pick the other workspace on the consent screen.

### My client limits the number of tools, or the context fills up.

Load only the toolsets you need with [`?toolsets=`](#toolsets), for example `?toolsets=reference,emails,compose`.

### How do I disconnect an assistant?

In Maylee, **Settings**, **API keys**, section **AI assistants (MCP)**, **Revoke access**. For an API key, click **Revoke key** on the same page.

## Support

- Questions about Maylee, your account or a connection: [contact@maylee.app](mailto:contact@maylee.app).
- A mistake in this documentation or in a client configuration: [open an issue](https://github.com/maylee-app/maylee-mcp/issues/new/choose).
- A security vulnerability: follow [SECURITY.md](SECURITY.md), and do not open a public issue.

Documentation: [docs.maylee.app](https://docs.maylee.app) · Website: [www.maylee.app](https://www.maylee.app) · [Privacy policy](https://www.maylee.app/privacy-policy) · [Terms of use](https://www.maylee.app/terms-of-use)

## License

The documentation and configuration files of this repository are released under the [MIT License](LICENSE). The Maylee service, including the hosted MCP server, is proprietary and governed by the [Maylee terms of use](https://www.maylee.app/terms-of-use).

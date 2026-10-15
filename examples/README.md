# Client configurations

Ready-to-copy configurations for the Maylee MCP server, `https://mcp.maylee.app/mcp`. Each client has an OAuth version, recommended: the client opens Maylee in your browser, you sign in, pick one workspace and the permissions. Clients that accept HTTP headers also have an API key version.

| Client | OAuth | API key | Where the file goes |
| --- | --- | --- | --- |
| Claude Code | [`claude-code/README.md`](claude-code/README.md), [`.mcp.json`](claude-code/.mcp.json) | [`claude-code/README.md`](claude-code/README.md#with-an-api-key) | `.mcp.json` at the root of a project, or the `claude mcp add` command |
| Claude Desktop, through `mcp-remote` | [`claude_desktop_config.json`](claude-desktop/claude_desktop_config.json) | [`claude_desktop_config.api-key.json`](claude-desktop/claude_desktop_config.api-key.json) | `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS), `%APPDATA%\Claude\claude_desktop_config.json` (Windows) |
| Cursor | [`.cursor/mcp.json`](cursor/.cursor/mcp.json) | [`mcp.api-key.json`](cursor/mcp.api-key.json) | `.cursor/mcp.json` in a project, or `~/.cursor/mcp.json` |
| VS Code | [`.vscode/mcp.json`](vscode/.vscode/mcp.json) | [`mcp.api-key.json`](vscode/mcp.api-key.json) | `.vscode/mcp.json` in a workspace, or **MCP: Open User Configuration** |
| Windsurf (Devin Desktop) | [`mcp_config.json`](windsurf/mcp_config.json) | [`mcp_config.api-key.json`](windsurf/mcp_config.api-key.json) | `~/.config/devin/mcp_config.json`, `%APPDATA%\devin\mcp_config.json` (Windows), or `~/.codeium/windsurf/mcp_config.json` on older Windsurf versions |

If the file already exists, add the `maylee` entry next to your other servers instead of replacing the file. Each `*.api-key.json` example is the content of the file named in the last column.

Claude (claude.ai and Claude Desktop) and ChatGPT add the server from their settings, without a file: see the [quick start](../README.md#quick-start).

## Using an API key

Create a key in Maylee, **Settings**, **API keys**, **Create a key** (workspace owners and admins), with the `read` scope, `write` to organize and draft, and **Send emails** only if the assistant must send. See [Authentication](../README.md#api-key).

- **Cursor and Windsurf** read the key from the `MAYLEE_API_KEY` environment variable (`${env:MAYLEE_API_KEY}`), so it stays out of the file. Set it where the client is started, for example `export MAYLEE_API_KEY=mk_your_api_key` in a terminal, then start the client from that terminal. An app opened from the Dock or the Start menu may not see variables set in a shell profile.
- **VS Code** asks for the key the first time the server starts and stores it securely (`${input:maylee-api-key}`).
- **Claude Code** expands `${MAYLEE_API_KEY}` in `.mcp.json`, or takes the key in the `--header` option of `claude mcp add`.
- **Claude Desktop through `mcp-remote`** reads the header from the `env` block of its configuration: replace `mk_your_api_key` with your key. Keep that file private.

Never commit a file that contains a real key. If a key leaks, revoke it in **Settings**, **API keys**, **Revoke key**.

## Toolsets

Every example can load fewer tools by adding `?toolsets=` to the URL, for example `https://mcp.maylee.app/mcp?toolsets=reference,emails,compose`. The Cursor examples do it by default. Values: `reference`, `account`, `emails`, `compose`, `config`.

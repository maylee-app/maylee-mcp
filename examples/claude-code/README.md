# Claude Code

## With OAuth (recommended)

```bash
claude mcp add --transport http maylee https://mcp.maylee.app/mcp
```

Then run `/mcp` in Claude Code, select `maylee` and sign in to Maylee in the browser page that opens: pick one workspace and the permissions.

Add `--scope user` to use the server in every project, or `--scope project` to write it to the `.mcp.json` file of the project, shared with your team. That file is the same as [`.mcp.json`](.mcp.json):

```json
{
  "mcpServers": {
    "maylee": {
      "type": "http",
      "url": "https://mcp.maylee.app/mcp"
    }
  }
}
```

Each person still signs in to their own Maylee account with `/mcp`.

## With an API key

```bash
claude mcp add --transport http maylee https://mcp.maylee.app/mcp \
  --header "Authorization: Bearer mk_your_api_key"
```

In a shared `.mcp.json`, read the key from an environment variable instead of writing it in the file. Claude Code expands `${MAYLEE_API_KEY}` when it starts:

```json
{
  "mcpServers": {
    "maylee": {
      "type": "http",
      "url": "https://mcp.maylee.app/mcp",
      "headers": {
        "Authorization": "Bearer ${MAYLEE_API_KEY}"
      }
    }
  }
}
```

Create the key in Maylee, **Settings**, **API keys**, **Create a key**, with the `read` scope, `write` to organize and draft, and **Send emails** only if the assistant must send.

## Fewer tools

Add `?toolsets=` to the URL to load only some toolsets, for example `https://mcp.maylee.app/mcp?toolsets=reference,emails,compose`.

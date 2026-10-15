# Security policy

The Maylee MCP server gives AI assistants access to email, so we take its security seriously. Thank you for helping keep Maylee users safe.

## Reporting a vulnerability

Email **contact@maylee.app** with the subject **Security**. Please do not open a public issue, pull request or discussion for a vulnerability.

A useful report includes:

- what is affected: the MCP endpoint, the OAuth flow or consent screen, a specific tool, or a file of this repository;
- the steps to reproduce, with the requests and responses involved;
- the impact you observed or expect, for example access to another workspace, a send without confirmation, or a permission bypass;
- the date and time of your tests, with their timezone;
- how to reach you, and whether you would like to be credited.

Before sending, remove access tokens, API keys, passwords and the content of real emails from your report. If you think a Maylee API key leaked, revoke it right away in Maylee, **Settings**, **API keys**, and connected assistants in the **AI assistants (MCP)** section of the same page.

## Testing guidelines

- Test only with your own Maylee account, workspace and mailboxes. Do not access, change or delete data that is not yours.
- Do not send email to people who did not agree to receive it.
- No denial of service, load testing, spam or social engineering.
- Give us a reasonable amount of time to fix an issue before you disclose it.

## Scope

- The hosted MCP server at `https://mcp.maylee.app/mcp`, its OAuth endpoints and its consent screen.
- The contents of this repository: documentation, `server.json`, `tools.json`, the scripts and the client configurations.

The Maylee apps and the Maylee REST API are covered by the same contact address.

# Sentinel MCP Server

[Sentinel](https://general.legal) is an AI-native legal contract review platform. Upload contracts and get them reviewed by a combination of AI and a human attorney — directly from your AI assistant.

The Sentinel MCP server lets any MCP-compatible client (Claude Code, Claude Desktop, etc.) upload, track, and download legal contracts through natural conversation.

## What you can do

| Tool | Description |
|------|-------------|
| **upload_contract** | Upload a `.docx` contract for legal review |
| **confirm_upload** | Confirm the upload and trigger the AI + human review pipeline |
| **list_contracts** | List your contracts, check review status, and find contract/version IDs |
| **download_contract** | Download a reviewed contract with the attorney's comments and redlines |

### Typical workflow

1. **Upload** — call `upload_contract` with your `.docx` filename to get a signed upload URL
2. **Transfer** — upload the file to that URL (the agent runs a `curl` command)
3. **Confirm** — call `confirm_upload` to kick off the review pipeline, optionally providing context for the legal team
4. **Wait** — the contract goes through AI review, then a human attorney reviews it (this can take several hours)
5. **Check** — call `list_contracts` periodically to check the status
6. **Download** — once the status is `delivered` or `completed`, call `download_contract` to get the reviewed version with the attorney's changes and comments

## Connecting

### Claude Code

```bash
claude mcp add sentinel https://sentinel-mcp-server-299005046024.us-west1.run.app/mcp
```

Then start a conversation and ask Claude to interact with your contracts.

### Claude Desktop

Add this to your Claude Desktop MCP configuration:

```json
{
  "mcpServers": {
    "sentinel": {
      "url": "https://sentinel-mcp-server-299005046024.us-west1.run.app/mcp"
    }
  }
}
```

### Other MCP clients

Point any MCP client that supports Streamable HTTP transport to:

```
https://sentinel-mcp-server-299005046024.us-west1.run.app/mcp
```

The server supports OAuth 2.1 with automatic discovery — your client will handle the login flow.

## Authentication

The server uses **OAuth 2.1** (Authorization Code + PKCE). When you connect for the first time:

1. Your MCP client automatically discovers the auth configuration
2. You're redirected to sign in with your Sentinel account (email/password, Google, or Microsoft)
3. If you belong to multiple organizations, you'll be prompted to choose one
4. Once authenticated, your session persists until the token expires

No API keys, secrets, or manual configuration required — just paste the server URL and sign in.

## Requirements

- A [Sentinel](https://general.legal) account with an active organization
- An MCP-compatible client (Claude Code, Claude Desktop, or any client supporting Streamable HTTP)

## Supported file formats

Currently only **`.docx`** (Microsoft Word) files are supported for upload.

## Contract review timeline

Contract reviews involve both AI analysis and human attorney review. Typical turnaround is **a few hours to one business day**, depending on contract complexity. Use `list_contracts` to check status at any time.

## Status meanings

| Status | Description |
|--------|-------------|
| `pending_upload` | Upload slot created, awaiting file |
| `ai_review` | AI is analyzing the contract |
| `attorney_queue` | Queued for human attorney review |
| `attorney_review` | Attorney is actively reviewing |
| `delivered` | Review complete, ready for download |
| `completed` | Review acknowledged by client |

## Support

- **Email**: support@general.legal
- **Website**: [general.legal](https://general.legal)

## About Sentinel

Sentinel is built for any workflow where contracts need professional legal review — whether you're an AI agent negotiating on behalf of a user, a business automating vendor agreements, or a team that wants faster contract turnaround. Every contract is reviewed by a licensed attorney, so you can be confident the results are legally sound.

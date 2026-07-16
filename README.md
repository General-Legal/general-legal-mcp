# General Legal MCP Server

[General Legal](https://general.legal) is an AI-native legal platform. Open a matter with a question or a document and get it reviewed by a combination of AI and a human attorney — directly from your AI assistant.

The General Legal MCP server lets any MCP-compatible client (Claude Code, Claude Desktop, etc.) start matters, submit documents, track review status, answer the attorney's questions, and download reviewed documents through natural conversation.

## Concepts

Work is organized into **matters** (deals). A matter can be started two ways:

- **From a question** — describe what you need and the attorney picks it up. No document required.
- **From a document** — submit a `.docx` or `.pdf` and a matter is created to hold it.

A matter can hold one or more **documents**, and each document can have multiple **versions** (e.g. as a contract is negotiated back and forth).

## What you can do

| Tool | Description |
|------|-------------|
| **start_deal** | Open a new matter from a question or request — no document required |
| **upload_document** | Submit a `.docx`/`.pdf` document — start a new matter, attach to an existing matter (`deal_id`), or add a new version of an existing document (`contract_id`) |
| **confirm_upload** | Confirm an uploaded file and trigger the AI + human review pipeline |
| **list_deals** | List your matters, check review status, and find matter / document / version IDs |
| **list_contracts** | List your document files and find their IDs |
| **list_thread_messages** | Read the attorney's questions and messages on a matter |
| **reply_to_thread** | Reply to the attorney during review |
| **download_contract** | Download a released version with the attorney's comments and redlines |

### Typical workflow

1. **Start a matter** — either:
   - call `start_deal` with your question, or
   - call `upload_document` with a `.docx`/`.pdf` (pass `deal_id` to attach to a matter you already started)
2. **Transfer** (document uploads only) — `upload_document` returns a one-time first-party
   upload link; open it in a browser to pick the file, or `curl -T` the file to it from a
   shell. File bytes never pass through the model, and storage URLs are never exposed
3. **Confirm** (document uploads only) — call `confirm_upload` to kick off the review pipeline, optionally providing context for the legal team
4. **Wait** — the matter goes through AI review, then a human attorney reviews it (this can take several hours)
5. **Track & respond** — call `list_deals` to check status; if the attorney has questions, read them with `list_thread_messages` and answer with `reply_to_thread`
6. **Download** — once the matter is `ready_for_review` or `closed`, call `download_contract` with a version ID to get the reviewed version with the attorney's changes and comments

## Connecting

### Claude Code

```bash
claude mcp add "general-legal" https://sentinel-mcp-server-299005046024.us-west1.run.app/mcp
```

Then start a conversation and ask Claude to interact with your matters.

### Claude Desktop

Add this to your Claude Desktop MCP configuration:

```json
{
  "mcpServers": {
    "general-legal": {
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

The server uses **OAuth 2.1** (Authorization Code + PKCE) with [Clerk](https://clerk.com) as the
authorization server. When you connect for the first time:

1. Your MCP client automatically discovers the auth configuration and registers itself with Clerk (Dynamic Client Registration)
2. You're redirected to sign in with your General Legal account (email/password, Google, or Microsoft)
3. On the consent screen you choose which organization you're acting on behalf of
4. Once authenticated, your session persists until the token expires

No API keys, secrets, or manual configuration required — just paste the server URL and sign in.

> **Only approve a General Legal connection that you started yourself** from your MCP client.
> Never approve a sign-in or "connect" link that someone sent you — a consent screen you didn't
> initiate may be an attempt to gain access to your organization's matters.

## Requirements

- A [General Legal](https://general.legal) account with an active organization
- An MCP-compatible client (Claude Code, Claude Desktop, or any client supporting Streamable HTTP)

## Supported file formats

Documents must be **`.docx`** (Microsoft Word) or **`.pdf`**, up to **20 MiB** through the current
MCP transport. Larger-file support is planned separately.

## Review timeline

Reviews involve both AI analysis and human attorney review. Typical turnaround is **a few hours to one business day**, depending on complexity. Use `list_deals` to check status at any time.

## Matter statuses

`list_deals` reports each matter's status:

| Status | Description |
|--------|-------------|
| `new` | Matter created, not yet in active review |
| `in_progress` | Attorney review underway |
| `awaiting_client` | The attorney has questions for you — answer with `reply_to_thread` |
| `ready_for_review` | Review complete, ready to download |
| `closed` | Matter completed |

## Support

- **Email**: support@general.legal
- **Website**: [general.legal](https://general.legal)

## About General Legal

General Legal is built for any workflow where contracts or legal questions need professional review — whether you're an AI agent negotiating on behalf of a user, a business automating vendor agreements, or a team that wants faster turnaround. Every matter is reviewed by a licensed attorney, so you can be confident the results are legally sound.

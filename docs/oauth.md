# General Legal — OAuth guide

Connect **General Legal** (`general-legal`) to work with legal matters through Claude, ChatGPT, or another OAuth-capable MCP client.

Endpoint: `https://mcp.general.legal/mcp`

## Connect

Add the endpoint as a remote MCP connection in your client. Follow its authorization flow, sign in with your General Legal portal account, and select your organization. An active organization mapped to your General Legal client is required to access matters.

For Claude Code:

```bash
claude mcp add --transport http general-legal https://mcp.general.legal/mcp
```

For clients using a remote-server JSON configuration:

```json
{
  "mcpServers": {
    "general-legal": {
      "type": "streamable-http",
      "url": "https://mcp.general.legal/mcp"
    }
  }
}
```

Authentication uses Clerk OAuth discovery. Only approve a connection you initiated in your MCP client. Do not paste credentials into tool arguments.

## Reconnect and access problems

Use your client's reconnect/sign-in action if authorization expires or is revoked. If the connection reports a missing organization, sign in with the correct organization. If it is not provisioned in General Legal, contact support@general.legal. A temporary authentication-service outage can be retried later.

This endpoint exposes the eight tools below after authentication. For API-key integrations, use the separate [agents guide](agents.md).

## Matter and document tools

A matter (called a deal by the tools) holds a legal question or one or more documents. Each document can have multiple versions.

| Tool | Purpose |
| --- | --- |
| `start_deal` | Open a matter from a question or request. |
| `upload_document` | Prepare a document upload, create a matter, attach to a `deal_id`, or add a version to a `contract_id`. |
| `confirm_upload` | Confirm the transfer and initiate review/notifications. |
| `list_deals` | Find matter status, document IDs, and released version IDs. |
| `list_contracts` | Find document files and contract IDs. |
| `list_thread_messages` | Read attorney questions and matter messages. |
| `reply_to_thread` | Send a message to the attorney. |
| `download_contract` | Get a released version's download link and authenticated MCP resource link. |

## Workflow

1. Call `start_deal` with a legal question, or `upload_document` with a supported filename and a stable `idempotency_key`.
2. For a document, transfer the file through the returned first-party upload link. A person can open it in a browser and select the file, or a shell-capable client can use the returned `curl -T` command. File bytes do not belong in tool arguments.
3. Call `confirm_upload` with the returned contract/version IDs and any context for the legal team.
4. Use `list_deals` to check progress. Use `list_thread_messages` and `reply_to_thread` for attorney questions.
5. When a reviewed version is available, call `download_contract` with its version ID. Open the first-party download link or use the authenticated `gl://contracts/versions/{version_id}` resource if your client supports MCP resources.

Uploads support `.docx`, `.pdf`, `.png`, `.jpg`, `.jpeg`, and `.md`, up to **20 MiB**. Upload links last about one hour; download links last about 15 minutes. Request a fresh download link after expiry.

Pass either `deal_id` or `contract_id` to `upload_document`, never both. Reuse the same idempotency key only for the exact same upload/destination. If you lose the returned IDs, repeat preparation with that key. A pending upload does not appear in `list_deals` until confirmation.

Preparation can return `awaiting_upload`, `processing`, or `confirmed`. A retry while confirmation is running does not issue a replacement upload link. Confirmation deduplicates the main state transition, but notifications may be resent on retries; it is not advertised as idempotent. A reply sends a message without an MCP edit/withdrawal operation. Review actions can start processing and notify the legal team.

## Status and results

| Status | Meaning |
| --- | --- |
| `new` | Created; not yet in active review. |
| `in_progress` | Review underway. |
| `awaiting_client` | The attorney needs a response. |
| `ready_for_review` | Reviewed work is available. |
| `closed` | Completed. |

Reviews involve AI and a human attorney and typically take a few hours to one business day. Check progress when needed rather than polling continuously.

The server uses stateless Streamable HTTP and supports MCP `2025-11-25`. Every tool supplies read-only, destructive, and open-world annotations and an output schema. Structured file results include `ok` and `message`; errors retain that envelope. Text-based matter/list/thread tools use a `result` string. Text fallbacks remain available, and downloads include a resource link.

## Support

Contact support@general.legal. Return to the [connection overview](../README.md).

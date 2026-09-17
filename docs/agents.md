# General Legal for Agents

Use **General Legal for Agents** (`general-legal-agents`) for API-key access and autonomous-agent workflows.

Endpoint: `https://agents-mcp.general.legal/mcp`

## Connect with an API key

Configure your MCP client's HTTP transport with exactly one header:

```http
Authorization: Bearer <GENERAL_LEGAL_API_KEY>
```

Alternatively, use `X-API-Key: <GENERAL_LEGAL_API_KEY>`. Do not send both headers. OAuth tokens are not accepted on this endpoint.

Example connection shape; configure the header using your client's supported mechanism:

```json
{
  "mcpServers": {
    "general-legal-agents": {
      "type": "streamable-http",
      "url": "https://agents-mcp.general.legal/mcp",
      "headers": {
        "Authorization": "Bearer <GENERAL_LEGAL_API_KEY>"
      }
    }
  }
}
```

The placeholder is not automatic environment-variable substitution. Supply your key through the client's configuration or credential integration. Keys are passed in HTTP headers, never business-tool arguments. Existing General Legal API keys work with the same account/data; changing the endpoint does not require a replacement key. Keep keys out of source control and logs.

The [public REST API reference](https://api.general.legal/api/v1/docs) covers direct API access without MCP. For an interactive OAuth connection, use the [OAuth guide](oauth.md).

## Signup and recovery tools

Without credentials, the agents endpoint exposes these tools when signup is enabled. API-key authentication unlocks the eight business tools described below.

| Tool | Purpose |
| --- | --- |
| `get_signup_terms` | Retrieve the current engagement-letter URL. |
| `sign_up` | Start US API-only signup and email a verification code. |
| `complete_signup` | Verify the code, accept the engagement letter, and return the API key once. |
| `resend_signup_code` | Request another code for a valid pending signup. |
| `request_api_key_recovery` | Start mailbox-verified recovery for an API-only account. |
| `complete_api_key_recovery` | Revoke all previous keys and return one replacement. |

### Create an account

1. Call `sign_up` with the company name, contact email, and US billing city, state, and ZIP.
2. Obtain the six-digit code from the mailbox owner. Use `resend_signup_code` if needed.
3. Immediately before completion, call `get_signup_terms`, retrieve and read the current engagement letter, and obtain explicit authority to accept it. Do not rely on cached terms.
4. Call `complete_signup` with the signup ID, code, and `engagement_letter_accepted=true`.
5. Save the returned one-time API key, then reconnect using it in the HTTP header.

Signup and recovery continue to return their keys through tool results on this agents service. There is no new credential-management service or requirement for a particular agent runtime. Use the storage available in your environment to retain the key for later sessions.

### Reconnect or recover a lost key

If a session stops or becomes inactive, reconnect with the saved key. If the key itself is lost, call `request_api_key_recovery` and obtain the emailed verification code. Before completing recovery, obtain authorization to revoke every existing key. Then call `complete_api_key_recovery` with `api_key_replacement_confirmed=true` and save its replacement key immediately. Other integrations using the revoked keys will need the replacement.

Recovery requests deliberately do not reveal whether an account exists. When signup/recovery is disabled, those tools are unavailable; existing-key business access remains available. Invalid, revoked, or suspended credentials cannot access account data.

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

The server uses stateless Streamable HTTP and supports MCP `2025-11-25`. Every tool supplies read-only, destructive, and open-world annotations and an output schema. Structured signup/file results include `ok` and `message`; errors retain that envelope. Text-based matter/list/thread tools use a `result` string. Text fallbacks remain available, and downloads include a resource link.

## Support

Contact support@general.legal. Return to the [connection overview](../README.md).

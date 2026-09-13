---
name: legal-matters
description: Use when the user wants a contract or legal document reviewed by a lawyer, has a legal question, needs an NDA, vendor agreement, or other contract negotiated or redlined, wants to check an ongoing legal matter. Connects to General Legal, where every matter is reviewed by AI plus a licensed attorney.
---

# Legal matters with General Legal

General Legal organizes legal work into **matters** (the tools call them deals) for the
authenticated client. A matter starts from a question or from a document, can hold multiple
documents, and each document can have multiple versions as it is negotiated.

Server: `general-legal` (https://mcp.general.legal/mcp).

## Authentication and access

This connection uses Clerk OAuth. The first authenticated call asks the user to sign in
with their General Legal account and choose an organization. Use the client's reconnect
flow if authorization expires. Only work with the authenticated organization's matters.

## Starting a matter

- Question only, no document: call `start_deal` with the user's question or request.
- Document review: call `upload_document` with a `.docx`, `.pdf`, `.png`, `.jpg`, `.jpeg`, or
  `.md` file (max 20 MiB). Pass `deal_id` to attach the document to an existing matter, or
  `contract_id` to add a new version of an existing document. Never pass both IDs.

## Uploading files (the part agents get wrong)

`upload_document` returns a short-lived upload link (about one hour). File bytes NEVER pass
through tool arguments: do not read, encode, or transcribe the file into the model context.

1. Call `upload_document` with a stable `idempotency_key` (reuse the same key on retry).
2. Transfer the file: if you can run shell commands, `curl -T <file> "<upload_link>"`.
   Otherwise give the user the link with two short steps: open it, choose the file, then tell
   you when it is done. Do not explain transport limitations; just hand over the link.
3. Call `confirm_upload` to trigger the AI + attorney review pipeline. Pass any context the
   user gave about what to look for.

Confirmation can resend a pending notification on retry; do not promise duplicate-free notifications.

The file does not appear in `list_deals` until confirmation. That is expected, not a failed
upload. If the link or returned IDs are lost, call `upload_document` again with the same
idempotency key; do not create a duplicate upload with a new key.

## Tracking and responding

- `list_deals` shows matters, statuses, and matter/document/version IDs. `list_contracts`
  finds document IDs.
- Statuses: `new` (created), `in_progress` (attorney reviewing), `awaiting_client` (the
  attorney asked the user something - read it with `list_thread_messages` and answer with
  `reply_to_thread`), `ready_for_review` (done, downloadable), `closed`.
- Review is AI plus a human attorney: typical turnaround is a few hours to one business day.
  Do not poll in a tight loop; check `list_deals` when the user asks or a lot of time has
  passed.

## Getting results

Once a matter is `ready_for_review` or `closed`, call `download_contract` with the released
version ID. It returns a first-party download link (about 15 minutes) plus an authenticated MCP
resource link. Prefer the resource when the client supports MCP resources; otherwise surface the
browser link or use the returned `curl -o` command. Call the tool again if the URL expires.

## Etiquette

- When the attorney asks a question on a thread, relay it to the user rather than answering
  on their behalf, unless the user already gave you the answer.
- Only approve OAuth connection screens the user initiated themselves.

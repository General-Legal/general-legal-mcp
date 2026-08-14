---
name: legal-matters
description: Use when the user wants a contract or legal document reviewed by a lawyer, has a legal question, needs an NDA, vendor agreement, or other contract negotiated or redlined, or wants to check the status of an ongoing legal matter. Connects to General Legal, where every matter is reviewed by AI plus a licensed attorney.
---

# Legal matters with General Legal

General Legal organizes legal work into **matters** (the tools call them deals) for the
authenticated client. A matter starts from a question or from a document, can hold multiple
documents, and each document can have multiple versions as it is negotiated.

Server: `general-legal` (https://mcp.general.legal/mcp). The first call triggers an OAuth
sign-in; the user signs in with their General Legal account and picks an organization. If they
do not have an account, send them to https://general.legal to create one.

## Starting a matter

- Question only, no document: call `start_deal` with the user's question or request.
- Document review: call `upload_document` with a `.docx` or `.pdf` (max 20 MiB). Pass
  `deal_id` to attach the document to an existing matter, or `contract_id` to add a new
  version of an existing document.

## Uploading files (the part agents get wrong)

`upload_document` returns a one-time upload link. File bytes NEVER pass through tool
arguments.

1. Call `upload_document` with a stable `idempotency_key` (reuse the same key on retry).
2. Transfer the file: if you can run shell commands, `curl -T <file> "<upload_link>"`.
   Otherwise give the user the link with two short steps: open it, choose the file, then tell
   you when it is done. Do not explain transport limitations; just hand over the link.
3. Call `confirm_upload` to trigger the AI + attorney review pipeline. Pass any context the
   user gave about what to look for.

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
version ID to get a download link for the reviewed document with the attorney's redlines and
comments.

## Etiquette

- When the attorney asks a question on a thread, relay it to the user rather than answering
  on their behalf, unless the user already gave you the answer.
- Only approve OAuth connection screens the user initiated themselves.

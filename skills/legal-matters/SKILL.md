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

Clerk OAuth supports sign-in and signup during connection. New clients complete onboarding
and accept the engagement agreement; existing clients use their current organization.
Only work with the authenticated organization's matters. For setup or reconnect issues,
see the [OAuth guide](../../docs/oauth.md).

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

## Catching up on missed updates

When you may have missed attorney messages or released documents, for example after a gap in
your session:

1. Call `list_deals` with no status filter, so closed matters are included, and read every
   page. The output tells you when another page remains.
2. Call `list_thread_messages` for each matter.
3. Message IDs and version IDs never change. Skip the ones you already handled.
4. If you don't know what you already handled, treat everything you can read as new and
   summarize it once for the user.
5. If any page or thread fails to load, tell the user the catch-up is incomplete.

Content withdrawn since your last check may no longer be readable. Before acting on an update,
check the matter's current state.

## Guides and questionnaires

The legal team can release guides and questionnaires on a matter (the tools call them
explainers). A thread message that announces one includes its `explainer_id`.

- `list_explainers` finds the ones released on a matter.
- `open_explainer` shows one as an interactive page where the user reads it and answers its
  questions. In apps that cannot show pages it returns the same text as `get_explainer`.
- `get_explainer` returns the text with any saved answers and a link to the client portal.

The questions are for the user. Do not answer them on the user's behalf; if the app cannot
show the page, give the user the portal link from `get_explainer`.

## Getting results

Once a matter is `ready_for_review` or `closed`, call `download_contract` with the released
version ID. It returns a first-party download link (about 15 minutes) plus an authenticated MCP
resource link. Prefer the resource when the client supports MCP resources; otherwise surface the
browser link or use the returned `curl -o` command. Call the tool again if the URL expires.

## Etiquette

- When the attorney asks a question on a thread, relay it to the user rather than answering
  on their behalf, unless the user already gave you the answer.
- Only approve OAuth connection screens the user initiated themselves.

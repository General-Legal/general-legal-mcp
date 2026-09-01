---
name: legal-matters
description: Use when the user wants a contract or legal document reviewed by a lawyer, has a legal question, needs an NDA, vendor agreement, or other contract negotiated or redlined, wants to check an ongoing legal matter, or needs to sign up for General Legal API access or recover an API key. Connects to General Legal, where every matter is reviewed by AI plus a licensed attorney.
---

# Legal matters with General Legal

General Legal organizes legal work into **matters** (the tools call them deals) for the
authenticated client. A matter starts from a question or from a document, can hold multiple
documents, and each document can have multiple versions as it is negotiated.

Server: `general-legal` (https://mcp.general.legal/mcp).

## Authentication and access

- Portal users: the first authenticated call triggers OAuth. The user signs in with their
  General Legal account and picks an organization.
- API-only clients: send a `glk_` key as `X-API-Key` or `Authorization: Bearer glk_...`, never
  both and never together with OAuth credentials. Keep API keys out of prompts, logs, and files;
  store them in a secret manager.
- No credentials: when autonomous signup is enabled, the server exposes only its signup and
  recovery tools. If it exposes no tools, use OAuth or send the user to https://general.legal.

### Autonomous signup

1. Call `sign_up` with the company name, contact email, and US billing city, state, and ZIP.
2. Ask the mailbox owner for the six-digit verification code. Never retrieve or guess it on the
   user's behalf. Use `resend_signup_code` if the pending signup needs a new code.
3. Immediately before completion, call `get_signup_terms`, retrieve and read the current
   engagement letter, and get the user's explicit authorization to accept it. Do not rely on a
   previously cached URL or version.
4. Call `complete_signup` with the signup ID, code, and
   `engagement_letter_accepted=true`. The API key is shown once: give it to the user to put in a
   secret manager and do not repeat it unnecessarily.

### Recovering an API key

`request_api_key_recovery` emails a code to the verified contact for an API-only account. Ask the
mailbox owner for it. Before calling `complete_api_key_recovery`, explicitly confirm that every
previous API key will be revoked. Pass `api_key_replacement_confirmed=true` only after that
confirmation, then have the user store the one-time replacement key immediately.

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

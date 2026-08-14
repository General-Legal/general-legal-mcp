---
name: company-formation
description: Use when the user wants to form an LLC or C-corp, incorporate a company or startup, set up a Delaware entity, get a shelf company instantly, or check on an in-progress formation. Runs a complete Delaware formation through General Legal, from filing options to executed documents.
---

# Delaware company formation with General Legal

Server: `general-legal-incorporation` (https://incorp-mcp.general.legal/mcp). The endpoint is
authless: no sign-in is needed to start. Each formation is instead protected by its
`formation_id`, a bearer capability token.

## The formation_id is a secret

`start_llc_formation` / `start_c_corp_formation` return the `formation_id` exactly once and it
grants full access to the formation and its documents. Keep it in the conversation, give it to
the user to store securely, and never write it into logs or files.

## Workflow

1. `get_filing_options` with the entity type. If the user has no preference, present an LLC as
   the simpler default and a C-corp when stock or fundraising needs justify it. Present every
   option with its total price and let the user choose - including the `instant` option when
   offered, which hands over a pre-formed shelf company named "GL AgentCo <n>, LLC" within
   minutes of signing (the name cannot be chosen; `company_name` must be omitted).
2. `start_llc_formation` or `start_c_corp_formation` with the intake. The LLC flow is
   sole-member: the founder acts as member, manager, and AI oversight officer. The C-corp flow
   has the founder as sole incorporator.
3. Payment: the response includes a Stripe `checkout_url`. Surface it to the user to open in a
   browser and pay; the agent must not attempt payment itself.
4. Poll `get_status` with the `formation_id`, waiting at least `poll_after_seconds` between
   calls. `waiting_on_human_review: true` means a human is in the loop and polling faster will
   not help. Follow the `next_step` guidance in each response - it covers signing, name
   rejection, and anything else the flow needs from the user.
5. `get_documents` returns short-lived signed download links for the formation documents;
   call it again if the links expire. After signing, only executed copies are returned.
6. `update_formation` changes the company name, but only before document generation; late
   changes are rejected with an explanation.

## After formation

A `completed` status includes `post_formation_guidance` (EIN, bank account, ongoing
obligations). Relay it to the user. For legal work beyond the formation itself - renaming a
shelf company, contracts, legal questions - use the `general-legal` server (see the
legal-matters skill), which connects to General Legal LLP's attorney-reviewed matters.

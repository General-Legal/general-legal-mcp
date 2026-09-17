# Changelog

## 1.3.0 - 2026-09-08

- Split matters connections into Clerk OAuth and API-key agents endpoints.
- Added separate OAuth and agents guides; updated the installed matters skill for OAuth.
- Documented explicit tool annotations, structured output schemas, and confirmation retry behavior.

## 1.2.0 - 2026-09-04

- Added the required pre-checkout Terms of Service, disclaimer, and separate equity buy-in
  disclosure to the company-formation workflow.
- Added path-specific post-formation and EIN guide instructions plus the authoritative guide
  index.

## 1.1.0 - 2026-09-01

- Added API-key authentication plus optional autonomous signup and API-key recovery tools.
- Documented MCP 2.0 structured tool results, tool annotations, and authenticated contract
  resources returned alongside short-lived download links.
- Expanded document uploads to DOCX, PDF, PNG, JPEG, and Markdown, with the current
  idempotent upload/confirm workflow.

## 1.0.0 - 2026-08-13

- This repository is now installable as an [Agent Plugin](https://agent-plugins.org)
  (spec 1.0.0): `npx plugins add General-Legal/general-legal-mcp`.
- `mcp.json` with both remote servers: `general-legal` (matters, contract review) and
  `general-legal-incorporation` (Delaware formation).
- Skills: `legal-matters` and `company-formation`.

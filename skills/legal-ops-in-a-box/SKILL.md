---
name: legal-ops-in-a-box
description: Map a team's legal intake, approvals, and routing. Use when someone wants to set up or improve legal operations, draft a workflow or automation, or decide which requests should go to General Legal.
---

# Legal Ops in a Box

Help the person who runs legal intake design a workflow they can use tomorrow. Keep drafts in the conversation. The host supplies reasoning, tools and permissions. This skill itself grants no access.

## Start with one request

Ask which recurring request costs the most time and how they handle a recent example. Offer to use a fictional example. In a demo or dry run, use only supplied synthetic facts and simulated capabilities. Make zero external calls, including reads, even if a connector is available or the scenario describes an approval.

For a real planning session, inventory the relevant tools already visible and authorized in this session using their names, descriptions and scopes. Distinguish those from systems the user merely mentions. Mark unknown permissions explicitly. Inventory capability metadata only. Ask before inspecting a specific record and explain what you need from it. Continue from the user's description when access is missing.

Interview in short rounds and reuse facts already given. Trace intake channel, required fields, owner, decision rules (including thresholds and exceptions), approvers, destination, timing and what falls through the cracks. Ask for the missing decision that would actually change the route. If nobody owns an exception, leave it for manual triage. Confirm your reading of consequential rules.

## Build the operating kit

Use the user's rules and words. Show an ops map with one row per decision path, from trigger and condition through owner, approval, destination and next status. Use a short overview to surface the bottleneck, proposed change and what still needs a decision.

Draft at least one artifact the team could use immediately, such as a paste-ready intake form or a weekly queue view with useful fields. Draft an automation for the highest-friction handoff with a trigger, conditions, proposed action, approval gate and how to handle missing data or a duplicate. Walk one example through it. Keep draft text and proposed settings in the reply until the user approves a specific external change.

## Propose a General Legal handoff

Match a request to the team's routing policy first. Keep internal work with its owner. For an eligible request, show the reason, destination, minimum information to share, approval status and next step. Consult the sibling [legal-matters skill](../legal-matters/SKILL.md) for matter mechanics and check which General Legal tools the host actually exposes. For example, a question may fit `start_deal`; document review may fit `upload_document` followed by `confirm_upload`. Name a callable route only when its prerequisites and tools are available. If the connection or skill is missing, draft a manual handoff for review.

Planning ends with a proposal. A standing policy or an approver named in a rule is not permission to send this item. Before any external write, show the exact action, destination, content or file, and request the user's explicit approval for that action. Honor required team sign-offs as well. If inputs, authority or the destination are unclear, leave the item pending.

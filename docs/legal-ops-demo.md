# Run the Legal Ops in a Box demo

From a checkout of the demo branch, run this in the repository root:

```sh
npx --yes plugins@1.3.9 discover .
cat skills/legal-matters/SKILL.md skills/legal-ops-in-a-box/SKILL.md
```

The first command should discover one plugin with three skills. The second prints the Matters reference followed by Legal Ops in a Box. For a demo without configuring an account, paste both into a fresh chat with tools disabled and append the opening below in the same message. If your host already has the branch's plugin and offers a skill picker, select `legal-ops-in-a-box` there and use the same opening. To try installation from the checkout in a supported test profile, run `npx --yes plugins@1.3.9 add .` and select the target host. Open a new session or refresh its skills. Installing from the default branch before this change is merged will load the earlier bundle.

**Open the interview**

> Use Legal Ops in a Box for a synthetic dry run. Aster Row's procurement lead loses time chasing legal approvals on vendor requests. Ask me what you need. Use only this fictional scenario. Make no tool calls.

**Answer the interview** (in one message or across turns)

> Requests arrive by email, and Procurement owns a shared tracker. Standard NDAs stay with Procurement. Vendor contracts over $50k can go to General Legal only after our GC approves sending that specific contract. Missing contract value goes back to the requester. No one records who approved the handoff. The queue has a standard NDA, a $65k fictional vendor contract awaiting GC approval, and a vendor contract with no value. For this exercise only, simulate the General Legal upload_document and confirm_upload capabilities. Email and the tracker are described by me and have no connected tools here. Give me the map, overview, intake form, automation draft and a proposed handoff.

**Change one fact**

> Now assume GC approved sending the $65k contract. Show what the handoff would look like. Stay in dry-run mode.

The NDA should stay with Procurement. The $65k item should wait for GC before becoming eligible for a proposed handoff. The missing-value item should return to the requester. Look for an approval field in the intake form, a duplicate check in the automation and a handoff preview without any upload, notification or matter ID.

**Try the missing-connector case**

> Remove the simulated General Legal tools. How would I hand this item to a colleague for review?

Expect a copyable manual handoff using the same approval rule. Discovery and the interview are separate: the installer can recognize a packaged skill, while the UI, invocation syntax and refresh behavior depend on the host. An MCP URL alone provides no copy of this skill.

# PM Delivery Orchestration — Cockpit

A Cowork-based control plane for PM delivery. You manage delivery **outcomes**; the system handles routing, governance, approvals, and traceability — turning a requirement into well-formed Jira tickets through gated, auditable steps, with **no blind approvals**.

One brain (the strong-model Cowork chat), one mirror (this cockpit artifact), one memory (Notion). The chat thinks; the cockpit hosts, governs, and validates; Notion stores state. The chat is disposable — run one fresh chat per workstream; nothing is lost because every decision is persisted to Notion.

## What it does
- **Intake** — paste a rough requirement + links on a workstream row.
- **Working session** — the Coordinator asks the scope-deciding questions up front (not buried), surfaces domain nuances, and writes confirmed decisions + a Stage 1 draft back to Notion.
- **Re-baselined decomposition** — candidate child stories cut against your verified domain model.
- **Gated approvals** — PRD direction → Google Doc → Jira preflight → metadata → parent Jira. Every gate opens a **Review & approve** modal showing the actual content before you confirm. Approval of one action never approves another.
- **Stage 1 sign-off lock** — gates stay locked until a decomposition exists and you explicitly sign off; sign-off auto-invalidates if the draft changes.
- **Breaks Log** — a running regression suite of governance defects you hit, so the system only grows where reality demanded it.

## Install (via the Skill)
This repo ships an auto-provision skill. With a **Notion** connector connected (Drive optional):

1. Add the `skill/pm-delivery-setup` skill to your Cowork skills.
2. Run it: *"set up PM Delivery"*.
3. It creates your four Notion databases, wires `pm-delivery-cockpit.html` to **your** connectors and database IDs, and creates the cockpit artifact.

The HTML here is a **template** — placeholders (`__WS_DS__`, `__NOTION_FETCH__`, …) are filled with your own IDs at setup. Don't hardcode anyone else's IDs.

## Requirements
- Cowork with a **Notion** connector (required) and optionally **Google Drive** (enables the PRD-doc gate).
- Jira/Rovo is optional and used only at the final create gate; without it the cockpit goes all the way to PRD doc + create-readiness and stops before the Jira write.

## Files
- `pm-delivery-cockpit.html` — the templated cockpit artifact.
- `pm-delivery-rules.v1.json` — **rules-as-code** (v1.3): governance, routing, output contracts (incl. the verified 15-section PRD structure `contracts.prd`), and the verified OT event model. Every Coordinator run reads this; the PRD agent writes PRDs in `contracts.prd` format.
- `skill/pm-delivery-setup/SKILL.md` — the setup skill (creates DBs, wires IDs, publishes the artifact, installs the rules file).

## Governance (by design)
Drafting ≠ execution · lookup ≠ application · memory always standalone · every external write needs explicit per-action approval · gates locked until Stage 1 sign-off · the cockpit never generates content (no in-artifact model) — generation stays in the strong-model chat.

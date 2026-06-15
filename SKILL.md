---
name: pm-delivery-setup
description: Set up a personal PM Delivery Orchestration cockpit. Creates the four Notion state-store databases, wires the shared HTML template to the current user's own Notion + Drive connectors, and creates the Cowork artifact. Use when someone says "set up PM Delivery", "install the delivery cockpit", "onboard me to the PM Delivery cockpit", or is a new teammate getting started.
---

# PM Delivery Orchestration — Setup

This skill provisions a working **PM Delivery cockpit** for the current user: one Cowork artifact (the cockpit) backed by four Notion databases (the state store). The cockpit governs delivery as gated, auditable steps — intake → working session → PRD direction → preflight → Jira — with no blind approvals.

The HTML in this repo (`pm-delivery-cockpit.html`) is a **template** with placeholders. Setup = create the user's own databases, replace the placeholders with their IDs and connector tool names, and publish the artifact. Never reuse anyone else's database IDs.

## Governing spec — rules-as-code (read it every run)
This skill ships `pm-delivery-rules.v1.json` (v1.3) — the single source of truth for governance, routing, output **contracts** (including the verified 15-section `contracts.prd` PRD structure), and `domainKnowledge` (the verified OT event model). 

- During setup, copy `pm-delivery-rules.v1.json` into the user's Cowork project/working folder so every chat can read it.
- Every Coordinator run MUST read this file first and obey it. In particular, any PRD it drafts MUST follow `contracts.prd` exactly (15 sections + sub-structure + qualityBars) — never an ad-hoc structure. The Coordinator prompts in the cockpit already instruct this; the file must be reachable for them to comply.

## Prerequisites — confirm before doing anything
1. A **Notion** connector is connected. Drive is optional (enables the "Create Google Doc" gate; without it that gate degrades to manual capture).
2. If Notion isn't connected, stop and ask the user to connect it, then resume.
3. Discover the user's actual connector tool names from the available tools — the Notion and Drive MCP server UUIDs differ per user. You need:
   - `notion-fetch`, `notion-create-pages`, `notion-update-page`, `notion-create-database` (or `update-data-source`)
   - Drive `create_file` (if Drive present)

## Step 1 — Create the four Notion databases
Create these as databases in the user's Notion (parent: a page they choose, or workspace root). Capture each returned **data source ID** (the `collection://<id>` value) — you will inject them into the HTML.

**1. PM Delivery — Workstreams (State Store)** — the core row per delivery item.
Columns:
- `Title` — title
- `Workstream ID` — text
- `Domain` — select: OTI (blue), R2M (purple), Product UI (green)
- `Stage` — select: Stage 0 (gray), Stage 1 (blue), Stage 2 (green)
- `Status` — select: Draft Intake (gray), Drafting (gray), Preflight (yellow), Blocked (red), Approval Needed (orange), Executing (blue), Parent Jira Created (green), Created (green), QA Review (purple), Done (green)
- `Active Agent` — select: PM Delivery Coordinator (blue), R2M Jira Ticket Agent (green), OTI Jira Ticket Agent (orange), PRD Creation Agent (purple), QA / Validation Agent (yellow), Drive Agent (pink), Memory Agent (red)
- `Next Safe Action` · `Coordinator Note` · `Intake Brief` · `Attachments` · `Blocked Actions` · `Created Artifacts` · `Approved Fields` · `Missing Fields` · `Jira Key` · `Gate Data` — all text
- `Last Updated` — last_edited_time

> `Gate Data` is the durable single source of truth for each row: a JSON blob holding gate state, routing, confirmed decisions, working-session Q&A, candidate decomposition, open items + answers, activity, and the Stage-1 sign-off flag. The cockpit reads/writes it.

**2. PM Delivery — Audit** — append-only action log.
Columns: `Event` (title) · `Actor` · `Gate` · `Action` · `Result` (all text) · `Workstream` (relation → Workstreams).

**3. PM Delivery — Breaks Log** — the regression suite of governance defects.
Columns: `Break` (title) · `Where` (select: Intake, PRD content, Google Doc, Preflight, Metadata approval, Parent Jira create, Result capture, Architecture inputs, Decomposition, Tasks, Sub-tasks, QA Tasks, Links, Memory) · `Drift type` (select: Intake-format mismatch, Missing state-store record, Unrecorded create, Field mismatch, Status drift, Approval-scope confusion, Approval too broad, Wanted to skip a step, Unsupported field value, Read-only step did a write, Scope expansion, Premature decomposition, Missing PRD, Missing AC, Orphaned, Needed clarification, Other) · `Severity` (select: High, Medium, Low) · `Expected behavior` · `Actual behavior` · `What happened` · `Evidence` · `Fix idea` (text) · `Regression test candidate` (checkbox) · `Feeds` (multi-select: Scanner, Reconciler, Intake contract, Rules page, Cockpit) · `Workstream` (relation → Workstreams) · `Logged` (created_time).

**4. PM Delivery — Run Log** — records each end-to-end run + the stop rule.
Columns: `Run` (title) · `Run by` (text) · `Role` (select: Owner, Second PM) · `No surprise tickets` · `State matched reality` · `No chat-digging` · `One yes at a time` · `PRD produced` · `All signals passed` · `New drift category appeared` · `Stop-rule met` (all checkbox) · `New categories (which)` (text) · `Run date` (date) · `Workstream` (relation → Workstreams).

## Step 2 — Wire the HTML template
Read `pm-delivery-cockpit.html` and replace every placeholder with the user's real values:

| Placeholder | Replace with |
|---|---|
| `__NOTION_FETCH__` | the user's `notion-fetch` tool name |
| `__NOTION_CREATE_PAGES__` | their `notion-create-pages` tool name |
| `__NOTION_UPDATE_PAGE__` | their `notion-update-page` tool name |
| `__DRIVE_CREATE_FILE__` | their Drive `create_file` tool name (leave as-is / blank if no Drive) |
| `__WS_DS__` | Workstreams data source ID |
| `__AUDIT_DS__` | Audit data source ID |
| `__BREAKS_DS__` | Breaks Log data source ID |
| `__RUN_DS__` | Run Log data source ID |

`const SEEDED=[]` is already empty — leave it; the user's rows are created via the cockpit's **+ New run**.

## Step 3 — Create the Cowork artifact
Create a Cowork artifact (id suggestion: `pm-delivery-command-center`) from the wired HTML, listing the four Notion tools (and Drive create_file if present) as its `mcp_tools`. Confirm it loads showing the connector pills "live" (not the amber "static preview" banner).

## Step 4 — First-run guidance
Tell the user:
1. **+ New run** → paste a rough requirement + any links → it saves to Notion as the Intake Brief.
2. Copy the **Coordinator prompt** into a **new chat** → the strong model runs the working session, writes decisions + draft back to the row.
3. Review the Stage 1 Workbench → **Sign off Stage 1** (gates stay locked until a decomposition exists and you sign off) → walk the gates: each shows a **Review & approve** modal with the actual content before you confirm.
4. One chat per workstream; Notion is the memory; the chat is disposable.

## Governance invariants (do not weaken)
- Every external write (Jira / Drive / memory) needs explicit per-action PM approval. Approval of one never approves another.
- Drafting ≠ execution; lookup ≠ application; memory is always standalone.
- Gates are locked until Stage 1 sign-off; sign-off requires a decomposition to exist and auto-invalidates if the draft changes.
- The cockpit hosts and validates content; it never generates it (no in-artifact model). Generation stays in the strong-model chat.

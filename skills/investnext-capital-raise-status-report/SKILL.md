---
name: investnext-capital-raise-status-report
description: Build a capital raise STATUS / PROGRESS report for one InvestNext raise — committed vs pipeline, paperwork and funding, and what still needs attention. Triggers: "status report", "progress report", "capital raise status", "fundraising dashboard", "pipeline report", "raise update", "offering status", "closing status", "how's the raise going", "where is [raise]", "fundraise progress", "what needs my attention on [raise]". A request to see one raise's progress is enough.
---

# Capital raise status report

A snapshot of **one capital raise**: how far it has progressed (committed / paperwork / funded vs pipeline), then the open items.

Lead with the raise’s totals, then funnel buckets from opportunity `stage` + `paperwork_status` + `funding_status`, then the in-flight roster. A short **Needs attention** list sits with the evidence — countersign, chase funding, send packet — not a separate product.

InvestNext has no subscription stepper and no data-room views. Do not invent either. Use paperwork and funding statuses.

**Read `investnext-data-model` before querying.** Then read **`investnext-report-render`** and open `skills/investnext-report-render/references/example-status-report.html`. This skill owns content and section rules; the render skill owns brand, tokens, and how the file ships.

**Never invent a new layout.** Thin data still copies the example: same section order, same classes. Delete empty sections. Do not redesign.

If this session has no `investnext:investnext-capital-raise-status-report` in the skill list, **keep going**: read this `SKILL.md` and the render skill’s example from the plugin folder. Desktop / Cowork often omit the plugin namespace.

## Output

Canonical deliverable: one self-contained HTML **file** at `outputs/<raise-slug>-status-report.html`. Follow **`investnext-report-render`**: inline `tokens.css` + `report.css` + logomark, keep `<!DOCTYPE html>`, no JavaScript, no artifact, no dark mode. Print-from-file is the export path.

**Omit empty** — never "None on file", "N/A", a bare dash, or "no amount set". Drop the row or section instead.

**Recommendations only.** Never draft or send investor messages.

## Connector gaps — never invent these

- **No subscription stepper.** MCP does not return per-step `mainStates`. Current step = opportunity `stage` plus `paperwork_status` and `funding_status`.
- **No days-in-step.** Opportunity and commitment have `created`, not "entered this stage at". Do not flag Stuck/Stale from `created`. You may show `created` as "Opened" only.
- **No data-room engagement.** No document views or last-access. Omit that section entirely.
- **No app URLs** to opportunities. Names as plain text.
- **Opportunity list is nested on a capital raise**, not a global search. Retrieve an opportunity for paperwork, funding, commitments, and amounts.
- Nested commitments on retrieve are thin (`id`, `account`, `total_amount`). Retrieve a commitment when you need per-vehicle `paperwork_status`.
- Money is already in **major units**. Do not divide by 100.

## Funnel (execution, not CRM colour)

Bucket each opportunity from retrieve fields. Render **only buckets that have members**, in this order:

| Bucket | When |
| --- | --- |
| Pipeline | No commitments. `stage` is a CRM/pipeline name (Prospect, Qualified, Interested, Reviewing, or a custom stage). Dollars = `expected_amount`. |
| Committed | Has commitments. Paperwork `Not Started`. |
| Paperwork in progress | `paperwork_status` is `In Progress`. |
| Needs attention | `paperwork_status` is `Needs Attention` (GP countersign / extra signer). |
| Awaiting funds | Paperwork `Complete` and funding `Not Funded`, `Partially Funded`, or `In Transit`. |
| Funded | `stage` is `Funded`, or funding `Complete`. Dollars = `recorded_amount` (fallback `committed_amount`). |
| Withdrawn / lost | `stage` is `Withdrawn`, `Lost`, or `Unqualified`, or funding `Withdrawn`. Show aside, not in the working funnel. |

Do not invent other buckets. Custom pipeline stage names stay as Pipeline (use the stage name in the roster, not as a new KPI).

**Do not sum pipeline expected with committed.** Headline KPIs use the raise's `total_committed`, `total_funded`, `total_pipeline`, `target_amount` — not a hand-rolled mix of requested + closed.

## Needs attention

Ordered by proximity to close, then dollars:

1. **Needs Attention** paperwork (countersign).
2. Paperwork complete, funds not in (or in transit / partial).
3. Committed, paperwork not started.
4. Paperwork in progress.
5. Large expected pipeline with no commitment (only if dollars are material).

3–7 bullets. Each names the contact, the amount, and the verb (countersign, chase funding, send packet). No email drafts. Omit the section if nothing is open.

## Defaults

- One raise per report. If several titles match, ask.
- Prefer `status=active` when listing raises; confirm before paging every closed raise.
- **Concentration** callout only when one opportunity's `committed_amount` exceeds ~40% of raise `total_committed`.
- Dates `DD Mmm YYYY`.
- Exclude obvious test rows (names with "test" / "admin" / "placeholder", joke mega-amounts) from headlines and the open-item list.

## Sections (in order)

Map each to the example's primitives (`.brand-bar`, `.head`, `.snap-stats`, `.card`, `table.data`, `.badge`, `.kv`).

1. **Brand bar** — InvestNext logomark + wordmark left; `Status report · DD Mmm YYYY` muted right.
2. **Title** — Raise title as `h1`. Subhead: project name · raise `state` · target date if present.
3. **Snap stats** — 2–4 tiles from the raise retrieve: target, committed, funded, pipeline. Optional fifth as a count of Needs Attention. Omit `$0`.
4. **Concentration** — only if >~40%. One card, then omit.
5. **Needs attention** — numbered open items. Omit if the raise has none.
6. **Funnel** — `table.data` of present buckets: bucket, count, dollars. Completed/funded dollars vs in-flight vs pipeline labelled in the row, not summed together.
7. **In-flight roster** — opportunities that are not Funded and not Withdrawn/Lost. Columns with data: contact (or opportunity `name`), account if retrieved, amount (`committed_amount` else `expected_amount`), stage, paperwork badge, funding badge. Omit empty columns.
8. **Footnote** — only if you dropped a section for a connector gap (e.g. no data-room views) or used a non-InvestNext source. Never "Sources: InvestNext connector". Illustrative example files may note they are not live data.

## Workflow

1. **Resolve the raise.** List `capital-raises` (`status=active` if the filter accepts it, `title_search` / `project_name`). If 2+ match, ask. Retrieve the raise (offerings, `total_committed`, `total_funded`, `total_pipeline`, progress).
2. **List opportunities** on that raise. Page if `count` exceeds the page.
3. **Retrieve** each in-flight opportunity (or the page the GP cares about) for `stage`, `paperwork_status`, `funding_status`, `committed_amount`, `recorded_amount`, `expected_amount`, `commitments`, `primary_contact`. Cap a huge roster and say so.
4. **Commitments** only when one opportunity has several vehicles and paperwork might differ — retrieve those commitments.
5. **Bucket, fill the example.** Copy `investnext-report-render/references/example-status-report.html` into the outputs path. Inline `tokens.css` and `report.css` from that skill. Delete unused sections/columns. Do not publish as a claude.ai page.

## Design rules

- **Probe, don't fabricate.** Missing data → ask or omit.
- **Brand:** `investnext-report-render` — type, color, money, dates, logomark, light page. Do not restyle here.
- **No internal IDs** on the page.
- Spell out **InvestNext**.

## Related

- `investnext-data-model` — required before querying
- `investnext-report-render` — brand + `example-status-report.html`
- `investnext-investor-tearsheet` — LP meeting prep; this skill is GP-side on one raise
- `investnext-ir-health` — firm-wide equity housekeeping; this skill stays one raise
- `investnext-property-report-studio` — interactive quarterly packet builder (artifact); not a substitute for this status report

---
name: investnext-ir-health
description: Weekly equity housekeeping across the InvestNext book — unfunded commitments, missing or check payment methods, outstanding accreditation, unsigned paperwork, and quiet investors. Triggers: "equity housekeeping", "Friday checklist", "investor services", "IR health", "IR dashboard", "who hasn't funded", "unfunded commitments", "accreditation outstanding", "missing bank info", "ACH vs check", "who is still on check", "compliance flags", "quiet investors", "who needs follow-up". Live MCP snapshot as a branded HTML file, not an artifact. One-raise call lists use investnext-capital-raise-status-report instead.
---

# Equity housekeeping

A **firm-wide** Friday snapshot for investor services: who to call, what is unsigned, whose accreditation lapsed, and who has not uploaded distribution payment info.

This is the same job as a capital-raise status report, across every active raise — not a live dashboard and not Monet’s artifact mock.

**Read `investnext-data-model` before querying.** Then read **`investnext-report-render`** and open `skills/investnext-report-render/references/example-equity-housekeeping.html`. This skill owns content; the render skill owns brand and how the file ships.

**Never invent a new layout.** Thin data still copies the example: same section order, same classes. Delete empty sections. Do not redesign.

Do **not** publish `references/legacy-artifact-dashboard.html` (or any artifact). That file is a retired mock.

If this session has no `investnext:investnext-ir-health` in the skill list, **keep going**: read this `SKILL.md` and the render skill’s example from the plugin folder.

## Output

Canonical deliverable: one self-contained HTML **file** at `outputs/equity-housekeeping.html`. Follow **`investnext-report-render`**: inline `tokens.css` + `report.css` + logomark, keep `<!DOCTYPE html>`, no JavaScript, no artifact, no dark mode. Print-from-file is the export path.

**Omit empty** — never "None on file", "N/A", a bare dash, or "no amount set". Drop the row or section instead.

**Recommendations only.** Never draft or send investor messages.

## Connector gaps — never invent these

- **Payment method / ACH vs check / last 4.** Not currently on MCP account payloads. Try the account retrieve for any `payment_method`, `ach`, `bank`, or masked last-4 field. If present, show **type** and **last 4 only** — never a full account or routing number. If absent, omit the Payment methods section *or* ask once: “I can include check vs ACH if you drop a payment report.” Join a dropped file on account name / email; still never print a full number. Plaid-tokenized accounts have no full number on InvestNext at all.
- **No last-contacted date.** Quiet = no new deal or commitment in 90+ days, not “we have not emailed them.”
- **No days-in-step** on opportunities. Do not flag stale from `created` alone. You may show `created` as last activity.
- **No data-room engagement.**
- **No app URLs.** Names as plain text.
- Money is already in **major units**. Do not divide by 100.

## Scope

- Whole book, **active raises first**. Ask before paging every closed raise.
- Cap a huge book and say so (e.g. first N raises / first page of flags).
- One-raise “call everyone who hasn’t funded on Fund I” is `investnext-capital-raise-status-report`, not this skill.

## Sections (in order)

Map each to the example's primitives (`.brand-bar`, `.head`, `.snap-stats`, `.card`, `table.data`, `.badge`).

1. **Brand bar** — InvestNext logomark + wordmark left; `Equity housekeeping · DD Mmm YYYY` muted right.
2. **Title** — `Equity housekeeping`. Subhead: `Investor services · all active raises` (or the date range if they asked for a week).
3. **Snap stats** — 2–4 tiles from counts you actually computed. Typical: unfunded dollars, missing payment info, on check, accreditation not current. Omit `$0` and empty tiles.
4. **Unfunded — call these** — commitments where funding is `Not Funded`, `Partially Funded`, or `In Transit`. Columns: contact, raise, account, committed, funded (omit the column if all empty), unfunded. Order by unfunded dollars. This is the firm-wide call list.
5. **Payment methods** — live-position accounts with **Missing** (no method) or **Check**. Columns: contact, account, method badge, last 4 only if returned. Omit ACH rows (they do not need a Friday nudge). Omit the section when the connector has no payment fields and the user did not drop a report.
6. **Accreditation to renew** — accounts with a live position whose `accreditation_status` is expired, pending, invalidated, or needs attention. Columns: account, contact, status badge.
7. **Paperwork flags** — `paperwork_status` of `Needs Attention`, or funded / recorded with paperwork not `Complete` (“Funded, not signed”). Columns: flag, raise, contact, account, amount. Same-account 2+ commitments: keep separate rows.
8. **Quiet investors** — active contacts whose latest commitment or position activity is 90+ days old. Columns: contact, last activity (`DD Mmm YYYY`), days quiet. Omit if you cannot date activity.
9. **Footnote** — only connector gaps you hit (payment method missing from MCP, quiet ≠ last emailed) or a non–InvestNext source (dropped payment report). Never "Sources: InvestNext connector".

## Workflow

1. **List active capital raises.** Prefer `status=active`. Confirm before paging closed raises.
2. **Opportunities and commitments.** For each raise (or the page you can finish), list opportunities; retrieve in-flight rows for `stage`, `paperwork_status`, `funding_status`, amounts, `primary_contact`, nested `commitments` / `account`. Retrieve a commitment when nested rows are thin or an account appears more than once.
3. **Accounts.** From those commitments and from contact/account retrieves: `accreditation_status`, any payment-method field. Filter to accounts that still hold a live position when you can.
4. **Payment methods.** If MCP has no type / last-4, ask once about a payment report; do not block the rest of the file on that answer.
5. **Quiet.** From commitment `created` or position activity dates only. Do not invent outreach dates.
6. **Render.** Copy `investnext-report-render/references/example-equity-housekeeping.html` into `outputs/equity-housekeeping.html`. Inline `tokens.css` and `report.css`. Delete unused sections/columns. Do not publish as a claude.ai page.

## Design rules

- **Probe, don't fabricate.** Missing data → ask or omit.
- **Brand:** `investnext-report-render` — type, color, money, dates, logomark, light page. Do not restyle here.
- **Dates** `DD Mmm YYYY`.
- **Money:** ≥$1B `$X.XB`, ≥$1M `$X.XM`, ≥$1k `$XXk` (or `$XXX,XXX` under $1M in a ledger table), else `$X`. Trim `.0`. Never `$0`.
- **No internal IDs** on the page.
- Spell out **InvestNext**.

## Related

- `investnext-data-model` — required before querying
- `investnext-report-render` — brand + `example-equity-housekeeping.html`
- `investnext-capital-raise-status-report` — one raise, including that raise’s unfunded call list
- `investnext-communications-roster` — deal roster vs a CRM/ESP list
- `investnext-investor-tearsheet` — one-LP meeting prep

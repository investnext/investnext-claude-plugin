---
name: investnext-report-render
description: Default InvestNext design system for shareable collateral from InvestNext data — HTML reports, print/PDFs, statements, one-pagers, briefs, fact sheets, snapshots, packets, and other print/share files. Owns tokens, type, color, money, the logomark, and how the file ships. Use whenever the user asks to make, export, or share a report, PDF, statement, one-pager, fact sheet, briefing, leave-behind, packet, snapshot, or other kept page, unless they ask for a different look (plain markdown, their own brand, unbranded). Not for ephemeral in-chat answers. Consumer skills own content; this skill owns look. Artifact exceptions: property report studio and investor portal demo.
---

# InvestNext report render

The default brand layer for **kept** InvestNext files. Consumer skills (`investnext-investor-tearsheet`, `investnext-capital-raise-status-report`, `investnext-ir-health`, `investnext-communications-roster`, later org charts) own **what** the report says. This skill owns **how it looks** and **how it ships**.

Change colors, type, or chrome **here**. Do not copy a second palette into a consumer skill.

Two principles:

1. **Examples drive layout; this skill drives the invariants.** Match the reference render for the report type. Then apply the conventions below (semantic color, money, dates, light page, no empty states).
2. **The body is authored once.** Same markup for a file. Do not publish a claude.ai artifact for static reports (tearsheet, status report, equity housekeeping, communications roster, statements, one-pagers). Artifact exceptions: property report studio and investor portal demo — those consumer skills say so.

If this session has no `investnext:investnext-report-render` in the skill list, **keep going**: read this `SKILL.md`, `references/tokens.css`, and `references/report.css` from the plugin folder (`skills/investnext-report-render/`). Desktop / Cowork local installs often omit the plugin namespace.

## When to apply (default brand)

Apply this design system to any shareable / print collateral built from InvestNext data **unless the user asks otherwise**.

**Apply** when they want a kept file or page: report, PDF, HTML, statement, one-pager, fact sheet, brief, briefing, snapshot, leave-behind, packet, print, export, download, share.

**Do not apply** to a short in-chat answer (a number, a yes/no, a small markdown table they did not ask to export).

**Override — skip this brand** only if they ask for plain markdown, unbranded output, their own letterhead / brand, or a specific other look.

### Route the content skill

| User intent | Content skill | Open |
| --- | --- | --- |
| Person / LP / prospect meeting prep (tearsheet, tear sheet, one-pager, fact sheet, investor snapshot, LP snapshot, investor brief, meeting brief, meeting prep, investor prep sheet, cheat sheet, leave-behind, "what should I know about [LP]") | `investnext-investor-tearsheet` | `example-tearsheet.html` |
| One raise's progress / pipeline / close (status report, progress report, fundraising dashboard, pipeline report, raise update, offering status) | `investnext-capital-raise-status-report` | `example-status-report.html` |
| Firm-wide equity housekeeping / Friday checklist / missing ACH / accreditation (not one raise) | `investnext-ir-health` | `example-equity-housekeeping.html` |
| Deal roster vs CRM/ESP list (ActiveCampaign, HubSpot, GoHighLevel, …) | `investnext-communications-roster` | `example-communications-roster.html` |
| Quarterly property / investor packet builder | `investnext-property-report-studio` | that skill's `studio.html` (artifact; still these tokens) |
| Investor / LP portal walkthrough | `investnext-investor-portal` | that skill's `portal.html` (artifact; still these tokens) |
| Other report / PDF / statement | this skill only | closest example; compose from primitives |

### Generic collateral (no matching consumer skill)

Still inline `tokens.css` + `report.css` + logomark. Compose from existing primitives. Person-centric → tearsheet example; raise-centric → status example; otherwise assemble `.brand-bar`, `.head`, `.snap-stats`, `.card`, `.kv`, `table.data`. Title the page with the user's words. Brand-bar right: `{their document name} · DD Mmm YYYY`.

Do **not** generate a Schedule K-1, tax form, subscription booklet, or ILPA capital-call / distribution notice. If they asked for an "investor statement", capital account statement, PCAP, or CAS, render a branded **holdings snapshot** from InvestNext positions and metrics (paid-in, distributions, capital account, IRR, equity multiple). Do not title it as an official GAAP PCAP or tax statement.

## Render contract

### 1. Open the matching example

| Report type | Reference (this skill) |
| --- | --- |
| Investor tearsheet (and synonyms above) | `references/example-tearsheet.html` |
| Capital raise status report | `references/example-status-report.html` |
| Equity housekeeping / IR health | `references/example-equity-housekeeping.html` |
| Communications roster | `references/example-communications-roster.html` |
| Other shareable report / statement / PDF | Closest of the examples; then primitives |

Match structure, classes, spacing, and composition. The example is shape, not a cap on rows. Omit fields with no data; never drop live data because the example showed one row.

New sections are allowed if composed from existing primitives (`.brand-bar`, `.head`, `.snap-stats`, `.card`, `.kv`, `table.data`, `.badge`, `.hist`, `ul.bullets`, `ol.actions`).

### 2. Inline the brand files

Self-contained output: one HTML **file**. Inline, in order:

1. Google Fonts for Poppins 400/500, Source Serif 4 500, Roboto 400/500
2. `references/tokens.css`
3. `references/report.css`
4. The cluster logomark from `references/logomark.svg` (20×20) in `.brand`

Do not `@import`. Do not add `@media (prefers-color-scheme: dark)` or `data-theme`. Light page only. Keep `print-color-adjust: exact`.

### 3. Ship as a file

Default: `outputs/<slug>-<report>.html` with `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`. Print-from-file is the export path.

Do **not** publish a claude.ai artifact for these reports (including generic statements and one-pagers). Artifacts strip document tags and often inject dark mode. Print-from-file is how a PDF is made.

## Brand invariants

- **Type:** headings Poppins 500; body Poppins 400 13px; snap-stat values Source Serif 4; numeric table cells Roboto tabular. Weight cap **500** (including `b` / `strong`).
- **Color:** text `grey-900` / `grey-300`; rules `grey-50`; fills `grey-25` / `primary-25`. Text-safe greys are 900–300; 200 and lighter are fill only. Warning **ink** is `#916200` (`--in-warning-ink`), not raw `--in-warning-600` yellow. Success `#19b0a0`. Destructive `#e6024e`.
- **Chrome:** InvestNext logomark + wordmark left; `Report type · DD Mmm YYYY` muted right. Flat white page, max-width 880px. No outer shadow.
- **Dates:** `DD Mmm YYYY`.
- **Money:** ≥$1B `$X.XB`, ≥$1M `$X.XM`, ≥$1k `$XXk` (or `$XXX,XXX` under $1M in a ledger table), else `$X`. Trim `.0`. Never `$0`. Do not sum mixed currencies. MCP money is already major units — do not divide by 100.
- **Empty:** omit the row, tile, or section. Never "N/A", "None on file", or a bare dash.
- **No internal IDs** on the page. Spell out **InvestNext**. Never cite InvestNext / "the connector" as a source. Footnote only non–InvestNext sources; if everything came from InvestNext, omit the footnote.
- **No links** on InvestNext objects. Email and website as plain text.

## Related

- `investnext-data-model` — objects and MCP, before querying
- `investnext-investor-tearsheet` — LP meeting prep (tearsheet, one-pager, brief, snapshot, fact sheet, prep sheet)
- `investnext-capital-raise-status-report` — one-raise status / progress / pipeline / dashboard
- `investnext-ir-health` — firm-wide equity housekeeping (file, not an artifact)
- `investnext-communications-roster` — deal vs CRM/ESP list
- `investnext-property-report-studio` — interactive packet builder (artifact; still uses these tokens)
- `investnext-investor-portal` — LP portal walkthrough (artifact; still uses these tokens)

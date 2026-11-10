---
name: investnext-investor-tearsheet
description: Prep a GP for an LP or prospect meeting from InvestNext (contact, accounts, positions, notes, pipeline) and write a branded HTML briefing. Also known as tearsheet, tear sheet, LP tearsheet, investor tearsheet, one-pager, one pager, fact sheet, investor snapshot, LP snapshot, investor brief, LP brief, meeting brief, meeting prep, investor prep sheet, investor briefing, relationship brief, cheat sheet, leave-behind, holdings summary. Triggers: "tearsheet for [contact]", "one-pager on [investor]", "prep me for my meeting with [LP]", "brief me on [investor]", "what should I know about [LP]", QBR / quarterly review. Supercharged by the InvestNext connector. Not a capital-account statement, PCAP, K-1, or tax form.
---

# Investor tearsheet

Acts as an IR associate prepping the managing partner for an LP or prospect meeting. Pulls InvestNext CRM / cap-table / fundraising data, then **asks** whether to join other connected systems (CRM, email, calendar) before rendering. Writes one **self-contained HTML** file.

GPs ask for this document under many names. Treat all of these as **this skill** (same example layout): tearsheet, tear sheet, LP tearsheet, investor tearsheet, one-pager / one pager on an investor, fact sheet (when it is about a person / LP, not a fund marketing piece), investor snapshot, LP snapshot, investor brief, LP brief, meeting brief, meeting prep, investor prep sheet, investor briefing, relationship brief, cheat sheet, leave-behind, holdings summary, "what should I know about [name]", QBR / quarterly or annual review. A capital-account statement / PCAP / CAS / K-1 is **not** this skill — use `investnext-report-render` generic collateral instead.

**Read `investnext-data-model` before querying.** Then read **`investnext-report-render`** and open `skills/investnext-report-render/references/example-tearsheet.html`. This skill owns content and section rules; the render skill owns brand, tokens, and how the file ships.

**Never invent a new layout.** Thin CRM, missing notes, or a “positions-only” run still copies the example: same section order, same classes (`.brand-bar`, `.head`, `.snap-stats`, `.card`, `table.data`, `.badge`, `.hist`, `.kv`). Delete empty sections. Do not redesign.

If this session has no `investnext:investnext-investor-tearsheet` in the skill list, **keep going**: read this `SKILL.md` and the render skill’s example from the plugin folder. Desktop / Cowork often omit the plugin namespace. Do not skip the tearsheet because the namespaced skill is missing.

## Output

Canonical deliverable: one self-contained HTML **file** at `outputs/<contact-slug>-tearsheet.html`. Follow **`investnext-report-render`**: inline `tokens.css` + `report.css` + logomark, keep `<!DOCTYPE html>`, no JavaScript, no artifact, no dark mode. Print-from-file is the export path. Share that file (or a PDF printed from it).

Match the example: brand bar, title, snap stats, then only the sections that have data. **Omit empty** — never "None on file", "N/A", a bare dash, "No positions yet", or "no amount set". Drop the row or clause instead.

## Connector gaps — never invent these

What MCP does **not** return. Drop the field/section rather than guess:

- **Notes:** `id` + `content` only. No author, no created date. Render note text verbatim; do not fabricate a timeline date.
- **Emails:** `_id`, `body`, `from_` (sender), `to`, `cc`. No subject, no timestamp. If the emails call 403s, omit correspondence.
- **Contact photo, LinkedIn, website:** not on the MCP contact payload. Header has no avatar. Do not add a monogram.
- **AUM / allocation / peer-GP commitments:** not in InvestNext. Only from public web, labelled `· public`, otherwise omit.
- **FoF ownership trees:** not on MCP positions.
- **Opportunity search by contact:** none. Discover pipeline by listing capital raises, then each raise's opportunities, match `primary_contact.id` (and additional contacts on retrieve). Cap the sweep at active raises first; ask before paging every closed raise.

Money on InvestNext MCP is already in **major units**. Do not divide by 100.

## Variants (internal — do not print)

| Variant | Inference | Emphasis |
| --- | --- | --- |
| First meeting / prospect | Zero positions | Org context, discovery agenda, no performance table |
| LP check-in | ≥1 position | Holdings + metrics, recent notes, open pipeline |
| Quarterly / annual review | 2+ positions, or user says QBR / annual | Full holdings walk, cashflow from transactions |
| Ambiguous | No connector, or thin record | Probe: existing LP, prospect, or missing data? |

## Sections (in order)

Map each to the example's primitives (`.brand-bar`, `.head`, `.snap-stats`, `.card`, `table.data`, `.badge`, `.hist`, `.kv`).

**Other systems** (optional) sits after Relationship history when the user opted in and a connector returned something. Omit it when they said InvestNext only, declined, or nothing matched. Never invent campaign history, inbox, or a connector the session does not have.

1. **Brand bar** — InvestNext logomark + wordmark left; `Investor tearsheet · DD Mmm YYYY` muted right.
2. **Title** — Contact name as `h1` (Display Medium). Subhead: title · company · city/state if present. No avatar.
3. **Snap stats** — 2–4 tiles from contact `metrics.total` and pipeline. Typical: paid-in capital, # positions / projects, open pipeline expected, last contribution/distribution if known. Round; omit `$0`.
4. **Meeting details** — only if calendar (or the user) supplied date/attendees/format. Probe for Goals if empty. `.kv` rows.
5. **Executive summary** — 3–6 bullets, most material first. Name distinct **projects** (do not collapse "four Alto projects" when the holdings are different vehicles). Prefer synthesizing contact notes; if you do, omit the notes list in Relationship history (emails can stay).
6. **Investor profile** — `.kv` or tiles: company, title, email, phone, location (city/state in the subhead; street only here if present), account types. Omit empty keys.
7. **Holdings** — `table.data`, one row per position: project, account, paid-in / contributions, distributions, capital account, IRR, equity multiple. Include a column only when at least one row has a value. Print connector IRR and equity multiple **as returned** — including −100% and sub-1.0x on open positions. Do not blank a cell because it "looks wrong." Optional badge on the project name when the position is realized or redeemed. Optional cashflow: last few `transactions` on the largest position for a QBR.
8. **Fundraising pipeline** — opportunities where this contact is primary (or additional). Raise title, stage badge, expected amount, committed / paperwork / funding if retrieved. Omit $0, withdrawn, and empty-amount clauses (keep the row if stage/status exists). Two rows with the same raise title stay two rows when amount or status differs. Omit the whole block if none.
9. **Accounts** — legal vehicles: name, type/subtype, accreditation. Not a second holdings tally (no "4 positions · $950k paid in"). Skip vehicles with no positions. Skip the section if a single obvious individual account is already named in holdings.
10. **Relationship history** — if the summary already synthesized notes, skip the notes list. Otherwise at most 5 **substantive** notes, verbatim, most recent first. Skip stubs: bare URLs, "input notes", headings with no body, whitespace. Emails as one-line "from → to" plus a short body excerpt; no fake dates. If both are empty, omit.
11. **Other systems** — only after a yes on the connector join (below). `.kv` rows: system name → what you found (list membership, last campaign, inbox excerpt, calendar). One row for an **email mismatch** (InvestNext email vs the other system). Omit the whole block if nothing came back. Do not mention a product the user did not select.
12. **Footnote** — only non–InvestNext sources (public web or a joined connector). If everything came from InvestNext, omit the footnote. Never write "Sources: InvestNext connector".

## Workflow

1. **Resolve the contact.** `name__icontains` (or email). If 2+ matches, ask. Then retrieve the contact (accounts + metrics).
2. **Accounts and positions.** For each nested account, retrieve the account (positions, metrics, legal type). Position retrieve if you need full `metrics` / project name. Do not sweep every project's positions.
3. **Transactions (optional).** For QBR or when the user asks "what have they done lately", pull `transactions` on the largest 1–2 positions. Summarize; do not paste the ledger.
4. **Notes and emails.** Nested on the contact. Emails may 403 — continue without them.
5. **Pipeline.** List capital raises (`status=active` if the filter accepts it). For each, list opportunities; keep rows whose `primary_contact.id` matches. Retrieve a matching opportunity for paperwork/funding/commitments. Stop after a reasonable page; say if truncated.
6. **Other systems (blocking question).** Inspect the session’s connectors / tools — not a hardcoded vendor list. Keep only systems that could hold investor context: CRM or ESP (ActiveCampaign, HubSpot, GoHighLevel, Salesforce, Mailchimp, …) or mail / calendar (Gmail, Outlook, Google Calendar, …).
   - **If any of those exist:** use Claude’s AskUserQuestion (the popup). First: “Pull investor context from other connected systems?” Options: **Yes** / **InvestNext only**. If Yes, a second popup (or the same one as a multi-select) listing **only the connectors that are actually connected**, plus InvestNext-only. Do not list a product that is not in the session.
   - **If none exist:** skip the question. Do not invent a menu.
   - For each selected connector, search by InvestNext email first, then name. Record email mismatches. Add a row only when something matched. If a search 403s or returns nothing, omit that system — do not guess.
   - Never add a source the user declined.
7. **Infer variant.** Probe if the shape is thin.
8. **Web research (optional, bounded).** Org news last 90 days, public role. Never personal information. Never paid enrichment. Label public facts `· public`.
9. **Render.** Copy `investnext-report-render/references/example-tearsheet.html` into `outputs/<contact-slug>-tearsheet.html`. Inline `tokens.css` and `report.css` from that skill (the example links them; the deliverable must be self-contained). Replace names/numbers, delete unused sections/columns. Same input → same section order. Do not publish as a claude.ai page.

## Design rules

- **Probe, don't fabricate.** Missing data → ask or omit.
- **Fact-grounded.** IR notes stay verbatim. Do not invent motivations unless the user asked for "thinking partner" notes (then last, tagged inferred).
- **Metrics as returned.** `irr` and `equity_multiple` come from InvestNext cashflows. −100% IRR is the engine result when there are no return cashflows; a 0.10x multiple on an open position is realized distributions ÷ paid-in, not a NAV multiple. Show the number. One summary bullet may explain remaining capital account so a GP is not surprised — never hide the cell. Demo / showcase books often look like this on purpose.
- **Brand:** `investnext-report-render` — type, color, money, dates, logomark, light page. Do not restyle here.
- **Dates** `DD Mmm YYYY`. Drop minute-level timestamps.
- **Money:** ≥$1B `$X.XB`, ≥$1M `$X.XM`, ≥$1k `$XXk` (or `$XXX,XXX` under $1M if the table is a holdings ledger), else `$X`. Trim `.0`. Never `$0`. Don't sum mixed currencies.
- **No internal IDs** in the rendered page. No "InvestNext" as a cited source.
- **No links** on InvestNext objects (no app URLs on MCP). Email and website as plain text. A note that is only a URL still counts as a stub — omit it.
- Spell out **InvestNext**.

## Probing examples

- No CRM: "Is this an existing investor or a prospect? Anything I should know going in?"
- Two contacts: "I see [Name] at [Co A] and [Name] at [Co B]. Which one?"
- Meeting implied, no calendar: "When is the meeting, who is attending, and what's the goal?"
- Other connectors visible: "Pull investor context from other connected systems?" then list only the connectors that exist.

## Related

- `investnext-data-model` — required before querying
- `investnext-report-render` — brand + `example-tearsheet.html`
- `investnext-capital-raise-status-report` — GP-side status on one raise; this skill is LP meeting prep
- `investnext-ir-health` — firm-wide equity housekeeping
- `investnext-communications-roster` — deal roster vs a CRM/ESP list; this skill is one person
- `investnext-property-report-studio` — interactive quarterly packet builder (artifact); not a substitute for this tearsheet
- `investnext-investor-portal` — LP portal walkthrough (artifact); Elena / Harborview sample world matches this skill

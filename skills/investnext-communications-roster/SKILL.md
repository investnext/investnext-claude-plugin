---
name: investnext-communications-roster
description: Reconcile InvestNext investors on a deal against a CRM or email-tool list (ActiveCampaign, HubSpot, GoHighLevel, Mailchimp, Salesforce, or a pasted list) — who is missing from communications, who is on the list but not in the deal, email mismatches, optional co-sponsor groups. Triggers: "communications roster", "CRM sync", "who isn't getting emails", "marketing list", "deal comms", "ActiveCampaign vs InvestNext", "HubSpot list", "GoHighLevel", "who is missing from the campaign", "reconcile investors with [CRM]". First question is always which system to join. Not a tearsheet and not equity housekeeping.
---

# Communications roster

Join **one InvestNext project (deal)** to **one communications system** and show the gaps.

The first move is always: **which system should I connect to?** Do not pull a CRM list or build the file until that is answered.

**Read `investnext-data-model` before querying InvestNext.** Then read **`investnext-report-render`** and open `skills/investnext-report-render/references/example-communications-roster.html`. This skill owns the join; the render skill owns brand and how the file ships.

**Never invent a new layout.** Copy the example. Replace “ActiveCampaign” with the system they chose. Delete empty sections.

If this session has no `investnext:investnext-communications-roster` in the skill list, **keep going**: read this `SKILL.md` and the render skill’s example from the plugin folder.

## Output

Canonical deliverable: one self-contained HTML **file** at `outputs/<project-slug>-communications-roster.html`. Follow **`investnext-report-render`**: inline `tokens.css` + `report.css` + logomark, keep `<!DOCTYPE html>`, no JavaScript, no artifact, no dark mode.

**Omit empty.** Never "N/A" or a bare dash.

**Recommendations only.** Never add or remove CRM contacts, and never send email.

## First question (blocking)

Before any CRM/ESP call and before rendering:

Use Claude’s AskUserQuestion (the popup).

**“Which system should I reconcile this deal against?”**

Build the options from **connectors that actually exist in this session**, plus **I’ll paste a list**. Typical names if they are connected: ActiveCampaign, HubSpot, GoHighLevel, Mailchimp, Salesforce, Klaviyo. Do not list a product that is not connected unless they typed that name themselves.

- If they pick a connector: use it. Then ask which list / tag / campaign / pipeline matches this deal if the tool has more than one.
- If they paste a list: accept CSV / sheet / screenshot of emails.
- If they refuse: stop. Do not default to ActiveCampaign or any other vendor.

This skill is CRM-agnostic. The example uses ActiveCampaign as a placeholder label only.

## Match rules

1. Email, case-insensitive, trim whitespace.
2. Then name (contact name vs CRM name) when emails differ or one side is empty.
3. **Email mismatch** — name matches (or they confirmed it is the same person) but addresses differ. Keep both addresses. Do not silently treat them as the same list membership.
4. Co-sponsor — if InvestNext returns a co-sponsor name or tag on the investor, group the “missing from CRM” and “in both” rows by that tag. Omit grouping when the field is absent.

Never invent a match. When unsure, leave the row on the unmatched side or ask.

## Connector gaps

- InvestNext has **no last-emailed date**.
- Portal email and campaign email are often different — that is the mismatch table, not an error to hide.
- Tags / co-sponsor names may be missing on MCP. Omit that grouping rather than guessing.

## Sections (in order)

1. **Brand bar** — `Communications roster · DD Mmm YYYY`.
2. **Title** — Project name. Subhead: `InvestNext × {their system} · N investors`.
3. **Snap stats** — in both, missing from CRM, CRM only, email mismatch. Omit a $0-style empty tile (omit count 0 tiles).
4. **Missing from {system}** — InvestNext investors on this project with no list match. They may not be receiving deal communications.
5. **Email mismatch** — same person, two addresses.
6. **On {system} only** — on the communications list, not an investor in this project. Useful when a deal is closed and they no longer want those people marketed to.
7. **By co-sponsor** — only if tags/names exist. Otherwise omit.
8. **Footnote** — only if you used a pasted list or dropped a section for a connector gap.

Do not dump the full “in both” roster unless they ask; the counts in snap stats are enough.

## Workflow

1. **Which system?** Blocking question above. Stop until answered.
2. **Resolve the project.** List/search projects. If 2+ match, ask. Retrieve positions (page if needed) → account → contact (name, email).
3. **Pull the other side.** Connector list/tag they named, or the pasted file. If the connector needs a list picker, ask.
4. **Join** with the match rules. Cap a huge book and say so.
5. **Render.** Copy `example-communications-roster.html`. Swap the CRM label everywhere. Inline tokens + CSS. Delete unused sections. Do not publish as a claude.ai page.

## Design rules

- **Probe, don't fabricate.**
- **Brand:** `investnext-report-render`.
- **Dates** `DD Mmm YYYY`.
- **No internal IDs.** Spell out **InvestNext**.
- Email as plain text, not a mailto link.

## Related

- `investnext-data-model` — required before querying
- `investnext-report-render` — brand + `example-communications-roster.html`
- `investnext-investor-tearsheet` — one person, optional other-systems join
- `investnext-ir-health` — firm-wide equity housekeeping, not a CRM join

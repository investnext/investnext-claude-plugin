---
name: investnext-data-model
description: Reference for the InvestNext data model — Contact, Account, Project, Class, Position, Capital Raise, Offering, Opportunity, Commitment, Distribution — how they relate, and how to query them through the InvestNext MCP connector. Use whenever a user asks about InvestNext data (investors, funds, SPVs, capital raises, commitments, distributions) or before calling InvestNext MCP tools. Also use when disambiguating "account" vs "contact" vs "project", or when choosing Position vs Commitment for executed vs in-flight capital.
---

# InvestNext data model

InvestNext is the capital raising and investor management platform used by real estate sponsors, syndicators, and private funds. Each customer has a **group** (firm). MCP results are scoped to the signed-in user's permissions.

Inspect the InvestNext connector's tool list at the start of a session. Tool names may be `list_contacts` / `get_contact` or carry a server prefix. Map tools to the resources below; if a name is missing, say so instead of guessing.

If `investnext:investnext-data-model` is not in the session skill list, still follow this file. Desktop / Cowork local installs often omit the plugin namespace; reading `SKILL.md` from the plugin folder is enough.

## Core objects

**Contact** — a person. Name, email, title, company, phone, address, country, `created`, and rollup `metrics`. A contact can own many Accounts. List filters: `name` / `name__icontains`, `email`, `company`, `city`, `state`, `country`, metrics ordering (`metrics__total__paid_in_capital`). Retrieve includes nested `accounts`. Nested actions: `notes`, `emails`.

**Account** (Investor) — the legal vehicle a Contact invests through (individual, joint, IRA, trust, LLC, …). Fields: `name`, `legal_account_type`, `legal_account_subtype`, `accreditation_status`, `metrics`. Retrieve includes `contact` and `positions`. Filter `contact=<id>` to list a contact's accounts. Do not treat an Account as a person — use Contact for people/org names. **Payment methods** (ACH vs check vs missing, bank last-4) are **not** on current MCP payloads. Full account/routing numbers are never available for Plaid-tokenized accounts. If a future retrieve includes a method type or masked last-4, skills may show those two fields only — never invent bank details.

**Project** (Company) — the GP's fund, SPV, or deal. List: `name`, `type`, `metrics`. Retrieve: `legal_name`, `currency`, `capital_raises`. Nested: `positions`, `classes`, `distributions`. This is the fund vehicle, not the fundraise.

**Class** (Unit class) — a share class inside a Project (Class A, Class B, preferred, …).

**Position** — an executed holding of one Account in one Project. Source of truth for **post-close** economics: `metrics` (contributions, distributions, paid-in capital, capital account, IRR, equity multiple, cash-on-cash, units, value). Nested action: `transactions` (investments, distributions/dividends, transfers, conversions, capital-balance adjustments). FoF (fund-of-funds) nesting is not exposed on MCP position payloads.

**Capital Raise** — an active or closed fundraise on a Project. List: `title`, `project`, `target_amount`, `target_date`, `state`, `progress`, `total_committed`, `total_pipeline`. Retrieve adds `offerings`, `funded_progress`, `total_funded`. Nested: `opportunities`. Filter `status` as a label (`active`, `closed`, …), `title_search`, `project_name`.

**Offering** (Offering round) — terms inside a Capital Raise (which Class is being sold). Nested on the raise as `offerings`, each with `class` / `secondary_class`.

**Opportunity** (Fundraising deal) — a Contact's place in a Capital Raise pipeline. List (via raise): `name`, `primary_contact`, `expected_amount`, `stage`. Retrieve: `additional_contacts`, `commitments`, `committed_amount`, `recorded_amount`, `paperwork_status`, `funding_status`, `created`. There is **no contact-scoped opportunity search** — discover via `capital-raises/{id}/opportunities` and match `primary_contact.id`.

`stage` is computed: with no commitments it is the CRM pipeline stage name (Prospect, Qualified, Interested, Reviewing, or custom); once commitments exist it becomes `Committed`, `Paperwork Complete`, `Funded`, or `Withdrawn` (or `Lost` / `Unqualified` from deal state). `paperwork_status` rolls up commitment package status: `Not Started`, `In Progress`, `Needs Attention` (GP countersign), `Complete`, `Failed`. `funding_status` rolls up: `Not Funded`, `Partially Funded`, `In Transit`, `Complete`, `Withdrawn`. Both are `null` when there are no commitments.

**Commitment** — a signed (or in-progress) commitment on an Opportunity. `amount` / `total_amount`, `recorded_amount`, `paperwork_status` (package label), nested `account` and `offering`. Nested commitments on an opportunity retrieve are thin (`id`, `account`, `total_amount`) — retrieve the commitment for package status per vehicle. **Positions, not Commitments, are executed holdings.** Use Commitment for in-flight / just-closed raise dollars.

**Distribution** — a project-level distribution event (`name`, `exdate`, `payment_date`, `amount`, `cash`, `reinvested`). Per-investor cash shows up as Position `transactions` (dividend) and in position `metrics.distributions`.

**Note** — CRM note on a Contact (and related deals). MCP returns `id` and `content` only — **no author, no created timestamp**.

**Email** — messages associated with a Contact. MCP returns `_id`, `body` (text), `from_` (sender), `to`, `cc`. **No subject, no date.** Requires email permission; 403 means omit the correspondence section.

## Relationship summary

- Contact → Account: one-to-many (one person, several legal vehicles)
- Account → Position: one-to-many
- Position → Project: many-to-one
- Project → Capital Raise: one-to-many
- Capital Raise → Offering: one-to-many
- Capital Raise → Opportunity: one-to-many
- Opportunity → Contact: primary + additional
- Opportunity → Commitment: one-to-many
- Commitment → Account + Offering
- Position transactions: the cashflow ledger after close

## How to reason

- Person or organization name → **Contact**. Legal vehicle → **Account**. Fund / SPV / deal → **Project**. Fundraise → **Capital Raise**. Pipeline row → **Opportunity**. Executed economics → **Position**.
- "How much has this LP in the fund?" → Contact → Accounts → Positions (metrics). Not Opportunities.
- "Where are they in the current raise?" → Capital Raises → Opportunities matching the contact. Then Commitments for paperwork/funding.
- "How is this raise progressing?" → one Capital Raise → its Opportunities. Bucket by `stage` / `paperwork_status` / `funding_status`. Needs Attention paperwork is the GP countersign queue.
- Money on MCP is already in **major units** (dollars, not cents). Do not divide by 100. `metrics` values that look like `{ext: …}` have already been flattened to the display number.
- Metrics live under `metrics.total` on contacts/accounts, and under `metrics.total` plus `metrics.company_*` / `metrics.class_*` on positions/projects. Pass-list includes `contributions`, `distributions`, `paid_in_capital`, `capital_account`, `irr`, `equity_multiple`, `cash_on_cash`, `units`, `value`, `active`, `cost_basis`, `annualized_return`.
- **`equity_multiple` is realized cash ÷ paid-in** (distributions over contributions from the position cashflow ledger). It does **not** add remaining `capital_account`. A 0.10x or 0.23x on an **open** position with capital still in is expected, not a bug. TVPI-style value is a different field (`value` / capital account), not this multiple.
- **`irr` of −1 / −100%** is the cashflow engine result when there are contribution cashflows and no (or almost no) return cashflows — see a contribution-only series. Treat it as the connector number, not as corrupt data. Do not blank it.
- Demo / showcase groups often include write-downs, −100% IRR, and sub-1x multiples on purpose. Print what MCP returns.
- If a field is not on the payload, say it is not available through the connector. Do not pull from Preqin or invent AUM.
- Disambiguate before querying when two contacts share a name.
- Pagination: default 50, max 200. If `count` exceeds the page, keep paging. Never silently drop.

## Tool quirks

- `filters` describes every list filter — call it when unsure of a param name.
- `whoami` returns the signed-in user, email, profile type, group.
- Opportunity is retrieve-only as a collection; list them off a capital raise.
- Position list is off a **project**, not a global search. Get a contact's positions from each Account retrieve.
- Notes and emails are contact nested actions, not a global activity feed.
- Ordering on contacts/accounts/projects accepts `metrics__total__<metric>` (and `-` for desc).

## Related

- `investnext-report-render` — brand layer for kept HTML reports
- `investnext-investor-tearsheet` — LP meeting prep
- `investnext-capital-raise-status-report` — one-raise status / progress report
- `investnext-ir-health` — firm-wide equity housekeeping
- `investnext-communications-roster` — deal roster vs a CRM/ESP list
- `investnext-property-report-studio` — quarterly packet builder (artifact)


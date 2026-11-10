# InvestNext plugin for Claude

Skills that teach Claude how to work with InvestNext data through the [InvestNext MCP](https://www.investnext.com/platform/mcp/) connector, and how to turn that data into branded, self-contained HTML reports. The plugin bundles the `investnext` connector (`https://mcp.investnext.com/mcp`), so installing it gives you the tools and the workflows together.

InvestNext MCP is a paid add-on. If Claude reports that access was denied, see [Connecting InvestNext MCP to Claude](https://support.investnext.com/en/articles/17221309-connecting-investnext-mcp-to-claude).

## Skills

- `investnext-data-model`: the InvestNext objects (Contact, Account, Project, Position, Capital Raise, Opportunity, Commitment, Distribution, and more), how they relate, and how to query them through MCP. The other skills read it before querying.
- `investnext-report-render`: the InvestNext brand layer for kept files: tokens, type, color, money and date formats, the logomark, and the reference layouts every report copies.
- `investnext-investor-tearsheet`: meeting prep for an LP or prospect: holdings, pipeline, accounts, and relationship history in one briefing.
- `investnext-capital-raise-status-report`: progress on one capital raise: committed, paperwork, funding, pipeline, and what needs attention.
- `investnext-ir-health`: firm-wide equity housekeeping: unfunded commitments, accreditation to renew, paperwork flags, and quiet investors.
- `investnext-communications-roster`: reconciles the investors on a deal with a CRM or email list and shows who is missing, extra, or has a different email.

Reports are written as one HTML file under `outputs/`. Print the file to make a PDF.

## Install

### Claude Code

```
/plugin install investnext --marketplace investnext/investnext-claude-plugin
```

Or from your shell:

```
claude plugin marketplace add investnext/investnext-claude-plugin
claude plugin install investnext@investnext
```

Then run `/mcp` and authenticate the `plugin:investnext:investnext` server with your InvestNext account.

### claude.ai and Cowork

Go to **Customize > Plugins > Add > Add marketplace** and enter `investnext/investnext-claude-plugin`, then install **InvestNext** and connect the bundled connector from the plugin's **Connectors** tab.

## Try it

- "Prep me for my meeting with [investor]."
- "How's the [raise] going?"
- "Run the Friday equity housekeeping checklist."
- "Which investors on [deal] aren't on our HubSpot list?"

## Development

Validate before pushing:

```
claude plugin validate . --strict
```

Load a working copy for one session with `claude --plugin-dir .`.

## Privacy

The plugin sends requests only to the InvestNext MCP server. See the [InvestNext privacy policy](https://www.investnext.com/legal/privacy-policy/).

## License

[Apache 2.0](LICENSE)

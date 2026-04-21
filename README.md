# OSOME Accounting Skills

Agent skills for accounting and bookkeeping, powered by the [OSOME MCP Server](https://mcp.osome.com/mcp).

## Prerequisites

Connect the OSOME MCP server to your Claude environment:

```
https://mcp.osome.com/mcp
```

This provides access to accounting tools for your OSOME-managed companies.

## Available Skills

| Skill | Description |
|-------|-------------|
| [financial-health-check](./skills/financial-health-check) | Get a snapshot of your company's financial position |
| [cash-flow-visibility](./skills/cash-flow-visibility) | Understand where money is coming from and going |
| [receivables-payables-tracker](./skills/receivables-payables-tracker) | Track who owes you and who you owe |
| [ledger-deep-dive](./skills/ledger-deep-dive) | Query specific transactions and accounts |
| [corporate-structure-overview](./skills/corporate-structure-overview) | View directors, shareholders, and subsidiaries |
| [tax-compliance-snapshot](./skills/tax-compliance-snapshot) | Check GST/VAT obligations and prepare for tax filing |

## Installation

### Claude Code (Recommended)

**Step 1:** Add the OSOME marketplace:

```
/plugin marketplace add OsomePteLtd/skills
```

**Step 2:** Install the plugin:

```
/plugin install osome-accounting-skills@osomepteltd-skills
```

**Step 3:** Reload plugins:

```
/reload-plugins
```

### Manual Installation

Clone and copy skills to your personal skills directory:

```bash
git clone https://github.com/OsomePteLtd/skills.git osome-skills
cp -r osome-skills/skills/* ~/.claude/skills/
```

Or copy to a project's `.claude/skills/` directory for project-scoped installation.

### Team/Project Configuration

Add to your project's `.claude/settings.json` for automatic team installation:

```json
{
  "extraKnownMarketplaces": {
    "osome-skills": {
      "source": {
        "source": "github",
        "repo": "OsomePteLtd/skills"
      }
    }
  }
}
```

## Usage

Once installed, skills are automatically triggered by relevant questions:

- "Am I making money?" - triggers `financial-health-check`
- "Where is my money going?" - triggers `cash-flow-visibility`
- "Who hasn't paid me?" - triggers `receivables-payables-tracker`
- "What did I spend on software?" - triggers `ledger-deep-dive`
- "Who owns the company?" - triggers `corporate-structure-overview`
- "How much GST do I owe?" - triggers `tax-compliance-snapshot`

You can also invoke skills directly:

```
/osome-accounting-skills:financial-health-check
/osome-accounting-skills:cash-flow-visibility
/osome-accounting-skills:tax-compliance-snapshot
```

## MCP Tools

These skills orchestrate tools from the OSOME MCP server:

### Company
- `list-companies` - List your companies
- `get-company` - Company details and fiscal year

### Accounting
- `get-chart-of-accounts` - Chart of accounts by jurisdiction
- `search-transactions` - Search transactions with filters
- `list-documents` - Accounting documents
- `get-document` - Document details
- `get-journal-entries` - Journal entries
- `get-bank-accounts` - Bank accounts and balances

### Reports
- `get-balance-sheet` - Balance sheet
- `get-profit-and-loss` - Profit & loss statement
- `get-cash-flow-statement` - Cash flow statement
- `get-trial-balance` - Trial balance
- `get-aged-payables` - Accounts payable aging
- `get-aged-receivables` - Accounts receivable aging

### Compliance
- `get-corporate-subsidiaries` - Subsidiary companies
- `get-directors-shareholders` - Directors and shareholders

## Multi-Jurisdiction Support

Skills automatically adapt to your company's jurisdiction:

| Jurisdiction | Tax System | Currency |
|--------------|------------|----------|
| Singapore | GST (9%) | SGD |
| Hong Kong | No GST/VAT | HKD |
| United Kingdom | VAT (20%) | GBP |
| UAE | VAT (5%) | AED |

## License

Proprietary - OSOME Pte Ltd. See [LICENSE](./LICENSE) for details.

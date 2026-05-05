# OSOME Accounting Skills

Agent skills for accounting and bookkeeping, powered by the [OSOME MCP Server](https://mcp.osome.com/mcp).

## Prerequisites

Connect the OSOME MCP server to your Claude/OpenCode environment:

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
| [corporate-structure-overview](./skills/corporate-structure-overview) | View directors and subsidiaries |
| [tax-compliance-snapshot](./skills/tax-compliance-snapshot) | Check GST/VAT obligations and prepare for tax filing |

## Installation

### Claude Code

Install directly from GitHub:

```
/plugin install github:OsomePteLtd/skills
```

### OpenCode

Copy skills to your config directory:

```bash
git clone https://github.com/OsomePteLtd/skills.git osome-skills
cp -r osome-skills/skills/* ~/.config/opencode/skills/
```

Or for project-scoped installation:

```bash
cp -r osome-skills/skills/* .opencode/skills/
```

### Manual Installation (Any Agent)

Copy skills to your agent's skills directory:

```bash
git clone https://github.com/OsomePteLtd/skills.git osome-skills

# For Claude Code
cp -r osome-skills/skills/* ~/.claude/skills/

# For OpenCode
cp -r osome-skills/skills/* ~/.config/opencode/skills/

# For any agent supporting .agents/ convention
cp -r osome-skills/skills/* ~/.agents/skills/
```

## Usage

Once installed, skills are automatically triggered by relevant questions:

- "Am I making money?" → `financial-health-check`
- "Where is my money going?" → `cash-flow-visibility`
- "Who hasn't paid me?" → `receivables-payables-tracker`
- "What did I spend on software?" → `ledger-deep-dive`
- "Who are my directors?" → `corporate-structure-overview`
- "How much GST do I owe?" → `tax-compliance-snapshot`

## MCP Tools

These skills orchestrate tools from the OSOME MCP server:

### Company
- `list-companies` - List your companies
- `get-company` - Company details and fiscal year

### Accounting
- `get-chart-of-accounts` - Chart of accounts by jurisdiction
- `search-transactions` - Search transactions with filters
- `list-documents` - Accounting documents (ID and name only)
- `get-document` - Document details (ID and name only)
- `upload-document` - Two-step document upload for PDFs, images, and spreadsheets
- `get-journal-entries` - Journal entries (ID, description, amount)
- `get-bank-accounts` - Bank accounts and balances

### Reports
- `get-balance-sheet` - Balance sheet report
- `get-profit-and-loss` - Profit & loss statement
- `get-cash-flow-statement` - Cash flow statement
- `get-trial-balance` - Trial balance
- `get-aged-payables` - Accounts payable aging
- `get-aged-receivables` - Accounts receivable aging

### Compliance
- `get-corporate-subsidiaries` - Subsidiary companies
- `get-directors-shareholders` - Directors (name, role, jurisdiction)

## Multi-Jurisdiction Support

Skills work with companies in these jurisdictions:

| Jurisdiction | Tax System | Currency |
|--------------|------------|----------|
| Singapore | GST (9%) | SGD |
| Hong Kong | No GST/VAT | HKD |
| United Kingdom | VAT (20%) | GBP |
| UAE | VAT (5%) | AED |

## License

Proprietary - OSOME Pte Ltd. See [LICENSE](./LICENSE) for details.

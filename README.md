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

## Installation

### Claude Code

```bash
# Install all skills
claude skills add github:OsomePteLtd/skills

# Or install a specific skill
claude skills add github:OsomePteLtd/skills/skills/financial-health-check
```

### Manual Installation

Copy the desired skill folder to your `.claude/skills/` directory:

```bash
cp -r skills/financial-health-check ~/.claude/skills/
```

## Usage

Once installed, skills are automatically triggered by relevant questions:

- "How is my business doing financially?" - triggers `financial-health-check`
- "Where is my money going?" - triggers `cash-flow-visibility`
- "Who owes me money?" - triggers `receivables-payables-tracker`
- "What did I spend on marketing?" - triggers `ledger-deep-dive`
- "Who are my company directors?" - triggers `corporate-structure-overview`

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

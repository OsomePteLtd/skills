# OSOME Accounting and Invoicing Skills

Agent skills for accounting, bookkeeping, and invoicing, powered by the OSOME MCP server.

## Prerequisites

Connect the OSOME MCP server to your Claude/OpenCode environment as a Streamable HTTP MCP endpoint:

```
https://mcp.osome.com/mcp
```

The MCP endpoint handles JSON-RPC over `POST /mcp`. Authentication is via OAuth bearer token; MCP clients can discover OAuth metadata from `/.well-known/oauth-protected-resource` and `/.well-known/oauth-authorization-server`. Accounting read-only tools require `openid`; invoicing reads require `invoice:read`; invoicing writes require `invoice:write`; document upload additionally requires `company:documents:write`.

## Available Skills

| Skill | Description |
|-------|-------------|
| [financial-health-check](./skills/financial-health-check) | Get a snapshot of your company's financial position |
| [cash-flow-visibility](./skills/cash-flow-visibility) | Understand where money is coming from and going |
| [receivables-payables-tracker](./skills/receivables-payables-tracker) | Track who owes you and who you owe |
| [ledger-deep-dive](./skills/ledger-deep-dive) | Query specific transactions and accounts |
| [corporate-structure-overview](./skills/corporate-structure-overview) | View directors and subsidiaries |
| [tax-compliance-snapshot](./skills/tax-compliance-snapshot) | Check GST/VAT obligations and prepare for tax filing |
| [invoice-creator](./skills/invoice-creator) | Create, update, submit, duplicate, preview, and generate invoice PDFs |
| [invoice-revenue-insights](./skills/invoice-revenue-insights) | Analyze invoice revenue, pending invoices, overdue invoices, and collections |

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
- "Create an invoice for Acme" → `invoice-creator`
- "Which invoices are overdue?" → `invoice-revenue-insights`

## MCP Tools

These skills orchestrate tools from the OSOME MCP server:

### Company
- `list-companies` - List your companies
- `get-company` - Company details and fiscal year

### Accounting
- `get-chart-of-accounts` - Chart of accounts by jurisdiction
- `search-transactions` - Search transactions with optional date range and limit (newest first; default 50)
- `list-documents` - Accounting documents (ID and name only)
- `get-document` - Document details (ID and name only)
- `prepare-document-upload` - Step 1 for document upload; returns a presigned POST target and `uploadToken`
- `complete-document-upload` - Step 3 for document upload; creates the OSOME document after bytes are uploaded to the presigned target
- `get-journal-entries` - Journal entries (ID, description, amount)
- `get-bank-accounts` - Bank accounts and balances

Document upload supports PDF, JPG/JPEG, PNG, CSV, XLS, and XLSX files up to 50 MB. Do not send file bytes or `contentBase64` to MCP: upload the file directly to the presigned POST target returned by `prepare-document-upload`, then call `complete-document-upload` with the same metadata and `uploadToken`.

### Invoicing
- `list-customers` - List invoice customers for a company
- `create-customer` - Create an invoice customer
- `create-invoice-draft` - Create a draft invoice
- `update-invoice-draft` - Update a draft invoice with customer, line items, dates, currency, notes, and payment methods
- `submit-invoice` - Finalize and send a draft invoice
- `generate-invoice-pdf` - Generate invoice document links; submits drafts first when needed
- `preview-invoice` - Get invoice data without mutation
- `get-invoice-document-links` - Get generated invoice document preview and original URLs
- `list-invoices` - List invoices with status, search, and pagination filters
- `duplicate-invoice` - Duplicate an invoice into a new draft
- `create-credit-note` - Create a draft credit note for an issued invoice
- `submit-credit-note` - Submit and apply a credit note
- `get-credit-note` - Get a credit note and safe invoice summary

Invoice reads require `invoice:read`; invoice writes require `invoice:write`. `submit-invoice`, `generate-invoice-pdf` for drafts, and `submit-credit-note` finalize externally visible accounting documents, so agents should confirm final details first. The MCP server does not expose invoice deletion or pro-forma invoice creation.

### Reports
- `get-balance-sheet` - Balance sheet report
- `get-profit-and-loss` - Profit & loss statement
- `get-cash-flow-statement` - Cash flow statement
- `get-trial-balance` - Trial balance
- `get-aged-payables` - Accounts payable aging
- `get-aged-receivables` - Accounts receivable aging
- `get-aged-receivables-v2` - Aged receivables v2 report
- `get-overdue-invoices` - Overdue receivables derived from aged receivables v2
- `get-invoice-pending-summary` - Awaiting, future, and overdue invoice pending totals
- `get-profit-and-loss-v2` - Profit & loss v2 report
- `get-revenue-summary` - Founder-friendly revenue summary from profit & loss v2

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

---
name: financial-health-check
description: Get a snapshot of your company's financial position. Use when the user asks about financial health, profitability, balance sheet, or wants a business overview. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Financial Health Check

Get an instant snapshot of your company's financial position.

## When to Use

Invoke this skill when the user asks:
- "How is my business doing financially?"
- "How is my business doing?"
- "What's my financial health?"
- "Company financial overview"
- "Give me a financial summary"
- "How profitable am I?"
- "Am I making money?"
- "Did I make a profit?"
- "What's my runway?"
- "How long can I survive?"
- "Quick financial overview"
- "Bottom line this month/quarter/year"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Let user select company (if multiple) |
| 2 | `get-company` | Get fiscal year end |
| 3 | `get-balance-sheet` | Assets, liabilities, equity snapshot |
| 4 | `get-profit-and-loss` | Revenue, expenses, net profit |
| 5 | `get-bank-accounts` | Cash positions across all accounts |

## Data Available

**From `get-company`:**
- Company ID and name
- Next fiscal year end date

**From `get-balance-sheet`:**
- Report with headings and line items
- Assets, liabilities, equity totals (in report format)

**From `get-profit-and-loss`:**
- Report with headings and line items
- Revenue, expenses, net profit (in report format)

**From `get-bank-accounts`:**
- Account name, bank name, currency
- Balance per account
- Account type and active status

## Execution Flow

1. Call `list-companies` to get available companies (returns `[{ id, name }]`)
2. If multiple companies, ask user to select one
3. Call `get-company` with selected company ID
   - Extract: `nextFiscalYearEnd` to determine date range
4. Call `get-balance-sheet` with company ID and date range
5. Call `get-profit-and-loss` with company ID and date range (both dates required)
6. Call `get-bank-accounts` with company ID
7. Parse report data and synthesize summary

## Report Structure

Reports return data in this format:
```json
{
  "report": {
    "headings": ["Account", "Amount"],
    "lines": [["Revenue", 890000], ["Expenses", 765000]]
  }
}
```

Parse the report to extract key figures.

## Output Format

Present a natural language summary including:

### Key Metrics
- **Net Profit/Loss:** [amount] for [period]
- **Gross Profit Margin:** [percentage] (if calculable from report)
- **Total Cash:** [sum across bank accounts]

### Financial Position
- **Total Assets:** [from balance sheet report]
- **Total Liabilities:** [from balance sheet report]
- **Net Equity:** [assets - liabilities]

### Cash Runway
If expenses are available from P&L:
`Total Cash / (Monthly Operating Expenses)`

## Example

**User:** "How is my business doing?"

**Response:**
> Based on your financial reports for ACME Pte Ltd:
>
> **Profitability (YTD):**
> - Revenue: SGD 890,000
> - Expenses: SGD 765,000
> - Net Profit: SGD 125,000 (14% margin)
>
> **Cash Position:**
> Total across 2 bank accounts: SGD 245,000
> - DBS Current: SGD 180,000
> - OCBC Savings: SGD 65,000
>
> **Balance Sheet:**
> - Total Assets: SGD 580,000
> - Total Liabilities: SGD 260,000
> - Equity: SGD 320,000
>
> **Runway:** At ~SGD 30K/month burn rate, approximately 8 months of cash.

## Error Handling

| Scenario | Response |
|----------|----------|
| No companies found | "You don't have access to any companies. Please check your permissions." |
| Balance sheet empty | "No financial data available for this period. Has bookkeeping been completed?" |
| Missing fiscal year | Use calendar year (Jan 1 - Dec 31) as fallback |
| Report parsing fails | Show raw report data and explain limitation |

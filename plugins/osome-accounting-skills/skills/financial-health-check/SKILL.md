---
name: financial-health-check
description: Get a snapshot of your company's financial position. Use when the user asks about financial health, profitability, balance sheet, or wants a business overview. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Financial Health Check

Get an instant snapshot of your company's financial position.

## When to Use

Invoke this skill when the user asks:
- "How is my business doing financially?"
- "What's my financial health?"
- "Company financial overview"
- "Give me a financial summary"
- "How profitable am I?"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Let user select company (if multiple) |
| 2 | `get-company` | Get fiscal year end, currency, jurisdiction |
| 3 | `get-balance-sheet` | Assets, liabilities, equity snapshot |
| 4 | `get-profit-and-loss` | Revenue, expenses, net profit |
| 5 | `get-bank-accounts` | Cash positions across all accounts |

## Execution Flow

1. Call `list-companies` to get available companies
2. If multiple companies, ask user to select one
3. Call `get-company` with selected company ID
   - Extract: `nextFiscalYearEnd`, branch (SG/HK/UK)
   - Determine date range: fiscal year start to today (or specified period)
4. Call `get-balance-sheet` with company ID and date range
5. Call `get-profit-and-loss` with company ID and date range
6. Call `get-bank-accounts` with company ID
7. Synthesize results into natural language summary

## Jurisdiction Handling

| Branch | Consideration |
|--------|---------------|
| SG | Include GST liability/receivable in working capital |
| UK | Include VAT liability/receivable in working capital |
| HK | No GST/VAT - simpler calculation |

## Output Format

Present a natural language summary including:

### Key Metrics
- **Net Profit/Loss:** [amount] for [period]
- **Gross Profit Margin:** [percentage]
- **Current Ratio:** [current assets / current liabilities]
- **Total Cash:** [sum across bank accounts]

### Financial Position
- **Total Assets:** [amount]
- **Total Liabilities:** [amount]
- **Net Equity:** [amount]

### Cash Runway
If expenses are consistent, estimate months of runway:
`Total Cash / (Monthly Operating Expenses)`

### Period Comparison (if available)
- Revenue change: +/- [percentage]
- Profit change: +/- [percentage]

## Example

**User:** "How is my business doing?"

**Response:**
> Your company ACME Pte Ltd is in good financial health.
>
> **Profitability:** Net profit of SGD 125,000 this fiscal year on revenue of SGD 890,000 (14% margin).
>
> **Liquidity:** Current ratio of 2.1 - healthy coverage of short-term obligations. Total cash across 2 bank accounts: SGD 245,000.
>
> **Balance Sheet:** Total assets SGD 580,000 with equity of SGD 320,000. GST payable of SGD 12,500 due next quarter.
>
> At current burn rate, you have approximately 8 months of cash runway.

## Error Handling

| Scenario | Response |
|----------|----------|
| No companies found | "You don't have access to any companies. Please check your permissions." |
| Balance sheet empty | "No financial data available for this period. Has bookkeeping been completed?" |
| Missing fiscal year | Use calendar year (Jan 1 - Dec 31) as fallback |

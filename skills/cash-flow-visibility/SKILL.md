---
name: cash-flow-visibility
description: Understand where money is coming from and going to. Use when the user asks about cash flow, spending, expenses, or money movement. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Cash Flow Visibility

Understand where money is coming from and going to.

## When to Use

Invoke this skill when the user asks:
- "Where is my money going?"
- "Cash flow analysis"
- "Show me my cash movements"
- "What are my biggest expenses?"
- "How is cash flowing through my business?"
- "Analyze my spending"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select company |
| 2 | `get-company` | Company context and fiscal year |
| 3 | `get-cash-flow-statement` | Operating, investing, financing breakdown |
| 4 | `search-transactions` | Recent transactions (limit 50, newest first) |
| 5 | `get-bank-accounts` | Current bank balances |

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` to get context
3. Determine date range:
   - If user specifies: use their range
   - Default: last 3 months or current quarter
4. Call `get-cash-flow-statement` with date range
5. Call `search-transactions` with `limit=50` for recent activity
6. Call `get-bank-accounts` for current positions
7. Analyze and present findings

## Output Format

### Cash Flow Summary

**Operating Activities:** [net amount]
- Cash from customers: [amount]
- Cash paid to suppliers: [amount]
- Cash paid for salaries: [amount]
- Other operating: [amount]

**Investing Activities:** [net amount]
- Equipment purchases: [amount]
- Asset sales: [amount]

**Financing Activities:** [net amount]
- Loans received/repaid: [amount]
- Dividends: [amount]

### Top Expense Categories
1. [Category]: [amount] ([percentage] of total)
2. [Category]: [amount] ([percentage] of total)
3. [Category]: [amount] ([percentage] of total)

### Largest Recent Transactions
| Date | Description | Amount | Type |
|------|-------------|--------|------|
| ... | ... | ... | In/Out |

### Net Cash Position
- Opening balance: [amount]
- Net change: [amount]
- Closing balance: [amount]

## Example

**User:** "Where is my money going this quarter?"

**Response:**
> Here's your cash flow analysis for Q3 2024:
>
> **Operating Cash Flow:** SGD -45,000
> - Customer receipts: +SGD 280,000
> - Supplier payments: -SGD 180,000
> - Salaries & wages: -SGD 120,000
> - Other operating: -SGD 25,000
>
> **Top Expenses:**
> 1. Salaries: SGD 120,000 (37%)
> 2. Software & Tools: SGD 45,000 (14%)
> 3. Marketing: SGD 38,000 (12%)
>
> **Largest Transactions:**
> - Sep 15: AWS Singapore - SGD 12,500 (Cloud hosting)
> - Sep 1: Payroll Run - SGD 42,000 (Salaries)
> - Aug 28: Google Ads - SGD 8,500 (Marketing)
>
> **Cash Position:** Started Q3 with SGD 290,000, now at SGD 245,000.

## Error Handling

| Scenario | Response |
|----------|----------|
| Cash flow statement empty | Fall back to `search-transactions` analysis only |
| No recent transactions | "No transactions found in this period" |
| Date range too wide | Warn user and suggest narrowing to last quarter |

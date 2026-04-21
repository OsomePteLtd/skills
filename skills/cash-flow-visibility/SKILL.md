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
- "Why am I always broke?"
- "What am I spending on?"
- "Show me my expenses"
- "What are my biggest costs?"
- "Burn rate"
- "Monthly spend breakdown"
- "Am I spending too much?"
- "Where did the money go?"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select company |
| 2 | `get-company` | Get fiscal year for date range |
| 3 | `get-cash-flow-statement` | Operating, investing, financing breakdown |
| 4 | `search-transactions` | Recent transactions with details |
| 5 | `get-bank-accounts` | Current bank balances |

## Data Available

**From `get-cash-flow-statement`:**
- Report with cash flow categories
- Structure: `{ report: { headings: [...], lines: [...] } }`

**From `search-transactions`:**
- Transaction groups by account
- Each line: date, description, amount
- Limited to newest N transactions (default 50)

**From `get-bank-accounts`:**
- Account name, bank, currency
- Current balance per account

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` to get fiscal year
3. Determine date range:
   - If user specifies: use their range
   - Default: last 3 months
4. Call `get-cash-flow-statement` with date range (both dates required)
5. Call `search-transactions` with `limit=50` for recent activity
6. Call `get-bank-accounts` for current positions
7. Analyze and present findings

## Report Structure

**Cash Flow Statement:**
```json
{
  "report": {
    "headings": ["Category", "Amount"],
    "lines": [
      ["Operating Activities", -45000],
      ["Investing Activities", -10000],
      ["Financing Activities", 0]
    ]
  }
}
```

**Transactions:**
```json
{
  "report": {
    "groups": [
      {
        "name": "Marketing Expenses (6200)",
        "lines": [
          ["2024-09-15", "Google Ads", "-8500.00"],
          ["2024-09-01", "Facebook Ads", "-6200.00"]
        ]
      }
    ]
  }
}
```

## Output Format

### Cash Flow Summary

**Operating Activities:** [amount]
**Investing Activities:** [amount]
**Financing Activities:** [amount]
**Net Cash Change:** [total]

### Recent Transactions

| Date | Account | Description | Amount |
|------|---------|-------------|--------|
| ... | ... | ... | ... |

### Current Cash Position

| Bank Account | Balance |
|--------------|---------|
| [name] | [amount] |
| **Total** | [sum] |

## Example

**User:** "Where is my money going this quarter?"

**Response:**
> **Cash Flow - Q3 2024**
>
> **From Cash Flow Statement:**
> - Operating Activities: -SGD 45,000
> - Investing Activities: -SGD 10,000
> - Financing Activities: SGD 0
> - **Net Change:** -SGD 55,000
>
> **Recent Transactions (top 10):**
> | Date | Account | Description | Amount |
> |------|---------|-------------|--------|
> | Sep 15 | Marketing | Google Ads | -8,500 |
> | Sep 10 | Rent | Office Lease | -8,000 |
> | Sep 5 | Revenue | Client Payment | +25,000 |
> | Sep 1 | Payroll | Salaries | -42,000 |
>
> **Cash Position:**
> | Account | Balance |
> |---------|---------|
> | DBS Current | SGD 180,000 |
> | OCBC Savings | SGD 65,000 |
> | **Total** | **SGD 245,000** |
>
> **Burn Rate:** ~SGD 18K/month (based on net change)

---

**User:** "What are my biggest expenses?"

**Response:**
> **Recent Expense Transactions (last 50):**
>
> **By Account:**
> - Payroll (6100): SGD 84,000
> - Marketing (6200): SGD 26,700
> - Rent (6300): SGD 24,000
> - Software (6400): SGD 15,000
>
> **Largest Individual Transactions:**
> 1. Sep 1: Payroll - SGD 42,000
> 2. Aug 1: Payroll - SGD 42,000
> 3. Sep 10: Office Rent - SGD 8,000
> 4. Sep 15: Google Ads - SGD 8,500

## Error Handling

| Scenario | Response |
|----------|----------|
| Cash flow statement empty | Use `search-transactions` analysis only |
| No recent transactions | "No transactions found in this period" |
| Date range missing | Prompt for date range (required for cash flow statement) |
| Report parsing error | Show raw report and explain limitation |

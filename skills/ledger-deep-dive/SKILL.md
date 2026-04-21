---
name: ledger-deep-dive
description: Search transactions by account and date range. Use when the user asks about specific expenses, transaction history, or account activity. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Ledger Deep Dive

Search and analyze transactions by account.

## When to Use

Invoke this skill when the user asks:
- "What transactions hit [account name]?"
- "Show me all marketing expenses"
- "What did I spend on [category]?"
- "Transaction history for [period]"
- "What's in my clearing accounts?"
- "How much did I spend on software/rent/marketing?"
- "Recent transactions"
- "Search for [keyword] in transactions"

## Limitations

The MCP server provides:
- Transaction search with date filtering (grouped by account)
- Chart of accounts (code, name, type, class)
- Document list (IDs and names only)
- Journal entry list (ID, description, amount only)

**Not available via MCP:**
- Detailed invoice line items, quantities, rates
- Journal entry DR/CR account breakdown
- Document amounts, dates, or status
- Contact/vendor details on documents

For detailed document information, users should access the OSOME dashboard directly.

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select company |
| 2 | `get-chart-of-accounts` | Find account codes by name |
| 3 | `search-transactions` | Transaction search with date filter |

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-chart-of-accounts` with jurisdiction code (SG/HK/UK) to find relevant account codes
3. Call `search-transactions` with:
   - `id`: company ID
   - `dateFrom`: start date (YYYY-MM-DD)
   - `dateTo`: end date (YYYY-MM-DD)
   - `limit`: max transactions (default 50, newest first)
4. Filter results by account name/code if user specified
5. Present transaction summary

## Chart of Accounts Response

```json
{
  "accounts": [
    { "code": "200", "name": "Accounts Receivable", "type": "current_asset", "accountClass": "asset" },
    { "code": "400", "name": "Office Supplies", "type": "expense", "accountClass": "expense" }
  ]
}
```

## Transaction Response Structure

Transactions are returned grouped by account:

```json
{
  "report": {
    "headings": ["Date", "Description", "Amount", ...],
    "groups": [
      {
        "name": "Marketing Expenses (6200)",
        "lines": [
          ["2024-09-15", "Google Ads", "-8500.00", ...],
          ["2024-09-01", "Facebook Ads", "-6200.00", ...]
        ]
      }
    ]
  }
}
```

## Output Format

### Transaction Query Response

**Account:** [Account Name] ([Account Code])
**Period:** [date range]

**Transactions:**
| Date | Description | Amount |
|------|-------------|--------|
| ... | ... | ... |

**Summary:**
- Total transactions: [count]
- Net amount: [sum]

## Examples

**User:** "What did I spend on marketing this quarter?"

**Response:**
> **Marketing Expenses (6200) - Q3 2024**
>
> | Date | Description | Amount |
> |------|-------------|--------|
> | Sep 15 | Google Ads | -SGD 8,500 |
> | Sep 1 | Facebook Ads | -SGD 6,200 |
> | Aug 20 | Agency Fee | -SGD 12,000 |
>
> **Total:** SGD 26,700 across 3 transactions

---

**User:** "Show me recent transactions"

**Response:**
> **Recent Transactions (Last 50)**
>
> | Date | Account | Description | Amount |
> |------|---------|-------------|--------|
> | Sep 20 | Bank | Transfer In | +SGD 50,000 |
> | Sep 18 | Rent | Office Lease | -SGD 8,000 |
> | Sep 15 | Marketing | Google Ads | -SGD 8,500 |
>
> Showing 50 most recent transactions.

---

**User:** "What's in my clearing accounts?"

**Response:**
> Let me search for clearing account transactions...
>
> **Clearing - Bank Transfer (1150)**
> | Date | Description | Amount |
> |------|-------------|--------|
> | Sep 10 | Pending transfer | SGD 5,400 |
>
> **Suspense Account (1160)**
> | Date | Description | Amount |
> |------|-------------|--------|
> | Aug 15 | Unidentified deposit | SGD 500 |
>
> Total uncleared: SGD 5,900

## Error Handling

| Scenario | Response |
|----------|----------|
| Account not found | List similar accounts from chart of accounts |
| No transactions | "No transactions found in this period" |
| Date range too wide | Suggest narrowing to last quarter |

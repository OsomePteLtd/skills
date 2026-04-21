---
name: ledger-deep-dive
description: Query specific transactions and accounts. Use when the user asks about specific expenses, account balances, journal entries, or transaction details. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Ledger Deep Dive

Answer specific questions about transactions and accounts.

## When to Use

Invoke this skill when the user asks:
- "What transactions hit [account name]?"
- "Show me all marketing expenses"
- "Journal entries for [period]"
- "What's in my clearing accounts?"
- "Explain this account balance"
- "Transaction history for [account]"
- "What did I spend on [category]?"
- "Show me the details of [document/invoice]"
- "How much did I spend on software/rent/marketing?"
- "Find a specific payment"
- "Details of invoice #[number]"
- "What's in my suspense account?"
- "Breakdown of [account]"
- "List all [vendor] payments"
- "Search for [keyword] in transactions"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select company |
| 2 | `get-chart-of-accounts` | Full account list for branch |
| 3 | `search-transactions` | Transaction search with filters |
| 4 | `get-journal-entries` | Double-entry journal records |
| 5 | `get-document` | Individual document details |

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` to determine branch (SG/HK/UK)
3. Call `get-chart-of-accounts` with branch
4. If user asks about specific account:
   - Match account name to chart of accounts
   - Call `search-transactions` filtered by account
5. If user asks about specific document:
   - Call `get-document` with document ID
6. If user asks about journal entries:
   - Call `get-journal-entries` with date range
7. Present detailed breakdown

## Jurisdiction-Specific Accounts

The chart of accounts varies by jurisdiction:

| Branch | Notable Accounts |
|--------|-----------------|
| SG | GST Input Tax, GST Output Tax, CPF Payable |
| UK | VAT Input, VAT Output, PAYE Payable, NI Payable |
| HK | MPF Payable (no GST/VAT) |

## Output Format

### Account Query Response

**Account:** [Account Name] ([Account Code])
**Type:** [Asset/Liability/Equity/Revenue/Expense]
**Current Balance:** [amount]

**Transaction History:**
| Date | Description | Debit | Credit | Running Balance |
|------|-------------|-------|--------|-----------------|
| ... | ... | ... | ... | ... |

**Summary:**
- Total debits: [amount]
- Total credits: [amount]
- Net movement: [amount]

---

### Document Query Response

**Document:** [Invoice/Bill/Receipt] #[number]
**Date:** [date]
**Contact:** [customer/vendor name]
**Status:** [draft/approved/paid/voided]

**Line Items:**
| Description | Account | Amount | Tax |
|-------------|---------|--------|-----|
| ... | ... | ... | ... |

**Total:** [amount]
**Tax:** [amount]
**Grand Total:** [amount]

**Journal Entry:**
| Account | Debit | Credit |
|---------|-------|--------|
| ... | ... | ... |

---

### Journal Entries Query Response

**Period:** [date range]
**Total Entries:** [count]

| Date | Reference | Description | Account | Debit | Credit |
|------|-----------|-------------|---------|-------|--------|
| ... | ... | ... | ... | ... | ... |

## Examples

**User:** "What did I spend on marketing this quarter?"

**Response:**
> **Marketing Expenses (Account 6200) - Q3 2024**
>
> **Total:** SGD 38,500
>
> | Date | Description | Amount |
> |------|-------------|--------|
> | Sep 15 | Google Ads | SGD 8,500 |
> | Sep 1 | Facebook Ads | SGD 6,200 |
> | Aug 20 | Marketing Agency | SGD 12,000 |
> | Aug 5 | LinkedIn Premium | SGD 1,800 |
> | Jul 28 | Trade Show Booth | SGD 10,000 |
>
> Marketing is 12% of total expenses this quarter.

---

**User:** "Show me invoice #INV-2024-089"

**Response:**
> **Invoice #INV-2024-089**
> - Customer: TechCorp Pte Ltd
> - Date: Sep 10, 2024
> - Status: Approved (unpaid)
>
> | Item | Qty | Rate | Amount |
> |------|-----|------|--------|
> | Consulting Services | 40 hrs | SGD 150 | SGD 6,000 |
> | Software License | 1 | SGD 2,500 | SGD 2,500 |
>
> Subtotal: SGD 8,500
> GST (9%): SGD 765
> **Total: SGD 9,265**
>
> **Journal Entry:**
> - DR Accounts Receivable: SGD 9,265
> - CR Revenue: SGD 8,500
> - CR GST Output: SGD 765

---

**User:** "What's in my clearing accounts?"

**Response:**
> **Clearing Accounts Summary**
>
> | Account | Balance | Needs Attention |
> |---------|---------|-----------------|
> | Suspense Account | SGD 1,250 | 3 uncleared items |
> | Clearing - PayNow | SGD 0 | Cleared |
> | Clearing - Bank Transfer | SGD 5,400 | 2 pending items |
>
> **Suspense Account Details (SGD 1,250):**
> 1. Aug 15: Unknown deposit SGD 500 - needs identification
> 2. Aug 22: Unmatched payment SGD 450 - check with vendor
> 3. Sep 1: Partial payment SGD 300 - awaiting reconciliation

## Error Handling

| Scenario | Response |
|----------|----------|
| Account not found | Suggest similar account names from chart of accounts |
| No transactions | "No transactions found for this account in the specified period" |
| Document not found | "Document not found. Check the document ID or search by date/vendor" |

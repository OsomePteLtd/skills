---
name: receivables-payables-tracker
description: Track who owes you money and who you owe. Use when the user asks about outstanding invoices, bills to pay, accounts receivable, accounts payable, or aging reports. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Receivables & Payables Tracker

Track who owes you money and who you owe.

## When to Use

Invoke this skill when the user asks:
- "Who owes me money?"
- "Who hasn't paid me?"
- "Chase up invoices"
- "Outstanding invoices"
- "What bills do I need to pay?"
- "What do I owe?"
- "Overdue payments"
- "Collection status"
- "Aged receivables / payables"
- "Client hasn't paid"
- "Unpaid invoices"
- "Bills due soon"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select company |
| 2 | `get-company` | Company context |
| 3 | `get-aged-receivables` | Customer invoices by age bucket |
| 4 | `get-aged-payables` | Vendor bills by age bucket |
| 5 | `list-documents` | Supporting invoice/bill details |

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` for context
3. Call `get-aged-receivables` (as of today or specified date)
4. Call `get-aged-payables` (as of today or specified date)
5. Optionally call `list-documents` for document-level detail
6. Analyze aging and present prioritized view

## Output Format

### Accounts Receivable Summary

**Total Outstanding:** [amount]

| Age Bucket | Amount | % of Total | Count |
|------------|--------|------------|-------|
| Current (0-30 days) | [amount] | [%] | [n] |
| 31-60 days | [amount] | [%] | [n] |
| 61-90 days | [amount] | [%] | [n] |
| 90+ days (Overdue) | [amount] | [%] | [n] |

**Top 5 Customers by Outstanding:**
1. [Customer Name]: [amount] - [days] days avg
2. ...

**Collection Priority:**
- Urgent (90+ days): [amount] from [n] customers
- Follow up (60-90 days): [amount] from [n] customers

---

### Accounts Payable Summary

**Total Outstanding:** [amount]

| Age Bucket | Amount | % of Total | Count |
|------------|--------|------------|-------|
| Current (0-30 days) | [amount] | [%] | [n] |
| 31-60 days | [amount] | [%] | [n] |
| 61-90 days | [amount] | [%] | [n] |
| 90+ days (Overdue) | [amount] | [%] | [n] |

**Top 5 Vendors by Outstanding:**
1. [Vendor Name]: [amount] - due [date]
2. ...

**Payment Priority:**
- Overdue (90+ days): [amount] to [n] vendors
- Due soon (0-30 days): [amount] to [n] vendors

---

### Net Position

**Net Working Capital Impact:** [AR - AP]
- If positive: You are owed more than you owe
- If negative: You owe more than you are owed

## Example

**User:** "Who owes me money?"

**Response:**
> **Accounts Receivable: SGD 156,000 outstanding**
>
> | Age | Amount | Invoices |
> |-----|--------|----------|
> | Current | SGD 85,000 | 12 |
> | 31-60 days | SGD 42,000 | 5 |
> | 61-90 days | SGD 18,000 | 3 |
> | 90+ days | SGD 11,000 | 2 |
>
> **Top customers to follow up:**
> 1. TechCorp Pte Ltd: SGD 45,000 (52 days avg)
> 2. StartupXYZ: SGD 28,000 (38 days avg)
> 3. BigClient Inc: SGD 22,000 (65 days avg)
>
> **Action needed:** SGD 29,000 is overdue (60+ days). Consider sending reminders to 5 customers.

---

**User:** "What bills do I need to pay?"

**Response:**
> **Accounts Payable: SGD 78,000 outstanding**
>
> | Age | Amount | Bills |
> |-----|--------|-------|
> | Current | SGD 52,000 | 8 |
> | 31-60 days | SGD 18,000 | 3 |
> | 61-90 days | SGD 5,000 | 1 |
> | 90+ days | SGD 3,000 | 1 |
>
> **Priority payments:**
> 1. AWS Singapore: SGD 12,500 (due in 5 days)
> 2. Office Landlord: SGD 8,000 (due in 10 days)
> 3. Supplier ABC: SGD 3,000 (92 days overdue!)
>
> **Cash needed:** SGD 52,000 due within 30 days. You have SGD 245,000 in bank accounts.

## Error Handling

| Scenario | Response |
|----------|----------|
| No receivables | "Great news - no outstanding customer invoices!" |
| No payables | "No outstanding bills to pay" |
| Very old items (180+ days) | Flag as potential bad debt / write-off candidates |

---
name: ledger-deep-dive
description: Search transactions by account and date range, and read or add transaction comments. Use when the user asks about specific expenses, transaction history, account activity, or the comment thread on a transaction. Requires OSOME MCP server at https://mcp.osome.com/mcp
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
- "What comments are on this transaction?"
- "Add a comment asking for the receipt"
- "Reply to the accountant on transaction [ID]"

## Limitations

The MCP server provides:
- Transaction search with date filtering (grouped by account)
- Transaction comment conversation reads
- Text comments on transaction conversations
- Chart of accounts (code, name, type, class)
- Document list (IDs and names only)
- Journal entry list (ID, description, amount only)
- Document upload for supporting receipts and evidence via `prepare-document-upload` and `complete-document-upload`

**Not available via MCP:**
- Detailed invoice line items, quantities, rates
- Journal entry DR/CR account breakdown
- Document amounts, dates, or status
- Contact/vendor details on documents

For detailed document information, users should access the OSOME dashboard directly. If the user needs to add a receipt or supporting file, use `prepare-document-upload`, upload bytes directly to the returned presigned target, then call `complete-document-upload`. If the user needs to ask or answer a transaction question, use the transaction comment tools instead of asking them to leave the chat.

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
| Optional | `get-transaction-conversation` | Read the comments attached to a transaction |
| Optional | `add-transaction-comment` | Add a text comment to a transaction conversation |
| Optional | `prepare-document-upload` | Prepare upload of receipts or supporting files for transaction follow-up |
| Optional | `complete-document-upload` | Complete document creation after bytes are uploaded to the presigned target |

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
6. If the user asks about comments on a transaction:
   - Identify the transaction and numeric transaction document ID
   - Call `get-transaction-conversation` with `companyId`, `transactionId`, and optional `limit`
   - Summarize the visible messages with dates, sender names, and unresolved context
   - If no conversation exists, say that there are no comments yet
7. If the user wants to add a transaction comment:
   - Confirm the exact text before posting when it changes the audit trail or asks someone to act
   - Call `add-transaction-comment` with `companyId`, `transactionId`, and `text`
   - This requires OAuth scope `company:documents:write`
   - Report the posted message and any recent conversation context returned by the tool
8. If the user wants to add a receipt or supporting document for follow-up, use the document upload flow:
   - Prepare with `companyId`, `filename`, `contentType`, `sizeBytes`, and lowercase MD5 `checksum`
   - Call `prepare-document-upload` (requires OAuth scope `company:documents:write`)
   - If `duplicate: true`, explain that OSOME already has the document
   - Upload the file bytes outside MCP to `upload.method`/`upload.url` using returned `upload.fields`; include returned form fields exactly and append the file last
   - Do not send file bytes, `contentBase64`, or arbitrary upload headers to MCP
   - Call `complete-document-upload` with the same metadata plus `uploadToken`

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

## Transaction Conversation Response Structure

```json
{
  "conversation": {
    "id": "6150830",
    "companyId": "2461469",
    "entryId": "48884794",
    "entryType": "AcDocument",
    "lastMessageAt": "2026-07-30T00:00:00.000Z"
  },
  "messages": [
    {
      "id": "14636917",
      "text": "Please attach the receipt for this payment.",
      "createdAt": "2026-07-30T00:00:00.000Z",
      "sender": { "nickname": "Customer", "isAgent": false }
    }
  ]
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

### Transaction Comments Response

**Transaction:** [description or ID]

**Comments:**
| Date | Sender | Comment |
|------|--------|---------|
| ... | ... | ... |

**Next step:** [what still needs an answer, if anything]

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

---

**User:** "What comments are on transaction 48884794?"

**Response:**
> I'll check the transaction conversation.
>
> **Transaction 48884794 comments**
>
> | Date | Sender | Comment |
> |------|--------|---------|
> | Jul 30 | Accountant | Please attach the receipt for this payment. |
>
> **Next step:** Upload the receipt or reply with where this payment came from.

---

**User:** "Add a comment saying this was the July software subscription."

**Response:**
> I'll confirm the transaction first, then post this comment:
>
> "This was the July software subscription."
>
> After you confirm, I will call `add-transaction-comment`.

## Error Handling

| Scenario | Response |
|----------|----------|
| Account not found | List similar accounts from chart of accounts |
| No transactions | "No transactions found in this period" |
| No transaction comments | "No comments found for this transaction" |
| Missing document write scope | Tell the user the OSOME MCP connection needs `company:documents:write` to add transaction comments |
| Date range too wide | Suggest narrowing to last quarter |

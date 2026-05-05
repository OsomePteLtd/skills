---
name: receivables-payables-tracker
description: Track who owes you money and who you owe. Use when the user asks about outstanding invoices, bills to pay, accounts receivable, accounts payable, or aging reports. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Receivables & Payables Tracker

Track who owes you money and who you owe.

## When to Use

Invoke this skill when the user asks:
- "Who owes me money?"
- "What bills do I need to pay?"
- "Outstanding invoices"
- "Aged receivables"
- "Aged payables"
- "What's overdue?"
- "Collection status"
- "Accounts receivable / accounts payable"
- "Who hasn't paid me?"
- "Chase up invoices"
- "What do I owe?"
- "Overdue payments"
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
| 5 | `get-bank-accounts` | Current cash for context |
| Optional | `upload-document` | Upload invoice PDFs, bills, remittance advice, or supporting documents |

## Data Available

**From `get-aged-receivables` / `get-aged-payables`:**
- Report format with headings and line items
- Customer/vendor names (in report lines)
- Amounts by aging bucket
- Structure: `{ report: { headings: [...], lines: [...] } }`

**From `get-bank-accounts`:**
- Account balances to show available cash

**From `upload-document` (optional):**
- Upload supporting documents for receivables or payables follow-up
- Supported file types: PDF, JPG/JPEG, PNG, CSV, XLS, XLSX
- Maximum file size: 50 MB

## Limitations

- **Document details limited** - `list-documents` only returns `{ id, name }`, not amounts or due dates
- **Invoice line items not available** - For detailed invoice info, users should access OSOME dashboard
- **Report structure varies** - Actual field names depend on the accounting system's report format
- **Document upload is two-step** - Call `upload-document` with `step: "prepare"`, upload the file to the returned presigned target, then call `step: "complete"` within 15 minutes

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` for context
3. Call `get-aged-receivables` (as of today or specified date)
4. Call `get-aged-payables` (as of today or specified date)
5. Call `get-bank-accounts` for cash context
6. Parse report data to extract aging buckets
7. Present prioritized view
8. If the user needs to upload an invoice, bill, remittance advice, or supporting file, call `upload-document`:
   - Prepare with `companyId`, `filename`, `contentType`, `sizeBytes`, and lowercase MD5 `checksum`
   - If `duplicate: true`, explain that OSOME already has the document
   - Upload the file to the returned presigned target
   - Complete with the same metadata plus returned `preparedFile` and `uploadToken` within 15 minutes

## Report Structure

Aged reports return data like:
```json
{
  "report": {
    "headings": ["Contact", "Current", "1-30 Days", "31-60 Days", "61-90 Days", "90+ Days", "Total"],
    "lines": [
      ["TechCorp Pte Ltd", 5000, 0, 2000, 0, 0, 7000],
      ["StartupXYZ", 0, 3000, 0, 0, 0, 3000]
    ]
  }
}
```

Parse according to the headings returned.

## Output Format

### Accounts Receivable Summary

**Total Outstanding:** [amount]

| Age Bucket | Amount | % of Total |
|------------|--------|------------|
| Current (0-30 days) | [amount] | [%] |
| 31-60 days | [amount] | [%] |
| 61-90 days | [amount] | [%] |
| 90+ days (Overdue) | [amount] | [%] |

**Customers with Outstanding (from report):**
- [Customer Name]: [amount]
- ...

---

### Accounts Payable Summary

**Total Outstanding:** [amount]

| Age Bucket | Amount | % of Total |
|------------|--------|------------|
| Current (0-30 days) | [amount] | [%] |
| 31-60 days | [amount] | [%] |
| 61-90 days | [amount] | [%] |
| 90+ days (Overdue) | [amount] | [%] |

**Vendors with Outstanding (from report):**
- [Vendor Name]: [amount]
- ...

---

### Cash Context

**Current Cash Position:** [from bank accounts]
**AR - AP Net:** [receivables - payables]

## Example

**User:** "Who owes me money?"

**Response:**
> **Accounts Receivable Report**
>
> | Customer | Current | 31-60 | 61-90 | 90+ | Total |
> |----------|---------|-------|-------|-----|-------|
> | TechCorp | 45,000 | 0 | 0 | 0 | 45,000 |
> | StartupXYZ | 20,000 | 8,000 | 0 | 0 | 28,000 |
> | BigClient | 0 | 0 | 22,000 | 0 | 22,000 |
>
> **Summary:**
> - Total AR: SGD 95,000
> - Current: SGD 65,000 (68%)
> - Overdue (60+): SGD 22,000 (23%)
>
> **Action:** Follow up with BigClient on SGD 22,000 (61-90 days).

---

**User:** "What bills do I need to pay?"

**Response:**
> **Accounts Payable Report**
>
> | Vendor | Current | 31-60 | 61-90 | 90+ | Total |
> |--------|---------|-------|-------|-----|-------|
> | AWS | 12,500 | 0 | 0 | 0 | 12,500 |
> | Landlord | 8,000 | 0 | 0 | 0 | 8,000 |
> | Supplier | 0 | 0 | 0 | 3,000 | 3,000 |
>
> **Summary:**
> - Total AP: SGD 23,500
> - Due soon (Current): SGD 20,500
> - Overdue (90+): SGD 3,000
>
> **Cash available:** SGD 245,000 (from bank accounts)
> **Coverage:** 10x current payables

## Error Handling

| Scenario | Response |
|----------|----------|
| No receivables | "No outstanding customer invoices found." |
| No payables | "No outstanding bills to pay." |
| Report parsing error | Show raw report data and explain structure |
| Very old items (180+ days) | Flag as potential bad debt |

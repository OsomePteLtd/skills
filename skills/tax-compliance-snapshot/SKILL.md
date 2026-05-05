---
name: tax-compliance-snapshot
description: Check tax obligations and prepare for tax filing. Use when the user asks about tax due, GST, VAT, tax preparation, or compliance deadlines. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Tax & Compliance Snapshot

Check your tax obligations and prepare for filing.

## When to Use

Invoke this skill when the user asks:
- "How much GST/VAT do I owe?"
- "What's my tax situation?"
- "Am I ready for tax filing?"
- "GST summary"
- "VAT return"
- "When is my GST/VAT due?"
- "Help me prepare for my accountant"
- "Tax obligations"
- "What do I need for year-end?"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select company |
| 2 | `get-company` | Get fiscal year end |
| 3 | `get-trial-balance` | Account balances including tax accounts |
| 4 | `get-chart-of-accounts` | Identify tax-related account codes |
| 5 | `get-profit-and-loss` | Revenue and expenses for tax computation |
| Optional | `upload-document` | Upload supporting tax documents, receipts, invoices, or filing evidence |

## Data Available

**From `get-company`:**
- Company ID, name
- Next fiscal year end date

**From `get-chart-of-accounts` (requires jurisdiction code):**
- Account codes, names, types
- Tax accounts like GST Input/Output

**From `get-trial-balance`:**
- Report with account balances
- Tax account balances (if accounts exist)

**From `get-profit-and-loss`:**
- Revenue and expense totals
- Net profit for tax estimation

**From `upload-document` (optional):**
- Prepare and complete uploads of supporting documents for tax filing
- Supported file types: PDF, JPG/JPEG, PNG, CSV, XLS, XLSX
- Maximum file size: 50 MB

## Limitations

- **Jurisdiction not returned by API** - Ask user for their jurisdiction (SG/HK/UK/UAE) or infer from company name
- **GST registration status not available** - Check if GST accounts exist in trial balance as proxy
- **Filing deadlines not from API** - Use standard deadlines based on jurisdiction
- **Document upload is two-step** - Call `upload-document` with `step: "prepare"`, upload the file to the returned presigned target, then call `step: "complete"` within 15 minutes

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` to get fiscal year end
3. Ask user for jurisdiction if not obvious from company name
4. Call `get-chart-of-accounts` with jurisdiction code (SG/HK/UK)
5. Call `get-trial-balance` with date range
6. Look for GST/VAT accounts in the report
7. Call `get-profit-and-loss` for revenue/expense totals
8. Calculate estimated tax position
9. If the user needs to provide supporting tax documents, call `upload-document`:
   - Prepare with `companyId`, `filename`, `contentType`, `sizeBytes`, and lowercase MD5 `checksum`
   - If `duplicate: true`, tell the user the document already exists and stop the upload flow
   - Upload the file to the returned presigned target
   - Complete with the same metadata plus returned `preparedFile` and `uploadToken`

## Jurisdiction Tax Reference

| Jurisdiction | Tax Type | Rate | Quarterly Deadlines |
|--------------|----------|------|---------------------|
| Singapore | GST | 9% | Apr 30, Jul 31, Oct 31, Jan 31 |
| UK | VAT | 20% | Varies by registration |
| Hong Kong | Profits Tax | 8.25%/16.5% | Annual only |
| UAE | VAT | 5% | 28 days after quarter end |

## Output Format

### GST/VAT Position

**Based on Trial Balance (if GST accounts found):**

| Account | Balance |
|---------|---------|
| GST Output Tax | [amount] |
| GST Input Tax | [amount] |
| Net Payable | [output - input] |

### Tax Filing Preparation

**From P&L Report:**
| Item | Amount |
|------|--------|
| Total Revenue | [amount] |
| Total Expenses | [amount] |
| Net Profit | [amount] |
| Estimated Tax | [amount at rate%] |

**Fiscal Year:** [start] to [end]

## Examples

**User:** "How much GST do I owe?"

**Response:**
> I'll check your GST position. What jurisdiction is your company registered in? (SG/UK/UAE)
>
> *[After user confirms Singapore]*
>
> **GST Position - ACME Pte Ltd**
>
> From trial balance:
> | Account | Balance |
> |---------|---------|
> | GST Output Tax (collected) | SGD 28,500 |
> | GST Input Tax (paid) | SGD 12,300 |
> | **Net GST Payable** | **SGD 16,200** |
>
> **Standard Q3 deadline:** October 31
>
> Note: Verify these figures match your GST F5 form before filing.

---

**User:** "Help me prepare for tax filing"

**Response:**
> **Tax Filing Preparation - ACME Pte Ltd**
>
> **Fiscal Year End:** December 31, 2024
>
> **From Profit & Loss:**
> | | Amount |
> |---|--------|
> | Total Revenue | SGD 890,000 |
> | Total Expenses | SGD 765,000 |
> | Net Profit | SGD 125,000 |
>
> **Available Reports:**
> - Balance Sheet - via `get-balance-sheet`
> - Profit & Loss - via `get-profit-and-loss`
> - Trial Balance - via `get-trial-balance`
>
> **For complete tax filing, you'll also need:**
> - Bank statements (from your bank)
> - Supporting documents such as receipts, invoices, bank statements, and tax schedules
>
> If you want to upload supporting documents now, I can use `upload-document` to prepare the upload, send the file to the returned upload target, and complete it in OSOME. Files must be PDF, JPG/JPEG, PNG, CSV, XLS, or XLSX and under 50 MB.

---

**User:** "What's my tax situation?" (Hong Kong)

**Response:**
> **Tax Position - ACME HK Limited**
>
> Hong Kong does not have GST/VAT.
>
> **From Profit & Loss:**
> | | Amount |
> |---|--------|
> | Total Revenue | HKD 2,500,000 |
> | Total Expenses | HKD 1,800,000 |
> | Net Profit | HKD 700,000 |
>
> **Estimated Profits Tax:**
> - First HKD 2M profit: 8.25%
> - Above HKD 2M: 16.5%
> - Estimate: ~HKD 57,750
>
> Note: Actual tax depends on allowable deductions and tax positions.

## Error Handling

| Scenario | Response |
|----------|----------|
| No GST accounts in trial balance | "No GST accounts found. Company may not be GST-registered." |
| HK company asking about GST | "Hong Kong does not have GST/VAT. Showing profits tax info." |
| Empty P&L | "No financial data for this period. Has bookkeeping been completed?" |
| Jurisdiction unknown | Ask user to confirm their company's jurisdiction |

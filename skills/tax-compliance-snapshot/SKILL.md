---
name: tax-compliance-snapshot
description: Check tax obligations, GST/VAT status, and prepare for tax filing. Use when the user asks about tax due, GST, VAT, tax preparation, or compliance deadlines. Requires OSOME MCP server at https://mcp.osome.com/mcp
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
| 2 | `get-company` | Jurisdiction, fiscal year, GST registration |
| 3 | `get-trial-balance` | GST/VAT account balances |
| 4 | `get-chart-of-accounts` | Identify tax-related accounts |
| 5 | `get-profit-and-loss` | Revenue and expenses for tax computation |
| 6 | `get-balance-sheet` | Assets and liabilities snapshot |

## Execution Flow

1. Call `list-companies` and select company
2. Call `get-company` to determine:
   - Jurisdiction (SG/HK/UK/UAE)
   - GST/VAT registration status
   - Fiscal year end date
3. Call `get-chart-of-accounts` to find tax accounts:
   - SG: GST Input Tax, GST Output Tax, GST Payable
   - UK: VAT Input, VAT Output, VAT Control
   - HK: No GST/VAT (skip this step)
4. Call `get-trial-balance` as of today
5. Extract tax account balances
6. Call `get-profit-and-loss` for the current period
7. Synthesize tax position summary

## Jurisdiction-Specific Tax

| Jurisdiction | Tax Type | Rate | Filing Frequency |
|--------------|----------|------|------------------|
| Singapore | GST | 9% | Quarterly |
| UK | VAT | 20% | Quarterly |
| Hong Kong | None | - | Annual profits tax only |
| UAE | VAT | 5% | Quarterly |

## Output Format

### GST/VAT Summary (SG/UK/UAE only)

**GST/VAT Position as of [date]:**

| Account | Balance | Notes |
|---------|---------|-------|
| Output Tax (collected) | [amount] | Tax charged to customers |
| Input Tax (paid) | [amount] | Tax paid on purchases |
| **Net Payable/(Refundable)** | [amount] | Output - Input |

**Filing Status:**
- Current quarter: [Q1/Q2/Q3/Q4 YYYY]
- Filing deadline: [date]
- Days until due: [n] days

**Quarter Breakdown:**
| Month | Sales | GST Collected | Purchases | GST Paid |
|-------|-------|---------------|-----------|----------|
| [Month 1] | [amount] | [amount] | [amount] | [amount] |
| [Month 2] | [amount] | [amount] | [amount] | [amount] |
| [Month 3] | [amount] | [amount] | [amount] | [amount] |

---

### Tax Filing Preparation

**Documents Ready:**
- [ ] Profit & Loss Statement
- [ ] Balance Sheet
- [ ] Trial Balance
- [ ] Bank Statements
- [ ] GST/VAT Returns (if applicable)

**Key Figures for Tax:**
| Item | Amount |
|------|--------|
| Total Revenue | [amount] |
| Total Expenses | [amount] |
| Net Profit Before Tax | [amount] |
| Estimated Tax (at [rate]%) | [amount] |

**Fiscal Year:**
- Current FY: [start] to [end]
- Days until year end: [n] days

---

### Compliance Calendar

| Obligation | Due Date | Status |
|------------|----------|--------|
| GST/VAT Filing Q[n] | [date] | [Upcoming/Overdue] |
| Annual Return | [date] | [Upcoming/Overdue] |
| Corporate Tax (ECI) | [date] | [Upcoming/Overdue] |

## Examples

**User:** "How much GST do I owe?"

**Response:**
> **GST Position - ACME Pte Ltd**
>
> As of October 15, 2024:
>
> | | Amount |
> |---|--------|
> | GST Collected (Output) | SGD 28,500 |
> | GST Paid (Input) | SGD 12,300 |
> | **Net GST Payable** | **SGD 16,200** |
>
> **Q3 2024 Filing:**
> - Filing deadline: October 31, 2024
> - 16 days remaining
>
> You need to pay SGD 16,200 to IRAS by month end.

---

**User:** "Help me prepare for tax filing"

**Response:**
> **Tax Filing Preparation - ACME Pte Ltd**
>
> **Fiscal Year:** Jan 1 - Dec 31, 2024
> **Days until year end:** 77 days
>
> **Year-to-Date Summary:**
> | | Amount |
> |---|--------|
> | Total Revenue | SGD 890,000 |
> | Total Expenses | SGD 765,000 |
> | Net Profit | SGD 125,000 |
> | Estimated Tax (17%) | SGD 21,250 |
>
> **Documents Available:**
> - Profit & Loss - Ready
> - Balance Sheet - Ready
> - Trial Balance - Ready
> - GST F5 Returns (Q1-Q3) - Filed
>
> **Next Steps:**
> 1. Review P&L for any missing expenses
> 2. Ensure all invoices are recorded
> 3. Reconcile bank accounts
> 4. File Q4 GST by Jan 31, 2025
> 5. Submit ECI within 3 months of year end

---

**User:** "What's my tax situation?" (Hong Kong company)

**Response:**
> **Tax Position - ACME HK Limited**
>
> Hong Kong does not have GST/VAT.
>
> **Profits Tax Summary:**
> | | Amount |
> |---|--------|
> | Total Revenue | HKD 2,500,000 |
> | Total Expenses | HKD 1,800,000 |
> | Net Profit | HKD 700,000 |
>
> **Estimated Profits Tax:**
> - First HKD 2M: 8.25% = HKD 57,750
> - Above HKD 2M: 16.5% = HKD 0
> - **Total: HKD 57,750**
>
> **Filing Deadline:**
> - Profits Tax Return due: [date based on FY]
> - Singed accounts required: Yes

## Error Handling

| Scenario | Response |
|----------|----------|
| No GST registration | "This company is not GST-registered. Only profits tax applies." |
| HK company asking about GST | "Hong Kong does not have GST/VAT. Showing profits tax info instead." |
| Missing tax accounts | "Tax accounts not found in chart of accounts. Please check GST registration." |
| No P&L data | "No financial data for this period. Has bookkeeping been completed?" |

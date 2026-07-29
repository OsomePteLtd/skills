---
name: invoice-revenue-insights
description: Analyze invoice revenue, pending invoices, aged receivables, overdue invoices, and invoice-backed collection priorities in OSOME. Use when the user asks about invoice revenue, overdue invoices, invoices awaiting payment, future invoice totals, receivables aging, customer collection priorities, or revenue summaries from invoicing. Requires OSOME MCP server at https://mcp.osome.com/mcp with openid and invoice:read scopes.
---

# Invoice Revenue Insights

## Overview

Analyze invoice revenue and collections through OSOME MCP read tools. Use this skill for invoice dashboards, overdue invoice triage, receivables aging, pending invoice totals, and revenue summaries.

## MCP Server

This skill requires the OSOME MCP server:

```text
https://mcp.osome.com/mcp
```

## Scope And Boundaries

- Use `openid` for company and bank context and `invoice:read` for invoice reports, invoices, and customers.
- This is a read-only skill. Do not create, submit, duplicate, or credit invoices here.
- For invoice creation, PDF generation, or credit notes, switch to `invoice-creator`.
- Default to current quarter for general revenue questions unless the user specifies dates or year-to-date.
- Keep exact date bounds in `YYYY-MM-DD` when the user supplies them.

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select the company |
| 2 | `get-company` | Confirm company context |
| 3 | `get-revenue-summary` | Founder-friendly revenue summary from P&L v2 |
| Optional | `get-profit-and-loss-v2` | Detailed revenue and expense report |
| Optional | `get-aged-receivables-v2` | Full receivables aging report |
| Optional | `get-overdue-invoices` | Past-due receivables only, derived from aged receivables v2 |
| Optional | `get-invoice-pending-summary` | Awaiting, future, overdue, and total invoice pending totals |
| Optional | `list-invoices` | Find invoices by status, search, page, and per-page filters |
| Optional | `preview-invoice` | Inspect a specific invoice without mutation |
| Optional | `get-invoice-document-links` | Get existing invoice document preview/original URLs |
| Optional | `get-bank-accounts` | Add cash context when discussing collection runway or liquidity |

## Revenue Workflow

1. Call `list-companies` and select the relevant company.
2. Decide the date range:
   - Use `period: "currentQuarter"` for broad current revenue questions.
   - Use `period: "yearToDate"` for YTD questions.
   - Use `dateFrom` and `dateTo` when the user provides exact bounds.
3. Call `get-revenue-summary` with `companyId` and the selected period or bounds.
4. Call `get-profit-and-loss-v2` when the user asks for line-level report detail or wants revenue compared with expenses.
5. Present revenue, period, currency, trend drivers from the returned report, and any caveats from the data.

## Collections Workflow

Use this workflow when the user asks who has not paid, what is overdue, what to chase, or how much cash is pending from invoices.

1. Call `get-invoice-pending-summary` for awaiting, future, overdue, total, currency, count, and amount.
2. Call `get-overdue-invoices` for overdue-only collection priorities.
3. Call `get-aged-receivables-v2` when the user asks for full aging, including current and future buckets.
4. Call `list-invoices` with `status: ["awaiting", "overdue"]` when the user needs invoice-level records.
5. Call `preview-invoice` for specific invoice details only when the list/report output is not enough.
6. Call `get-bank-accounts` if the user asks whether collections affect near-term cash coverage.

## Data Notes

- `get-overdue-invoices` reads aged receivables v2 and returns only past-due groups. Current buckets are intentionally excluded.
- `get-aged-receivables-v2` returns report headings and groups; parse according to the returned headings rather than hard-coding column positions.
- `get-invoice-pending-summary` returns pending totals such as awaiting, future, overdue, total, currency, count, and amount.
- `list-invoices` supports `page`, `perPage`, `search`, and status filters: `draft`, `awaiting`, `overdue`, `paid`, `cancelled`.
- `preview-invoice` can include customer, dates, status, totals, line items, payment methods, document IDs, and document URLs.

## Output Format

For revenue questions, show:

- Period and date bounds used
- Revenue total and currency
- P&L context when requested
- Any unreconciled or multicurrency caveats returned by the report

For collections questions, show:

- Pending total split into awaiting, future, and overdue when available
- Overdue invoices grouped by age bucket or severity
- Customer/invoice rows with amount, due date, status, and invoice number when available
- Suggested follow-up order, starting with oldest or largest overdue balances
- Cash context only when it helps answer the user's question

## Example Responses

**User:** "What invoices are overdue?"

Response:

> **Overdue invoices**
>
> | Customer | Bucket | Amount | Notes |
> |----------|--------|--------|-------|
> | Acme Pte Ltd | 91+ days overdue | SGD 3,000 | Highest priority |
>
> **Total overdue:** SGD 3,000
> **Next action:** Chase oldest overdue balances first.

**User:** "How much revenue did we make this quarter?"

Response:

> **Revenue summary**
>
> Period: [dateFrom] to [dateTo]
>
> Revenue: SGD [amount]
>
> I used the current quarter because no custom dates were provided.

## Error Handling

| Scenario | Response |
|----------|----------|
| Missing company | Ask the user to select a company after `list-companies` |
| Missing invoice read scope | Tell the user the MCP connection needs `invoice:read` |
| Empty overdue report | Say no overdue invoices were found for the selected period |
| Empty revenue report | Say no revenue data was returned for the selected period |
| Report shape differs | Show the returned headings/groups and explain what can be parsed safely |
| User asks to send or edit an invoice | Use `invoice-creator`; this skill is read-only |

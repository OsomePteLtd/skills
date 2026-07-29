---
name: invoice-creator
description: Create, update, submit, duplicate, preview, and generate PDF links for OSOME invoices, including invoice customers and credit notes. Use when the user asks to create an invoice, send an invoice, generate an invoice PDF, edit a draft invoice, duplicate an invoice, manage an invoice customer, issue a credit note, or correct an issued invoice. Requires OSOME MCP server at https://mcp.osome.com/mcp with openid, invoice:read, and invoice:write scopes.
---

# Invoice Creator

## Overview

Create and manage OSOME invoices through the OSOME MCP server. Use the workflow below to keep draft preparation, final submission, document links, and credit-note corrections explicit.

## MCP Server

This skill requires the OSOME MCP server:

```text
https://mcp.osome.com/mcp
```

## Scopes And Safety

- Use `openid` for company selection and `invoice:read` for invoice reads: customers, invoice preview, invoice lists, document links, and credit-note inspection.
- Use `invoice:write` for writes: customer creation, draft creation/update, submission, PDF generation for drafts, duplication, and credit notes.
- Confirm final invoice details before `submit-invoice` or `generate-invoice-pdf` on a draft. These tools finalize and send a draft invoice.
- Confirm final credit-note details before `submit-credit-note`. This applies the credit note to the invoice.
- Do not look for delete or pro-forma tools. The MCP server intentionally does not expose `delete-invoice` or `create-proforma-invoice`.
- Correct issued invoices with credit notes rather than trying to mutate or delete historical invoices.

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select the company |
| 2 | `get-company` | Confirm company context |
| 3 | `list-customers` | Find the invoice recipient customer |
| Optional | `create-customer` | Create a customer and use the returned `id` as `customerId` |
| 4 | `create-invoice-draft` | Create an empty invoice draft |
| 5 | `update-invoice-draft` | Assign customer, dates, currency, notes, payment methods, and line items |
| 6 | `preview-invoice` | Review stored invoice data without mutation |
| 7 | `submit-invoice` | Finalize and send a draft invoice |
| Optional | `generate-invoice-pdf` | Submit draft if needed, then return document links |
| Optional | `get-invoice-document-links` | Return existing document preview/original URLs |
| Optional | `list-invoices` | Find invoices by status, search, and pagination |
| Optional | `duplicate-invoice` | Copy an invoice into a new draft |
| Optional | `create-credit-note` | Create a draft credit note for an issued invoice |
| Optional | `submit-credit-note` | Submit and apply a credit note |
| Optional | `get-credit-note` | Inspect a credit note and safe invoice summary |

## Create Invoice Workflow

1. Call `list-companies` and select the relevant company.
2. Call `list-customers` with `companyId` and `search` when the user names a recipient.
3. If no matching customer exists and the user provides customer details, call `create-customer`.
4. Call `create-invoice-draft` with `companyId`.
5. Call `update-invoice-draft` with the returned invoice `id`, `companyId`, and the draft invoice data.
6. Call `preview-invoice` and summarize the recipient, dates, line items, tax treatment, currency, notes, and totals.
7. If the user asks to send the invoice, call `submit-invoice` only after the final details are clear.
8. If the user asks for the invoice PDF or document links, call `generate-invoice-pdf` for drafts or `get-invoice-document-links` for already-sent invoices.

## Draft Data

Use `update-invoice-draft` for draft fields:

- `customerId`: numeric customer ID. Recipient details are auto-populated from the customer.
- `dueDate`: due date in `YYYY-MM-DD`.
- `issueDate`: issue date in `YYYY-MM-DD`.
- `supplyStartDate` and `supplyEndDate`: optional supply period dates.
- `currency`: currency code such as `SGD`, `USD`, `GBP`, `HKD`, or `AED`.
- `includeTaxIntoPrice`: whether tax is included in line item prices.
- `invoiceNumber`: optional custom invoice number.
- `notes`: optional invoice notes.
- `paymentMethods`: array of `{ id, type }`.
- `lineItems`: array of invoice lines.

Each line item requires:

- `name`
- `price`
- `unit`
- `order`
- `quantity`

Optional line item fields include `description`, `taxCategory`, `taxRate`, `discount`, `discountUnit`, `discountAmount`, `taxAmount`, `subTotal`, and `grandTotal`. Let the service calculate totals when the user has not supplied exact totals.

## Submit And PDF Rules

- `submit-invoice` requires `customerId`, `dueDate`, `issueDate`, `currency`, and `includeTaxIntoPrice`.
- `submit-invoice` uses the stored draft recipient and line items from `preview-invoice`; make sure they were already set with `update-invoice-draft`.
- `generate-invoice-pdf` returns existing links for non-draft invoices.
- `generate-invoice-pdf` submits a draft before returning links, so include the same required submit fields when the invoice is still draft.
- `get-invoice-document-links` is read-only and returns preview/original URLs for an existing invoice.

## Credit Note Workflow

Use credit notes when the user wants to correct, cancel out, refund, or reverse an issued invoice.

1. Find the invoice with `list-invoices`, `preview-invoice`, or the user-provided invoice ID.
2. Call `create-credit-note` with `companyId` and `invoiceId`.
3. Review the returned draft credit note.
4. Call `submit-credit-note` with credit note `id`, `companyId`, `issueDate`, and optional `creditNoteNumber` and `reason` after details are clear.
5. Call `get-credit-note` when the user asks for credit-note status or document information.

## Output Format

For draft preparation, show:

- Invoice ID and status
- Customer name and customer ID
- Issue date, due date, currency, and tax-inclusive setting
- Line items with quantity, unit, price, discount, tax, and total when available
- Notes and payment methods when present
- Next action: preview, edit, send, or generate PDF

For sent invoices or generated PDFs, show:

- Invoice number and status
- Document preview/original URLs when returned
- Any remaining action needed from the user

## Error Handling

| Scenario | Response |
|----------|----------|
| Missing company | Ask the user to select a company after `list-companies` |
| Missing customer | Search `list-customers`; offer to create a customer if details are available |
| Missing line items | Do not submit; ask for the line items needed for `update-invoice-draft` |
| Draft PDF without submit fields | Ask for required submit fields before `generate-invoice-pdf` |
| User asks to delete invoice | Explain that delete is not available and use credit-note correction for issued invoices |
| Missing invoice scope | Tell the user the MCP connection needs `invoice:read` or `invoice:write` |

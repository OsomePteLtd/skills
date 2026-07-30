---
name: transaction-cashcoding
description: Review and apply OSOME transaction cashcoding recommendations through MCP. Use when the user asks to categorize, code, reconcile, classify, or apply an accounting account to a bank transaction. Requires OSOME MCP server at https://mcp.osome.com/mcp with openid for reads and company:documents:write before applying a recommendation.
---

# Transaction Cashcoding

## Overview

Use OSOME cashcoding recommendations to help a user choose the accounting account for a transaction, then apply the selected recommendation only after confirmation.

## Scopes And Safety

- Use `openid` to select the company and read recommendation data.
- Use `company:documents:write` only when applying a recommendation.
- Treat `apply-transaction-cashcoding-recommendation` as a write action: confirm the transaction, account, and description before calling it.
- Do not apply a recommendation when the user only asked what OSOME recommends.
- Do not guess the transaction ID from description alone. Ask for the OSOME transaction document ID or transaction page URL when the MCP results do not include a safe numeric transaction document ID.

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | Select the company |
| Optional | `search-transactions` | Find the likely transaction context |
| 2 | `list-transaction-cashcoding-recommendations` | Read suggested accounting accounts for a transaction |
| 3 | `apply-transaction-cashcoding-recommendation` | Apply the selected account through OSOME cashcoding |

## Workflow

1. Call `list-companies` and select the company.
2. Identify the transaction. Prefer a numeric OSOME transaction document ID from the transaction page or an MCP result that explicitly includes the transaction document ID.
3. Call `list-transaction-cashcoding-recommendations` with `companyId` and `transactionId`. Include `description`, `vendorContactId`, `vendorName`, or `topK` when available.
4. Present the recommendations as account name, account ID, source, accuracy, and any incompatible hint.
5. Ask the user to confirm the exact transaction and account before making a change.
6. Call `apply-transaction-cashcoding-recommendation` with `companyId`, `transactionId`, `accountId`, and `description`. Add `contactId` or `vendorName` when the transaction does not already carry a contact.
7. Summarize the applied result with the transaction, account, document ID, and line item details returned by MCP.

## Output Format

For recommendations:

| Account | Account ID | Source | Accuracy | Note |
|---------|------------|--------|----------|------|
| ... | ... | ... | ... | ... |

For applied cashcoding:

- Transaction: [transaction description or ID]
- Account: [account name] ([account ID])
- Result: applied
- Document ID: [document ID when returned]

## Error Handling

| Scenario | Response |
|----------|----------|
| Missing transaction ID | Ask for the OSOME transaction page URL or numeric transaction document ID |
| No recommendations | Explain that OSOME did not return a recommendation and suggest manual review in OSOME |
| Incompatible recommendation | Show the incompatible hint and do not apply it without explicit user confirmation |
| Missing write scope | Tell the user the MCP connection needs `company:documents:write` to apply cashcoding |
| Already reconciled transaction | Explain that the transaction appears already reconciled and should be reviewed in OSOME |

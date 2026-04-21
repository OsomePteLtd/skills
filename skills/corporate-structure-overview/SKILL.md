---
name: corporate-structure-overview
description: View company directors, shareholders, and subsidiaries. Use when the user asks about ownership, corporate structure, directors, or group companies. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Corporate Structure Overview

Understand ownership, directors, and subsidiaries.

## When to Use

Invoke this skill when the user asks:
- "Who are my directors?"
- "Who owns the company?"
- "Shareholder breakdown"
- "Ownership structure"
- "Cap table"
- "List my subsidiaries"
- "Group structure"
- "Corporate structure for investors"
- "Company officers"
- "Who's on the board?"
- "Equity split"

## MCP Server

This skill requires the OSOME MCP server:
```
https://mcp.osome.com/mcp
```

## Tools Used

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `list-companies` | List accessible companies |
| 2 | `get-company` | Company details and fiscal year |
| 3 | `get-directors-shareholders` | Directors and shareholders |
| 4 | `get-corporate-subsidiaries` | Subsidiary companies |
| 5 | `get-trial-balance` | Financial summary per entity (optional) |

## Execution Flow

1. Call `list-companies` to get all accessible companies
2. If user asks about specific company:
   - Call `get-company` for details
   - Call `get-directors-shareholders`
   - Call `get-corporate-subsidiaries`
3. If multiple companies exist:
   - Show group structure overview
   - For each entity, call `get-directors-shareholders`
4. Optionally add financial context with `get-trial-balance`
5. Present hierarchical corporate structure

## Output Format

### Single Company View

**Company:** [Company Name]
**Registration:** [Registration Number]
**Jurisdiction:** [SG/HK/UK/UAE]
**Incorporated:** [Date]
**Fiscal Year End:** [Date]

---

**Directors:**
| Name | Role | Appointed | Status |
|------|------|-----------|--------|
| ... | Director / Managing Director | [date] | Active |

**Shareholders:**
| Name | Shares | Percentage | Type |
|------|--------|------------|------|
| ... | [number] | [%] | Ordinary |

**Total Issued Capital:** [amount]

---

### Group Structure View

```
[Holding Company Name]
├── [Subsidiary 1] (100% owned)
│   └── [Sub-subsidiary] (100%)
├── [Subsidiary 2] (75% owned)
└── [Associate Company] (30% owned)
```

**Group Summary:**
| Entity | Jurisdiction | Ownership | Directors |
|--------|--------------|-----------|-----------|
| Holding Co | SG | - | 3 |
| Sub 1 | SG | 100% | 2 |
| Sub 2 | HK | 75% | 2 |

---

### Financial Context (if requested)

**Consolidated Position:**
| Entity | Assets | Liabilities | Equity |
|--------|--------|-------------|--------|
| Holding | SGD 1.2M | SGD 400K | SGD 800K |
| Sub 1 | SGD 500K | SGD 200K | SGD 300K |
| Sub 2 | HKD 2M | HKD 800K | HKD 1.2M |

## Examples

**User:** "Who are my company directors?"

**Response:**
> **ACME Pte Ltd - Directors**
>
> | Name | Role | Since |
> |------|------|-------|
> | John Smith | Managing Director | Jan 2020 |
> | Jane Doe | Director | Mar 2021 |
> | Bob Chen | Director | Jun 2022 |
>
> All directors are currently active. The company has 3 directors which meets Singapore's requirement of at least 1 local director.

---

**User:** "What's my corporate structure?"

**Response:**
> **ACME Holdings Pte Ltd - Group Structure**
>
> ```
> ACME Holdings Pte Ltd (Singapore)
> ├── ACME Services Pte Ltd (100%)
> ├── ACME HK Limited (100%)
> └── TechPartner Pte Ltd (40% - Associate)
> ```
>
> **Ownership Summary:**
> - ACME Holdings is owned by John Smith (60%) and Jane Doe (40%)
> - 2 wholly-owned subsidiaries
> - 1 associate company (40% stake)
>
> **Directors across group:**
> - John Smith: Director in all 3 entities
> - Jane Doe: Director in Holdings and Services
> - Local HK Director: Director in ACME HK only

---

**User:** "Who owns the company?"

**Response:**
> **ACME Pte Ltd - Shareholders**
>
> | Shareholder | Shares | Percentage | Type |
> |-------------|--------|------------|------|
> | John Smith | 60,000 | 60% | Ordinary |
> | Jane Doe | 30,000 | 30% | Ordinary |
> | Employee Trust | 10,000 | 10% | Ordinary |
>
> **Capital Structure:**
> - Authorized: 100,000 ordinary shares
> - Issued: 100,000 ordinary shares
> - Par value: SGD 1.00
> - Total issued capital: SGD 100,000

## Compliance Notes

| Jurisdiction | Key Requirements |
|--------------|------------------|
| Singapore | At least 1 local resident director |
| Hong Kong | At least 1 individual director (any nationality) |
| UK | At least 1 director (16+ years old) |

## Error Handling

| Scenario | Response |
|----------|----------|
| No subsidiaries | "This company has no subsidiaries on record" |
| No shareholders | "Shareholder information not available" |
| Multiple companies | Show summary table and ask which to explore |

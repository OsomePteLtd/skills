---
name: corporate-structure-overview
description: View company directors and subsidiaries. Use when the user asks about corporate structure, directors, or group companies. Requires OSOME MCP server at https://mcp.osome.com/mcp
---

# Corporate Structure Overview

View directors and subsidiary companies.

## When to Use

Invoke this skill when the user asks:
- "Who are my company directors?"
- "Who are my directors?"
- "What's my corporate structure?"
- "List my subsidiaries"
- "Show me the group structure"
- "Company officers"
- "Who's on the board?"
- "List directors"
- "Group companies"

## Limitations

The MCP server provides basic director and subsidiary information only:
- Director names, roles, and jurisdictions
- Subsidiary company names

**Not available via MCP:**
- Shareholding percentages (cap table)
- Share counts or classes
- Appointment dates
- Ownership percentages between entities

For detailed shareholding information, users should contact their OSOME account manager or check ACRA/company registry records directly.

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
| 3 | `get-directors-shareholders` | Directors list (names, roles, jurisdiction) |
| 4 | `get-corporate-subsidiaries` | Subsidiary companies |

## Execution Flow

1. Call `list-companies` to get all accessible companies
2. If user asks about specific company:
   - Call `get-company` for details
   - Call `get-directors-shareholders`
   - Call `get-corporate-subsidiaries`
3. If multiple companies exist:
   - Show group structure overview
   - For each entity, call `get-directors-shareholders`
4. Present the corporate structure

## Output Format

### Single Company View

**Company:** [Company Name]
**Jurisdiction:** [SG/HK/UK/UAE]
**Fiscal Year End:** [Date]

---

**Directors:**
| Name | Role | Jurisdiction |
|------|------|--------------|
| ... | Director / Managing Director | SG/HK/UK |

---

### Group Structure View

**[Holding Company Name]**
- Subsidiary: [Subsidiary 1]
- Subsidiary: [Subsidiary 2]

**Directors across group:**
| Name | Role | Companies |
|------|------|-----------|
| ... | ... | [list] |

## Examples

**User:** "Who are my company directors?"

**Response:**
> **ACME Pte Ltd - Directors**
>
> | Name | Role | Jurisdiction |
> |------|------|--------------|
> | John Smith | Managing Director | SG |
> | Jane Doe | Director | SG |
> | Bob Chen | Director | SG |
>
> The company has 3 directors. Singapore requires at least 1 local resident director.

---

**User:** "What's my corporate structure?"

**Response:**
> **ACME Holdings Pte Ltd - Group Structure**
>
> **Subsidiaries:**
> - ACME Services Pte Ltd
> - ACME HK Limited
>
> **Directors:**
> | Name | Role |
> |------|------|
> | John Smith | Managing Director |
> | Jane Doe | Director |
>
> For detailed shareholding percentages, please check your ACRA BizFile records or contact your OSOME account manager.

---

**User:** "Who owns the company?" / "Cap table?"

**Response:**
> Detailed shareholding information (cap table) is not available through this tool.
>
> **What I can show you:**
> - Directors: [list names and roles]
> - Subsidiaries: [list if any]
>
> **For shareholding details:**
> - Singapore: Check ACRA BizFile+ (https://www.bizfile.gov.sg)
> - Hong Kong: Check Companies Registry (https://www.cr.gov.hk)
> - Or contact your OSOME account manager

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
| No directors found | "Director information not available" |
| Multiple companies | Show summary table and ask which to explore |
| Cap table request | Explain limitation and suggest alternatives |

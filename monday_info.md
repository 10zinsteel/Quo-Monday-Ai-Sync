# Monday.com Integration Reference

## What You Need to Collect

### 1. Personal API Token
**Where to get it:**
1. Log into Monday.com
2. Click your avatar (top-right corner)
3. Go to **Developers** → **My Access Tokens**
4. Click **Show** → copy the token

**Used for:** authenticating all API calls from n8n
**Store in:** n8n credential (Monday.com API)

---

### 2. CRM Board ID
**Where to get it:**
1. Open the CRM board in your browser
2. Look at the URL: `https://yourworkspace.monday.com/boards/1234567890`
3. The number at the end is your Board ID

**Used for:** telling n8n which board to create/update leads on

---

### 3. Column IDs
Monday.com uses internal column IDs (not display names) in the API. You need the ID for each column you want to read or write.

**How to get them — Option A (easiest):**
1. Open your CRM board
2. Click any column header → **Column Settings** → **Customize**
3. The URL will show the column ID, e.g. `column_id=phone_mkm5h`

**How to get them — Option B (API):**
Run this GraphQL query (via Postman or n8n HTTP Request node):
```graphql
{
  boards(ids: [YOUR_BOARD_ID]) {
    columns {
      id
      title
      type
    }
  }
}
```
POST to `https://api.monday.com/v2` with header `Authorization: YOUR_TOKEN`

**Columns needed for the Quo → Monday sync:**

| Column Purpose | Expected Type | ID (fill in) |
|---|---|---|
| Lead name (item name) | Item Name | *(built-in, no ID needed)* |
| Phone number | Phone | |
| Lead status | Status | |
| Call notes | Long Text | |
| Follow-up date | Date | |
| Priority | Status or Dropdown | |
| Lead source | Dropdown | |

---

## Authentication

- **Type:** Personal API Token (Bearer)
- **Header:** `Authorization: YOUR_TOKEN`
- **API endpoint:** `https://api.monday.com/v2` (GraphQL, POST only)

---

## n8n Integration

### Native Monday.com Node
n8n has a built-in Monday.com node. Operations used in this workflow:
- **Create Item** — creates a new lead row on the board
- **Change Multiple Column Values** — updates an existing lead's fields
- **Create Update** — adds an activity log comment to a lead item

### GraphQL via HTTP Request Node (for duplicate check)
Used to search for an existing lead by phone number before creating a new one:
```graphql
{
  items_by_column_values(
    board_id: YOUR_BOARD_ID,
    column_id: "phone",
    column_value: "+1234567890"
  ) {
    id
    name
  }
}
```

---

## Status Labels (must match exactly what's in your board)

The AI will output one of these lead statuses — the labels must match your board's Status column labels exactly:

| AI Output | Monday.com Label | Meaning |
|---|---|---|
| `Interested` | `Interested` | CCI code word detected |
| `Callback Requested` | `Callback Requested` | Lead asked to be called back |
| `Not Interested` | `Not Interested` | Lead declined |
| `Needs Review` | `Needs Review` | No clear signal, needs human review |

> If your board uses different label names, update these in the n8n workflow's AI prompt.

---

## Pricing / API Limits
- Monday.com API is included with all paid plans
- Rate limit: 5,000 requests/minute (effectively unlimited for this use case)
- GraphQL API only (no REST endpoints)

# Quo API Reference

## Overview
Quo is a business phone number service. Its API is built on OpenPhone's infrastructure.

## Base URL
```
https://api.openphone.com
```
> Confirm this in Quo's app under Settings → Developer/Integrations. It may be `https://api.quo.com`.

## Authentication
- Type: API Key (header-based)
- Header: `Authorization: YOUR_API_KEY`
- Requirement: Active Quo subscription required for API access

## Getting Your API Key
1. Log into your Quo account
2. Go to Settings → Developer / Integrations
3. Generate an API key
4. Store it securely (add to `.env` file, never commit to git)

---

## Endpoints

### Contacts
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/contacts` | List all contacts |
| GET | `/v1/contacts/:id` | Get contact by ID |
| POST | `/v1/contacts` | Create a new contact |
| PATCH | `/v1/contacts/:id` | Update a contact |
| DELETE | `/v1/contacts/:id` | Delete a contact |
| GET | `/v1/contact-custom-fields` | Get custom field definitions |

**Contact body example:**
```json
{
  "defaultFields": {
    "firstName": "John",
    "lastName": "Doe",
    "phoneNumbers": [{ "name": "primary", "value": "+1234567890" }],
    "emails": [{ "name": "work", "value": "john@example.com" }],
    "role": "Manager",
    "company": "Acme Corp"
  },
  "customFields": {}
}
```
> Phone numbers must be in E.164 format (e.g. `+12025551234`).
> Custom field definitions can only be created/edited inside the Quo app, not via API.

---

### Messages
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/messages` | List messages |
| GET | `/v1/messages/:id` | Get message by ID |
| POST | `/v1/messages` | Send an SMS |

---

### Calls
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/calls` | List calls (filter by phoneNumberId, participants) |
| GET | `/v1/calls/:callId` | Get call by ID |
| GET | `/v1/call-recordings/:callId` | Get recordings for a call |
| GET | `/v1/call-summaries/:callId` | Get AI summary (Business/Scale plans only) |
| GET | `/v1/call-transcripts/:id` | Get full call transcript with speaker timestamps |

---

### Conversations
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/conversations` | List conversations |

---

### Phone Numbers
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/phone-numbers` | List all phone numbers |
| GET | `/v1/phone-numbers/:id` | Get phone number by ID |

---

### Users
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/users` | List users |
| GET | `/v1/users/:id` | Get user by ID |

---

### Webhooks
| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/webhooks` | List all webhooks |
| POST | `/v1/webhooks` | Create a webhook |
| GET | `/v1/webhooks/:id` | Get webhook by ID |
| DELETE | `/v1/webhooks/:id` | Delete a webhook |

**Webhook event types:**
- `calls` — incoming/outgoing call events
- `messages` — incoming/outgoing message events
- `call.summary` — AI call summary completed
- `call.transcript` — call transcript completed

---

## Pricing
- Domestic SMS (US/Canada): $0.01 per segment
- International SMS: $0.01 + country-specific rate per segment
- Segment limits: 160 chars (standard) / 70 chars (if emojis or special characters)
- MMS is not supported
- Active subscription required for API access

---

## n8n Integration Strategy

### Option 1 — HTTP Request Node (Scheduled/Triggered)
Use for: creating/updating contacts, sending messages, fetching call logs
- Node: **HTTP Request**
- **Method**: `GET` (or `POST`/`PATCH` for write operations)
- **URL**: e.g. `https://api.openphone.com/v1/calls`
- **Authentication**: `Header Auth`
  - **Header name**: `Authorization`
  - **Header value**: `<API key>`
- Example: Monday.com new item → HTTP Request → POST `/v1/contacts`

### Option 2 — Webhook Node (Real-Time)
Use for: reacting to incoming calls or messages instantly
- Node: **Webhook** (receive) + register URL via POST `/v1/webhooks`
- Example: Quo incoming call → n8n Webhook → update Monday.com item

---

## OpenAPI Spec
Full spec (JSON): `https://openphone-public-api-prod.s3.us-west-2.amazonaws.com/public/openphone-public-api-v1-prod.json`

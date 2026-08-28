---
name: bruin-api-integration
description: >-
  Help a client build, debug, or extend an integration with the Bruin Public
  API — MetTel's REST API for tickets, inventory, sites, users, and webhooks.
  Use when the task involves OAuth 2.0 authentication against Bruin, listing or
  looking up inventory, creating a ticket with the correct note-type payload for
  a product (Smart Phones, Cable/Ethernet/Business-Line/Starlink circuits,
  SD-WAN, PIAB), managing sites or users, or subscribing to ticket webhooks.
  Also use for questions about Bruin Client IDs, bearer tokens, required scopes,
  or ticket category codes.
---

# Bruin Public API Integration

The Bruin Public API is a standard REST API secured with **OAuth 2.0 client
credentials**. It lets a client programmatically manage tickets, read
inventory, manage sites and users, and receive webhook notifications about
ticket activity. This skill turns MetTel's published docs into a working
reference for building and debugging those integrations.

You are assisting a **client's technical team**. MetTel does not build custom
integrations for clients — your job is to explain the API accurately, generate
correct request payloads, and debug their calls. When something can only be done
inside the Bruin portal (generating credentials, registering webhooks) or
requires MetTel to provision access, say so and point them to their **Customer
Software Engineer (CSE)**.

## The two things people confuse first

1. **OAuth Client ID vs. Bruin Client ID** — these are different values.
   - **OAuth Client ID / Secret** — credentials generated in the Bruin portal
     (My Company → Developer Configuration → API Access). Used **only** to
     request a bearer token.
   - **Bruin Client ID** — a numeric ID for the client's organization inside
     Bruin (e.g. `9994`). Required as a parameter/body field in **almost every**
     API call. When an endpoint asks for `clientId`, it means this one.
   Their CSE provides the Bruin Client ID.

2. **Auth endpoint vs. API base URL** — tokens come from an *auth* host; API
   calls go to a different *API* host. Both differ by environment. See
   [references/authentication.md](references/authentication.md).

## Core request pattern

Every API call is the same two steps:

1. **Get a bearer token** — `POST` client credentials to the environment's auth
   endpoint. Tokens last **3600s (1 hour)**; refresh before expiry, don't wait
   for a 401.
2. **Call an endpoint** — send the bearer token in the `Authorization: Bearer`
   header, and pass the **Bruin Client ID** as a parameter/body field.

```bash
# 1. Token
curl -X POST https://apigw.bruin.com/authorize/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=OAUTH_CLIENT_ID" \
  -d "client_secret=OAUTH_CLIENT_SECRET" \
  -d "scope=public_api"

# 2. Call (Bruin Client ID as a query param)
curl -X GET "https://api.bruin.com/api/Inventory?ClientId=9994" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

Each generated credential is scoped to specific endpoints via
`FunctionPermission*` scopes. A `403` means the credential lacks the scope for
that endpoint — the fix is a portal/CSE change, not a code change.

## Creating a ticket — the heart of the API

Most work is created through tickets. There are **three write endpoints** —
pick the right one first:

- `POST /api/Ticket` — repairs, changes, disconnects, offboarding. A `category`
  code + flat `notes[]`. **The valid `category` and required `notes` differ by
  product and operation** — that's what the per-product references cover.
- `POST /api/Ticket/PlaceOrder` — wireless **device orders** (SKUs + add-ons).
- `POST /api/Ticket/NewOrder` — **PIAB orders** (recursive item tree).

Read [references/ticket-model.md](references/ticket-model.md) first for the
universal `POST /api/Ticket` body and note-type conventions, then open the
per-product reference. For the two order endpoints and ticket-detail/topic
reads, see [references/ordering.md](references/ordering.md).

## How to navigate this skill

Load only the reference you need for the task in front of you.

| Task | Open |
| --- | --- |
| Authenticate, environments, scopes, token lifecycle | [references/authentication.md](references/authentication.md) |
| Understand the universal ticket body + note types | [references/ticket-model.md](references/ticket-model.md) |
| Place device/PIAB orders, list topics, read ticket details | [references/ordering.md](references/ordering.md) |
| List inventory / look up an `inventoryID` / get attributes (BTN, IMEI, PIC) | [references/inventory.md](references/inventory.md) |
| Read or create sites (locations) | [references/site.md](references/site.md) |
| Read, create, or update users (assignees/contacts) | [references/user.md](references/user.md) |
| Receive/parse ticket lifecycle webhooks | [references/webhooks.md](references/webhooks.md) |
| **Build a ticket payload for a specific product** | `references/tickets/<product>.md` (see below) |
| Multi-call end-to-end workflows | `references/workflows/*.md` |

### Per-product ticket references

Each file lists every documented operation for that product, its `category`
code, and the full note-type schema (required / optional / conditional notes).

| Product | Reference |
| --- | --- |
| Smart Phones (wireless) | [references/tickets/smart-phones.md](references/tickets/smart-phones.md) |
| Cable Internet | [references/tickets/cable-internet.md](references/tickets/cable-internet.md) |
| Ethernet Internet | [references/tickets/ethernet-internet.md](references/tickets/ethernet-internet.md) |
| Business Line (voice) | [references/tickets/business-line.md](references/tickets/business-line.md) |
| Starlink | [references/tickets/starlink.md](references/tickets/starlink.md) |
| SD-WAN | [references/tickets/sd-wan.md](references/tickets/sd-wan.md) |
| PIAB (Phone-in-a-Box) | [references/tickets/piab.md](references/tickets/piab.md) |

### Workflows

| Workflow | Reference |
| --- | --- |
| Cell phone lifecycle (new / existing / offboard) | [references/workflows/cell-phone-lifecycle.md](references/workflows/cell-phone-lifecycle.md) |
| Ordering PIAB | [references/workflows/ordering-piab.md](references/workflows/ordering-piab.md) |
| Wireline repair tickets | [references/workflows/repair-tickets.md](references/workflows/repair-tickets.md) |

## Guardrails for helping clients

- **Never invent note types or category codes.** If an operation isn't in the
  references, tell the client it isn't documented and to check the
  [Swagger](https://api.bruin.com/index.html) or ask their CSE. The docs
  intentionally cover only a subset of Swagger — an undocumented endpoint may
  still work, but don't fabricate its schema.
- **Enforce conditional notes.** Many notes are only required when a parent note
  takes a specific value (e.g. `PhoneNumberChange = "Request New Number"`
  requires `DSNPANXX`). Check the product reference before emitting a payload.
- **Every ticket needs a Site contact.** A `Ticket` contact is optional (falls
  back to the Site contact). Contacts must map to a Bruin account by email, or
  include `name` + `phone` to be created.
- **`category` topics are per-account.** The authoritative list for a client is
  `GET /api/ticket/topics`. The codes in these references are the standard set.
- **Placeholders, not secrets.** Use `OAUTH_CLIENT_ID`, `ACCESS_TOKEN`, and a
  sample Bruin Client ID in examples — never real credentials.

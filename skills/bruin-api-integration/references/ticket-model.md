# The Ticket Model

A **ticket** is the core work item in Bruin. It tracks something that needs to
be investigated, repaired, ordered, or changed on one or more services. Almost
every write operation in the API is "create a ticket."

## Endpoint

```
POST https://api.bruin.com/api/Ticket
```

Send `Content-Type: application/json` and the bearer token in the
`Authorization` header.

## Universal request body

Every ticket, regardless of product, has the same four sections:

```json
{
  "clientId": 9994,
  "category": "017",
  "Services": [
    { "ServiceNumber": "3059473030" }
  ],
  "notes": [
    { "noteType": "MTK", "noteValue": "Problem description here" }
  ],
  "contacts": [
    { "type": "Site",   "email": "site@example.com",   "name": "Site Contact",   "phone": "9827783377" },
    { "type": "Ticket", "email": "ticket@example.com", "name": "Ticket Contact", "phone": "9827783374" }
  ]
}
```

### Top-level fields

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `clientId` | integer | Yes | The **Bruin Client ID** (not the OAuth Client ID). |
| `category` | string | Yes | Ticket topic key. Differs by product + operation (e.g. `"004"`, `"VAS"`, `"020"`). Authoritative per-account list: `GET /api/ticket/topics`. |

### `Services` (array, ≥1 required)

Identifies which inventory the ticket is opened against.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `ServiceNumber` | string | Yes | The WTN (Working Telephone Number) or circuit ID of the Bruin inventory item. |
| `ServiceEvent` | string | No | Event type (e.g. `"Jitter"`, `"PacketLoss"`). |
| `ServiceInterfaces` | string[] | No | Interfaces, e.g. `["GE1", "GE2"]`. |

Use `GET /api/Inventory` to resolve a `ServiceNumber` — see
[inventory.md](inventory.md).

### `notes` (array, optional at the model level)

This is where product-specific data goes. **Which notes are required depends on
the `category` and often on the value of other notes.** See the per-product
references. Each note:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `noteType` | string | Yes (per note) | The note's type key, e.g. `"MTK"`, `"DDD"`, `"PhoneNumberChange"`. |
| `noteValue` | string | Usually | The structured value (a selection key or free text). |
| `noteText` | string | Sometimes | Free-text / display content for some note types. |

> Field names are **case-insensitive** in practice (`noteType` and `NoteType`
> both appear in MetTel's own examples). Prefer the casing shown in the
> product reference you're following.

### `contacts` (array)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `type` | string | Yes | `"Site"` or `"Ticket"`. |
| `email` | string | Yes | Must be valid. If it matches an existing Bruin user, that profile is used. |
| `phone` | string | Conditional | Required only if the contact isn't already a Bruin user. |
| `name` | string | Conditional | `First Last`. Required only if the contact isn't already a Bruin user. |

Rules:
- **A `Site` contact is always required.** Requests without one are rejected.
- If no `Ticket` contact is given, the `Site` contact is used for both.
- To be usable as a contact, an email must belong to a Bruin account (or supply
  `name` + `phone` so one can be created).

## Common note types (shared across products)

These recur in many ticket types:

| noteType | Meaning |
| --- | --- |
| `MTK` | Main ticket note / problem description (free text). |
| `DDD` | Desired Due Date (`MM/DD/YYYY`). |
| `TroubleStart` | Date trouble started (`MM/DD/YYYY`). |
| `LDSCR` | Line description (free text label). |

## How to read the per-product references

Each product reference lists operations. For every operation you get:

- **Operation name + `category` code + subcategory ID** (when documented).
- A **copy-pasteable example body**.
- A **note schema** using this legend:

  | Badge | Meaning |
  | --- | --- |
  | **Required** | The note must be present. |
  | **Optional** | The note may be omitted. |
  | **Conditional** | Required only when a parent note takes a specific value. |

- **Conditional notes are shown nested under the parent value that triggers
  them.** Example: under `PhoneNumberChange`, choosing the value
  `"Request New Number"` makes `DSNPANXX` (desired area code) **required**;
  choosing `"Restore Previous Number"` makes `NumbertoRestore` **required**
  instead.

When building a payload: pick the operation, include all **Required** notes,
resolve every **Conditional** based on the values you chose, then add any
**Optional** notes the client wants.

## Validation error shape

Failed writes return the standard error schema:

```json
{
  "message": "An error occurred while processing your request.",
  "messageDetail": "…",
  "data": { "ClientId": ["ClientId is required."] },
  "type": "ValidationError",
  "code": 500,
  "traceId": "0HMV4T9P1G2K5:00000001"
}
```

`data` maps a field name → the validation messages for it. Give the `traceId`
to MetTel support when escalating.

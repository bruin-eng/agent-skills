# Webhooks (ticket lifecycle events)

Webhooks push ticket events to a client's callback URL so external systems can
track ticket progress without polling. Use them to notify end users of
milestones, extract project-timeline data, and track outage incidents.

## Enrollment (portal, not API)

Clients register webhooks in the Bruin portal: My Company → Developer
Configuration → **Webhook Access → Register Webhooks**. You cannot register
them via API — direct the client to the portal / their CSE.

The registration form asks for:

- **Name** — free label (set it to the use case).
- **Callback URL** — where events are `POST`ed.
- **Authentication Method** — one of:
  - **Basic** — client provides a username + password (any values).
  - **Secret Token** — client provides a token **> 34 and < 64 characters**.

> To test, point the callback at a scratch endpoint like
> [webhook.site](https://webhook.site/), or use a Postman POST to simulate
> events to your own callback.

## Event lifecycle

A typical ticket flows:

```
Created → NewNoteAdded → NewNoteAdded → … → Resolved → Closed
```

- **Resolved** starts a **2-business-day** timer, after which the ticket
  auto-transitions to **Closed**.
- **Resolved** is reversible (**Unresolved**); **Closed** is final — a new issue
  needs a new ticket.
- **Recommendation:** track ticket state off **Resolved** and/or **Closed**. Use
  **Closed** when you must be certain all tasks are complete.

## Events (the `Action` field)

| `Action` | When it fires |
| --- | --- |
| `Created` | A new ticket is created. Payload includes the full order/notes snapshot — an "eye in the sky" for all tickets on the account. |
| `NewNoteAdded` | A note is added (repair updates, shipping labels, care-team notes, RFO/resolution notes, …). High volume and varied. |
| `Resolved` | Ticket moved to resolved (all tasks complete). Lifecycle-only payload. |
| `Unresolved` | A resolved ticket was pulled back into work (common for recurring repair outages). |
| `UnsuccessfulResolved` | The ticket was **cancelled** by MetTel (wrong ticket type, duplicate). Closes all tasks; use it to prevent stale records in your system. |
| `Closed` | Ticket closed (manually, or auto 2 business days after Resolved). Final. |

## Payload envelope (all events)

Every delivery has the same outer shape:

```json
{
  "Id": "6a1c169a-d518-4495-b6b2-9e7caae631c5",
  "Attempt": 1,
  "Notification": {
    "Id": "…|bp…|pti…|ti…",
    "ClientId": 9994,
    "EntityId": "11423324",
    "ApplicationName": "Ticket",
    "Action": "Created",
    "Body": { "TicketId": 11423324, "Category": "VAS", "ClientId": 9994, "...": "..." }
  }
}
```

Envelope fields to key on:

| Field | Meaning |
| --- | --- |
| `Id` | Unique delivery ID. |
| `Attempt` | Delivery attempt number (retries increment it) — use for idempotency. |
| `Notification.Action` | The event type (table above). |
| `Notification.EntityId` / `Body.TicketId` | The ticket ID. Correlate events across the lifecycle by this. |
| `Notification.ClientId` | Bruin Client ID. |
| `Body.Category` | The ticket's category code. |
| `Body.EnteredByUsername` | Who/what triggered the event (a user email, or a MetTel service account like `RequestProcessService`). |

## `Body` differences by event

- **`Created`** — large payload: `Body.TicketMessage.order` with `items[]`
  (product, `inventoryId`, `wtn`, `attributes`), `notes[]`, `contact{}`,
  `subCategoryNotes[]`, `subscribers[]`, plus system `notes[]`. This is where you
  read what the ticket actually contains.
- **`NewNoteAdded`** — `Body.NewNoteIds[]`, `Body.DetailIds[]`, and
  `Body.NoteMessages[]`, each note: `{ NoteID, NoteType, NoteValue,
  ServiceNumbers[], AttachmentExisted, EnteredDate }`. Notes vary enormously —
  **filter on the `NoteType`s you care about.** Useful signals seen in the wild:
  - `RFO` — Reason For Outage (e.g. `"No Trouble Found"`)
  - `ResolutionTime` — when it was resolved
  - `CLS` — closed marker
  - `ReturnLabel` — return shipping label link/QR (for device returns)
  - `ADN` — a plain added note
- **`Resolved` / `Unresolved` / `Closed`** — lightweight lifecycle payloads:
  `TicketId`, `Category`, `ClientId`, `DetailIds[]`, `EnteredByUsername`.
- **`UnsuccessfulResolved`** — like the above but with `NewDetailId` (singular).

## Handling guidance

- **Idempotency:** dedupe on `Notification.Id` (or `Id` + `Attempt`); retries
  re-deliver the same event.
- **Correlate** all events for a ticket by `Body.TicketId` / `EntityId`.
- **Don't assume a fixed note set** on `NewNoteAdded` — allow-list the
  `NoteType`s you ingest and ignore the rest.
- **Respond `2xx` quickly**, then process async, so retries aren't triggered by
  slow handlers.

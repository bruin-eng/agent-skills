# Workflow: Wireline Repair Tickets

Open a repair ticket against an existing service. A standard `POST /api/Ticket`
([../ticket-model.md](../ticket-model.md)) — the `category` and product-specific
`notes` depend on the service type. Uses [../inventory.md](../inventory.md) and
[../webhooks.md](../webhooks.md).

---

## Two repair categories (all products)

Every repair is one of:

- **`VAS` — Service Affecting Trouble** — a problem that isn't stopping
  operations (degraded/intermittent).
- **`VOO` — Service Outage Trouble** — an outage needing urgent attention (dead
  circuit, dead fire-alarm line).

The category is the same idea across products; only the note schema differs by
product.

---

## Steps

1. **Identify the affected service** — get the `serviceNumber` (WTN or circuit
   ID). Recommended: `GET /api/Inventory` filtered by `SiteID` (or fetch broadly
   and filter by `siteLabel`/address). Capture `serviceNumber`.
2. **Pick the category** — `VAS` or `VOO`.
3. **Build the notes** — use the product's ticket reference for the exact note
   schema and conditionals:
   - [../tickets/cable-internet.md](../tickets/cable-internet.md)
   - [../tickets/ethernet-internet.md](../tickets/ethernet-internet.md)
   - [../tickets/business-line.md](../tickets/business-line.md)
   - [../tickets/starlink.md](../tickets/starlink.md)
   - [../tickets/sd-wan.md](../tickets/sd-wan.md)
   - [../tickets/piab.md](../tickets/piab.md) (specialty business lines)

   Common repair notes: `MTK` (problem description), `DDD` (desired due date),
   `TroubleStart` (when trouble began). Product-specific ones vary — e.g. cable
   uses `cableproblem` / `rebootmodem` / `Light Status` / `fieldtech`; PIAB
   specialty lines use `Require ISW`.
4. **Field tech / intrusive testing** — where a note controls dispatch (e.g.
   `fieldtech`, `Require ISW`), setting it to `false`/"no immediate dispatch" is
   generally recommended: it lets MetTel investigate first and avoids
   potentially billable technician hours. It does not block a dispatch later.
5. **Submit** — `POST /api/Ticket`. Response includes `ticketId` /
   `ticketStatus` (`"O"`). Store it.

---

## Tracking

- **`Created`** — MetTel's own monitoring can auto-create repair tickets. If you
  want your system to mirror all repairs (not only ones you opened), subscribe
  to `Created` and filter on `Category` `VAS`/`VOO`.
- **`NewNoteAdded`** — stream updates into your system; watch for `RFO` (reason
  for outage), `ResolutionTime`, and other status notes.
- **`Closed`** — close the corresponding ticket on your side; no further action
  occurs after close.

> A resolved repair can be **Unresolved** if an outage recurs shortly after —
> handle the `Unresolved` event if you track resolved state.

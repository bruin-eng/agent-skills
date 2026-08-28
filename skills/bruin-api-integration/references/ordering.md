# Ordering & Ticket Endpoints

There are **three different write endpoints** for creating work in Bruin, plus
read endpoints for topics and ticket details. Picking the right one is the first
decision.

| Endpoint | Use for | Body shape |
| --- | --- | --- |
| `POST /api/Ticket` | Repairs, changes, disconnects, offboarding — anything driven by a `category` + flat `notes[]`. | The universal ticket model ([ticket-model.md](ticket-model.md)). |
| `POST /api/Ticket/PlaceOrder` | **Smart phone / wireless device orders** (new and refresh). | `scenario` + `items[]` with SKUs & add-ons. |
| `POST /api/Ticket/NewOrder` | **PIAB (POTS-in-a-Box) orders**. | `orderType` + recursive `items[]` with `subItems` and `noteLevel` notes. |

All three return a ticket ID you must store for webhook correlation and detail
lookups.

---

## `POST /api/Ticket` responses

Two response shapes appear depending on the operation:

```json
{ "ticketIds": [11584486], "message": "Succeed for serviceNumber: 8008769823. TicketId: 11584486.\n" }
```
```json
{ "ticketId": 11477819, "ticketStatus": "O", "createdTime": "2026-06-05T16:29:01.48-04:00", "requireApproval": false }
```

---

## `POST /api/Ticket/PlaceOrder` — device orders (wireless)

Used to order phones/devices with service. See
[workflows/cell-phone-lifecycle.md](workflows/cell-phone-lifecycle.md) for the
end-to-end flow.

### `scenario` (top-level, required)

| Value | Meaning |
| --- | --- |
| `deviceAndService` | **New** device + new line. |
| `changeDevice` | Refresh device, **keep** the same carrier/service. |
| `changeDeviceAndService` | Refresh device **and** change carrier/plan. |

### Body outline

```json
{
  "clientId": 9994,
  "scenario": "deviceAndService",
  "region": "US",
  "customerReference": "PO-12345",
  "orderContact": { "firstName": "Kyle", "lastName": "Smith", "email": "ksmith@example.com", "phoneNumber": "555-000-0000" },
  "serviceAddressId": 248090,
  "ticketSubscribers": [ { "email": "pm@example.com" } ],
  "shippingAddress": { "street": "170 S Main St", "city": "Salt Lake City", "state": "UT", "zip": "84101" },
  "deliveryPreferences": { "shippingMethod": "Ground", "specialInstructions": "Leave at front desk", "attentionTo": "Seth Altman" },
  "items": [
    {
      "sku": "SM-A146UZKDXAA",
      "quantity": 1,
      "carrier": "Verizon",
      "purchaseOption": "Equipment Purchase",
      "phoneNumbers": ["7755445345"],
      "existingDeviceDecision": "keep",
      "userInfo": { "firstName": "Seth", "lastName": "Altman", "email": "saltman@example.com" },
      "addOns": [
        { "sku": "MTL-20W-WC", "quantity": 1, "purchaseOption": "Equipment Purchase" },
        { "sku": "UNLTLKTXT4GBPOOL" }
      ]
    }
  ],
  "billingAccount": "419071"
}
```

### Field notes

| Field | Notes |
| --- | --- |
| `clientId` | Bruin Client ID (hardcode). |
| `region` | Hardcode (`"US"`). |
| `customerReference` | Your reference (often a ServiceNow RITM/REQ). |
| `orderContact` | Must be a Bruin user; receives progress emails. |
| `serviceAddressId` | The **addressId** the line bills to (from `POST`/`GET /api/Site`). Often hardcoded to HQ. |
| `ticketSubscribers` | Optional extra notification emails; each must be a Bruin user. |
| `shippingAddress` | Optional; defaults to the service address. |
| `deliveryPreferences.shippingMethod` | `Ground` or `Overnight`. |
| `items[].sku` | Primary device SKU (from your CSE/account care). |
| `items[].carrier` | `AT&T`, `Verizon`, `T-Mobile`, or `Single SIM 1.0`. |
| `items[].purchaseOption` | `"Equipment Purchase"` (buy outright), or a recurring `"<Term> <Ownership>"` e.g. `"24 Months FINANCE"`, `"12 months RENT"`, `"36 months MDaaS"`. |
| `items[].userInfo` | End user the device is assigned to; must be a Bruin user. |
| `items[].phoneNumbers` | **Refresh only** — old device's number at index `[0]`. |
| `items[].existingDeviceDecision` | **Refresh only** — `keep`, `trade`, or `depot`. `trade`/`depot` generate a return label; `keep` may incur lease charges. |
| `items[].addOns[]` | Cases, chargers, and **the data plan** (plan SKU); add-ons may omit `purchaseOption` when implicit. |
| `billingAccount` | **Required to actually place the order.** Omit it to validate a payload without creating a real order (won't appear in the portal). |

Up to **60 devices** per order (the `items` array).

### Response

```json
{ "ticketId": 11553217, "pricing": { "oneTimeTotal": 2939.97, "monthlyTotal": 0.0 }, "items": [ { "sku": "...", "bruinSkuId": 117080, "productName": "...", "oneTimePrice": 2899.99, "addOns": [ ... ] } ] }
```

---

## `POST /api/Ticket/NewOrder` — PIAB orders

Used for POTS-in-a-Box orders (device kit + specialty business lines + dispatch
+ backboard). Structurally different from PlaceOrder: a **recursive `items[]`
tree** and **level-tagged notes**. See
[workflows/ordering-piab.md](workflows/ordering-piab.md) for the full walkthrough.

### Key structural rules

- `orderType`: fixed `"PIAB_NewOrder"`.
- Top-level: `clientId`, `addressId`, `siteId`, `contacts[]` (Site + Ticket,
  both must be Bruin users), `ticketCreatorUsername`, `items[]`, `notes[]`,
  `shippingOption`.
- **`items[].subItems` is recursive** — a SKU can contain sub-items which
  contain sub-items (device → lines/licenses → contract term). Keep `quantity`
  consistent across a parent and its sub-items.
- **Notes carry a `noteLevel`:**
  - `"Ticket"` — order-wide (in the top-level `notes[]`): `RefTicketNumber`,
    `MTK`, `BAN` (+`SubAccount` = `-1` to create one), `Hierarchy`, `SiteLabel`,
    `OrderAttributes` (`"{}"`), etc.
  - `"Subcategory"` — per-SKU (in that item's `itemNotes[]`): `ShippingAddress`
    (= addressID), `DDD`, `AttentionTo`.
  - `"Item"` — per-line, keyed by `noteForItemNo` (1, 2, 3…): `DirectoryListing`,
    `PhoneNumberType`, `APINPANXX`, `SeriesCompLine`, `SIPLine1`.
- `SIPLine1` line-type codes: `231` Voice · `245` Fire Alarm · `246` Burglar
  Alarm · `247` Modem · `248` Elevator · `250` Fax · `251` Elevator Modem.

### Response

```json
{ "ticketId": 11477819, "ticketStatus": "O", "createdTime": "2026-06-05T16:29:01.48-04:00", "requireApproval": false }
```

> Processing takes ~20 seconds. Store `ticketId`.

---

## `GET /api/ticket/topics` — list valid categories

Returns the authoritative set of `category` codes available for the account. Use
this to validate/enumerate categories rather than hardcoding.

---

## `GET /api/Ticket/{ticketId}/details` — ticket details

Call after an order/ticket completes (e.g. on the `Closed` webhook) to pull
provisioned line data.

- `ticketDetails[]` entries carry a `detailType` and `detailValue` — e.g.
  `detailType: "WTN"` gives the assigned phone number, plus
  `currentTaskName`/`currentTaskStatus`.
- For PIAB, look for notes whose `noteType` matches `FXS<n>Info` (e.g.
  `FXS1Info`) — the `noteValue` is a `|`-delimited string containing `DID:`
  (phone number) and `Line Type:` among other fields.

---

## Testing safely (no dev environment)

There is **no client-facing dev server** — you test in production, so guard
against real fulfillment:

- Coordinate with your account care rep / CSE to review, cancel, or simulate
  test orders.
- Set the `MTK` note to a clear warning, e.g.
  `"TESTING SOFTWARE, DO NOT TAKE ANY ACTION ON THIS TICKET UNLESS INSTRUCTED BY AN ACCOUNT CARE REPRESENTATIVE."`
- For `PlaceOrder`, omitting `billingAccount` validates without placing a real
  order.

Skipping these can cause real device shipments, provisioned lines, technician
dispatches, and charges.

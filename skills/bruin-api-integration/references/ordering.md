# Ordering & Ticket Endpoints

There are **two documented write endpoints** for creating work in Bruin, plus
a read endpoint for topics. Picking the right one is the first decision. For
listing tickets or reading ticket details/PON status, see
[ticket-queries.md](ticket-queries.md).

| Endpoint | Use for | Body shape |
| --- | --- | --- |
| `POST /api/Ticket` | Repairs, changes, disconnects, offboarding: anything driven by a `category` + flat `notes[]` (including PIAB device/line repairs). | The universal ticket model ([ticket-model.md](ticket-model.md)). |
| `POST /api/Ticket/PlaceOrder` | Smartphones, accessories, **PIAB kits**, Specialty Business Lines, QuickOrder, and change scenarios. | `scenario` + `items[]` (or `quickOrderId`) + `serviceLines`. |

Both return a ticket ID you must store for webhook correlation and detail
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

## `POST /api/Ticket/PlaceOrder`: orders (wireless, PIAB, SBL)

Creates an end-to-end order ticket in one call: resolves/creates the order
contact (and optional site contact), resolves or creates the service address,
resolves billing/sub-account, prices every item and add-on, and returns a
`ticketId` plus a pricing summary.

See [workflows/cell-phone-lifecycle.md](workflows/cell-phone-lifecycle.md) for
the wireless new/refresh/offboard playbook, and
[workflows/ordering-piab.md](workflows/ordering-piab.md) for a new PIAB kit.

**Breaking change:** item-level `phoneNumbers` and `portExisting` are **no
longer accepted**. Use `items[].serviceLines[]` for SBL DIDs and for
change-scenario WTNs.

### `scenario` (top-level, required)

When a product has configured scenario options, the requested scenario must be
one of them. Alias `deviceOnly` is accepted and normalized to `device`.

| Value | Meaning |
| --- | --- |
| `deviceAndService` | **New-order** path: new smartphone, accessory, PIAB, or SBL. |
| `changeDevice` | Device swap on an existing line. Requires current WTN + `existingDeviceDecision` per primary. Existing plans/features carry forward. Only Device Management Service add-ons are accepted for the new device. |
| `changeDeviceAndService` | Device swap **and** service change. Same current-device rules. `addOns` add plans/features; `removeAddOns` (and single-select conflicts) remove them. |
| `changeService` | Service change **with no device swap**. Requires current WTN; `sku` is optional (derived from the line). `existingDeviceDecision` is ignored. No equipment charge. |

### Required fields

| Field | Requirement |
| --- | --- |
| `clientId` | Always. Bruin Client ID. |
| `scenario` | Always. |
| `region` | Always. Pass `"US"` (validated only; no other effect). |
| `orderContact` | Always: `firstName`, `lastName`, `email`, `phoneNumber`. |
| `items[]` **or** `quickOrderId` | Exactly one of the two. |
| `billingAccount` | Required when any item is a Specialty Business Line. |
| `items[].sku` | Required on every item except a `changeService` primary. Add-ons always require it. |
| `items[].serviceLines[0].phoneNumber` | Required per primary for `changeDevice`, `changeDeviceAndService`, and `changeService` (current device WTN). |
| `items[].existingDeviceDecision` | Required per primary for `changeDevice` and `changeDeviceAndService`. Ignored for `changeService`. |

### New smartphone example (`deviceAndService`)

Do **not** put `serviceLines` or `existingDeviceDecision` on a new smartphone
order; those are change-scenario fields.

```json
{
  "clientId": 1234,
  "scenario": "deviceAndService",
  "region": "US",
  "billingAccount": "449687",
  "serviceAddressId": 1926955,
  "customerReference": "PO-INVENTORY",
  "deliveryPreferences": { "shippingMethod": "Ground" },
  "orderContact": {
    "firstName": "Jordan", "lastName": "Reyes",
    "email": "jreyes@example.com", "phoneNumber": "555-000-0000"
  },
  "shippingAddress": {
    "street": "55 Water St.", "city": "New York", "state": "NY", "zip": "10041"
  },
  "items": [{
    "sku": "MA501LL/A",
    "quantity": 1,
    "purchaseOption": "Equipment Purchase",
    "inventoryType": "Customer",
    "userInfo": { "firstName": "Alex", "lastName": "User" },
    "addOns": [
      { "sku": "UNLTLKTXT2GBPOOL", "purchaseOption": "Month to Month" },
      { "sku": "SOME-CASE-SKU", "purchaseOption": "Equipment Purchase", "inventoryType": "Depot" }
    ]
  }]
}
```

### New PIAB example (`deviceAndService`)

Primary kit + `addOns`. Line identity is `serviceLines` on the Specialty
Business Line add-on (`quantity` must equal `serviceLines.length`).
`billingAccount` is required. Do **not** send nested `"Purchase"` SKUs,
`NOCONTRACT`/`36MONTH` as SKUs, per-line ticket notes, or an `IP-Port` item.
Full walkthrough: [workflows/ordering-piab.md](workflows/ordering-piab.md).

```json
{
  "clientId": 9994,
  "scenario": "deviceAndService",
  "region": "US",
  "customerReference": "PO-12345",
  "orderContact": {
    "firstName": "Kyle", "lastName": "Smith",
    "email": "ksmith@mettel.net", "phoneNumber": "555-000-0000"
  },
  "siteContact": {
    "firstName": "Eugene", "lastName": "Krabs",
    "email": "ekrabs@mettel.net", "phoneNumber": "8008769823"
  },
  "serviceAddressId": 2340046,
  "deliveryPreferences": { "requestedDate": "2026-05-30", "shippingMethod": "Ground", "attentionTo": "Eugene Krabs" },
  "items": [{
    "sku": "CDS-90X2-PIABKIT",
    "quantity": 1,
    "purchaseOption": "Equipment Purchase",
    "addOns": [
      { "sku": "PIAB-DeviceMonitoringandManagement", "quantity": 1 },
      { "sku": "RESELLER-PIABALLOWANCE-1GB", "quantity": 1, "purchaseOption": "Month to Month" },
      { "sku": "PIAB-SiteSurvey/GoLive", "quantity": 1 },
      { "sku": "PIABOneVisistInstall-OneTime", "quantity": 1 },
      {
        "sku": "SpecialtyBusinessLineAndLicense",
        "quantity": 3,
        "serviceLines": [
          { "portExisting": false, "lineType": "Burglar Alarm" },
          { "portExisting": false, "lineType": "Elevator" },
          { "portExisting": false, "lineType": "Elevator" }
        ],
        "addOns": [{ "sku": "BRUIN-LICENSED-SOFTWARE", "quantity": 3 }]
      },
      { "sku": "BoardBlack", "quantity": 1, "purchaseOption": "Equipment Purchase" }
    ]
  }],
  "notes": "Ordering a new PIAB with install and backboard to Headquarters",
  "billingAccount": "350674"
}
```

`lineType` is a **name** (`Voice`, `Fire Alarm`, `Burglar Alarm`, `Modem`,
`Elevator`, `Fax`, `Elevator Modem`), not a `SIPLine1` code. Porting: set
`portExisting: true` and `phoneNumber` on that line.

### Refresh example (`changeDevice`)

Current WTN goes in `serviceLines[0].phoneNumber`, **not** `phoneNumbers`.

```json
{
  "clientId": 1234,
  "scenario": "changeDevice",
  "region": "US",
  "billingAccount": "449687",
  "serviceAddressId": 1926955,
  "customerReference": "PO-CHANGEDEVICE",
  "deliveryPreferences": { "shippingMethod": "Ground" },
  "orderContact": {
    "firstName": "Jordan", "lastName": "Reyes",
    "email": "jreyes@example.com", "phoneNumber": "5550000000"
  },
  "items": [{
    "sku": "ME259LL/A",
    "purchaseOption": "Equipment Purchase",
    "serviceLines": [{ "phoneNumber": "7755445345" }],
    "existingDeviceDecision": "keep",
    "addOns": [{ "sku": "MMTN2AM/A" }]
  }]
}
```

### `serviceLines` (replaces `phoneNumbers` / item-level `portExisting`)

| Field | Applies to | Notes |
| --- | --- | --- |
| `phoneNumber` | SBL, change scenarios | SBL DID or current WTN. Required when `portExisting` is true, and required for all change scenarios. |
| `portExisting` | SBL | Per-line port-in flag. Ignored on change-scenario lines. |
| `lineType` | SBL / PIAB | Name: `Voice`, `Fire Alarm`, `Burglar Alarm`, `Modem`, `Elevator`, `Fax`, `Elevator Modem`. Defaults to `Voice`. Ignored on change scenarios. |

Rules: SBL new-order entries must equal `quantity`. Change scenarios: at least
one entry, `serviceLines[0].phoneNumber` is the current WTN, one WTN per
primary (no multi-line fan-out). Smartphone/accessory new orders may omit it.

### Other item fields

| Field | Notes |
| --- | --- |
| `quantity` | Defaults to `1`. SBL `quantity > 1` expands to one line per unit. |
| `carrier` | Optional network selection. On `changeService`, a different value is a network change (validated against the device). |
| `purchaseOption` | `"Equipment Purchase"`, a recurring `"<Term> <Ownership Type>"` (e.g. `"24 Months FINANCE"`, `"12 Months RENT"`, `"36 Months MDaaS"`), or an entitled plan name for Service add-ons. Case-insensitive. |
| `inventoryType` | Goods only: `Depot`, `Customer`, or `General`. Not applied to Service/plan add-ons. Omitted → platform picks from configured order / stock, default `General`. |
| `existingDeviceDecision` | Device swaps: `keep`, `trade`, or `depot` (case-insensitive). |
| `removeAddOns` | `changeDeviceAndService` / `changeService`: add-on SKUs to drop. Single-select conflicts are removed automatically. |
| `userInfo` | Smartphone: `firstName` + `lastName` of the assigned user. Not used on `changeService`. `email` / `employeeId` / `phoneNumber` are accepted but reserved (no effect). |
| `addOns` | Recursive, same shape as an item. |

**Purchase-option resolution (Goods):** a valid named option is used; omitted
with exactly one recurring option → that option; omitted/invalid with several
→ rejected (never silently replaced). `"Equipment Purchase"` is always accepted
for Goods.

**Add-ons:** Service plans match by name among entitled options (omitted → first
entitled). Goods add-ons follow the Goods rules (omitted → Equipment Purchase).

### Address, contacts, billing, shipping

- **Service address:** `serviceAddressId` **or** `serviceAddress{street,suite,city,state,zip,country}`. ID wins if both are sent. Neither → rejected. `country` is inferred from state/ZIP (accepted, unused).
- **Site contact:** omit → order contact is reused (single-contact ticket). Supply to create a distinct site contact. Email domain must be registered to the account; an unregistered domain is **silently skipped** (no error).
- **`ticketSubscribers[]`:** emails looked up and attached; unknown emails are skipped, not created, no error.
- **Billing:** omit `billingAccount` → resolved from the service address when one is supplied, otherwise no account is attached. Valid account + omitted `subAccountNumber` → a new sub-account is created. SBL items **require** `billingAccount`.
- **`quickOrderId`:** if supplied, the QuickOrder template is used and `items` are ignored.
- **Shipping method** (`deliveryPreferences.shippingMethod`), case-insensitive:

  | Name | Business days |
  | --- | --- |
  | `Next Day Air` | 1 |
  | `2nd Day Air` | 2 |
  | `Ground` | 5 |

  Omitted/empty defaults to the first configured method (currently **2nd Day
  Air**). An unrecognized value is **accepted as sent** with no shipping method
  resolved and **no error** — typos silently produce a ticket with no method.
  Also accepted: `requestedDate`, `specialInstructions`, `attentionTo`,
  `signatureRequired`, `insuranceRequired`.

Accepted with no effect: `notes` (recorded as free text, not parsed),
`metadata`, `orderContact.employeeId`, `deliveryPreferences.businessAttentionTo`.

### Response

```json
{
  "ticketId": 12345,
  "pricing": { "oneTimeTotal": 250.00, "monthlyTotal": 139.90 },
  "items": [{
    "sku": "CDS-9090-PIABKIT",
    "bruinSkuId": 12345,
    "productName": "POTS in a Box Kit",
    "monthlyPrice": 0.00,
    "oneTimePrice": 0.00,
    "addOns": [{
      "sku": "SPECIALTYBUSINESSLINEANDLICENSE",
      "bruinSkuId": 97220,
      "productName": "Specialty Line User w/Bruin License",
      "monthlyPrice": 69.95,
      "oneTimePrice": 0.00
    }]
  }]
}
```

Change scenarios may also include per-item `removedAddOns[]`
(`sku`, `productName`, `reason`) for services dropped explicitly or by
single-select conflict.

Validation failures return **HTTP 400** with a message (missing items/SKU,
unknown billing/sub-account, SBL `serviceLines` count ≠ `quantity`, missing
WTN/`existingDeviceDecision`, purchase-option / inventory-type / scenario
errors). Unrecognized shipping method is **not** a 400.

---

## `GET /api/ticket/topics`: list valid categories

Returns the authoritative set of `category` codes available for the account. Use
this to validate/enumerate categories rather than hardcoding.

---

## Testing safely (no dev environment)

There is **no client-facing dev server**; you test in production, so guard
against real fulfillment:

- Coordinate with your account care rep / CSE to review, cancel, or simulate
  test orders.
- On `PlaceOrder`, put the warning in top-level `notes` (not `MTK`):
  `"TESTING SOFTWARE, DO NOT TAKE ANY ACTION ON THIS TICKET UNLESS INSTRUCTED BY AN ACCOUNT CARE REPRESENTATIVE."`
  On `POST /api/Ticket`, use the `MTK` note for the same warning.
- Omitting `billingAccount` does **not** dry-run. A PIAB-with-lines order is
  **rejected** without it. For other products the API may resolve billing from
  the service address or attach no account — that is still a real ticket.

Skipping these can cause real device shipments, provisioned lines, technician
dispatches, and charges.

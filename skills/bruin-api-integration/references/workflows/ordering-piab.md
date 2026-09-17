# Workflow: Ordering PIAB (POTS-in-a-Box)

Order a PIAB kit with specialty business lines, dispatch(es), and a backboard
via `POST /api/Ticket/PlaceOrder` (`scenario: "deviceAndService"`). Endpoint
rules: [../ordering.md](../ordering.md). Related: [../site.md](../site.md),
[../user.md](../user.md), [../webhooks.md](../webhooks.md).

PIAB **repairs and changes** still use `POST /api/Ticket` — see
[../tickets/piab.md](../tickets/piab.md). Do **not** use `POST /api/Ticket/NewOrder`
for a new PIAB; the published how-to uses PlaceOrder only.

Recommended webhook subscriptions: **NewNoteAdded** + **Closed**.

---

## Steps

1. **Create the site** (new location) — `POST /api/Site`. Store `siteID` and
   `addressID`. `addressID` is the order's `serviceAddressId`. A bad address
   returns `400`; override with `CreateIfAddressUnverified`. Confirm with
   `GET /api/Site` if needed.
2. **Contacts** — PlaceOrder takes `orderContact` (required; often a PM) and
   optional `siteContact` (on-premises, valid mobile). Omit `siteContact` and
   the order contact is reused. PlaceOrder **creates** missing users by email.
   Site-contact email domain must be registered to the account; an unregistered
   domain is **silently skipped** (no error, no site contact). Pre-create with
   `POST /api/User` only if you need portal/notification setup first.
3. **Skip the site call (optional)** — send `serviceAddress{street,city,state,zip}`
   instead of `serviceAddressId`. The API resolves or creates the address and a
   SiteLabel. Prefer a stored `serviceAddressId` when you will reuse it. If both
   are sent, the ID wins; if neither, the order is rejected.
4. **Gather SKUs** — account-specific, from account care / CSE. Example strings
   (many cannot be submitted as-is):
   - `CDS-90X2-PIABKIT` (primary kit)
   - `PIAB-DeviceMonitoringandManagement`, `RESELLER-PIABALLOWANCE-1GB`
   - `PIAB-SiteSurvey/GoLive`, `PIABOneVisistInstall-OneTime` (dispatches)
   - `SpecialtyBusinessLineAndLicense` (+ nested `BRUIN-LICENSED-SOFTWARE`)
   - `BoardBlack` (backboard kit)
   Purchasing options are **`purchaseOption` values, not SKUs** (e.g.
   `"Equipment Purchase"`, `"36 Months FINANCE"`). Do **not** send `NOCONTRACT`,
   `36MONTH`, or a nested `"Purchase"` SKU.
5. **Place the order** — `POST /api/Ticket/PlaceOrder` with
   `scenario: "deviceAndService"`. One primary kit; everything else is `addOns`.
   Store `ticketId`.
6. **Track** — `NewNoteAdded` for `ShippingTracking`, `ShipUpdate`,
   `Disp_DateConf` / `Disp_ApptConf`; then `Closed` = install complete / lines
   working.
7. **Get line data** — on `Closed`, `GET /api/Ticket/{ticketId}/details` and read
   notes matching `FXS<n>Info`. Parse the `|`-delimited `noteValue` for `DID:`
   and `Line Type:`.

---

## PlaceOrder body (new PIAB)

Do **not** send a change scenario (`changeDevice`, `changeDeviceAndService`,
`changeService`) for a new install.

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
  "ticketSubscribers": [{ "email": "dshim@mettel.net" }],
  "shippingAddress": {
    "street": "170 S Main St", "city": "Salt Lake City", "state": "UT", "zip": "84101"
  },
  "deliveryPreferences": {
    "requestedDate": "2026-05-30",
    "shippingMethod": "Ground",
    "specialInstructions": "Leave at front desk",
    "attentionTo": "Eugene Krabs"
  },
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

- `billingAccount` is **required** when the order includes Specialty Business
  Lines. Omit `subAccountNumber` to create a new sub-account under that BAN.
  Omitting `billingAccount` is **not** a dry run: a PIAB-with-lines call is
  **rejected** without it.
- Top-level `notes` is free text on the ticket. It is **not** parsed as note
  types (`MTK`, `BAN`, `Hierarchy`, …). Billing, site, and shipping come from
  the structured fields.
- `addOns` is recursive (a line SKU can have its own add-ons).
- `shippingMethod`: `Ground` (5 days), `2nd Day Air` (2), `Next Day Air` (1).
  Omitted → first configured method (currently **2nd Day Air**). A typo is
  accepted with **no** shipping method and **no** error.

### `serviceLines` (line identity)

This is the **only** place for phone number, port-in, and PIAB line type. Do
not send item-level `phoneNumbers` / `portExisting`, and do not send per-line
ticket notes (`SIPLine1`, `PhoneNumberType`, `DirectoryListing`, …) — PlaceOrder
writes those from this array.

| Field | When | Notes |
| --- | --- | --- |
| `lineType` | Recommended on every PIAB line | Name, not a code. Omitted → `Voice`. |
| `portExisting` | Per line | `true` to port that DID; `false`/omit for a new number. |
| `phoneNumber` | Required when `portExisting` is `true` | The DID to port. May be omitted on new-number lines. |

`serviceLines.length` **must equal** `quantity` (HTTP 400 on mismatch). Mix new
and ported lines on the same add-on. There is **no** separate `IP-Port` SKU.

| Line type | `lineType` value |
| --- | --- |
| Voice | `Voice` |
| Fire Alarm | `Fire Alarm` |
| Burglar Alarm | `Burglar Alarm` |
| Modem | `Modem` |
| Elevator | `Elevator` |
| Fax | `Fax` |
| Elevator Modem | `Elevator Modem` |

PlaceOrder maps these to ticket notes: `PhoneNumberType`, `DIDnums`, `SIPLine1`,
`DirectoryListing` (`"False"`), `SeriesCompLine` (`"No"`).

---

## After submit

Response: `ticketId` plus `pricing.{oneTimeTotal,monthlyTotal}` and priced
`items[]` / `addOns[]`. Store `ticketId`.

Watch `NewNoteAdded`: `ShippingTracking` (FedEx tracking), `ShipUpdate`
(delivered), `Disp_DateConf` / `Disp_ApptConf` (tech date/time). `Closed` =
install complete. Resellers often start billing at Closed; MetTel line billing
starts 21 days after order placement.

---

## Testing (production only)

No client dev server. Send test orders to account care / CSE to review, cancel,
or simulate. Put the warning in top-level `notes`:

```json
"notes": "TESTING SOFTWARE, DO NOT TAKE ANY ACTION ON THIS TICKET UNLESS INSTRUCTED BY AN ACCOUNT CARE REPRESENTATIVE."
```

Skipping this can trigger real shipments, provisioned lines, dispatches, and
charges. Omitting `billingAccount` will not dry-run a PIAB order.

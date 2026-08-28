# Workflow: Cell Phone Lifecycle

End-to-end playbook for a wireless device's life: **new order → refresh →
offboard**. Uses [../ordering.md](../ordering.md) (`PlaceOrder`),
[../user.md](../user.md), [../site.md](../site.md), [../inventory.md](../inventory.md),
and [../webhooks.md](../webhooks.md).

Recommended webhook subscriptions for all three: **NewNoteAdded** + **Closed**.

---

## A. New cell phone (`scenario: deviceAndService`)

1. **Create the user** (if not already in Bruin) — `POST /api/User`. The email
   domain must be authorized (CSE) and unique. Store the returned `dirId`.
   The device is assigned to this user at order time.
2. **Get the billing address** — `serviceAddressId` is the `addressId` the line
   bills to (one of the addresses on the bill, not necessarily the install
   address). Get it from `GET /api/Site` (or the `POST /api/Site` response).
   Commonly hardcoded to HQ.
3. **Get SKUs** — device + add-on (case/charger) + data-plan SKUs come from
   account care / CSE; they're account-specific.
4. **Place the order** — `POST /api/Ticket/PlaceOrder` with
   `scenario: "deviceAndService"`, `orderContact`, `serviceAddressId`,
   `items[]` (device `sku`, `carrier`, `purchaseOption`, `userInfo`, `addOns`
   including the plan), and `billingAccount`. Store the returned `ticketId`.
5. **Track via webhooks** — watch `NewNoteAdded` for:
   - `ShippingTracking` — FedEx tracking number (value is `[tracking|url]`).
   - `ShipUpdate` — delivery status (`… Shipment Status : Delivered`).
   - `ADN` — free-text staff notes to surface in your UI.
   Then `Closed` = order complete / phone activated.
6. **Gather line data** — on `Closed`, call
   `GET /api/Ticket/{ticketId}/details` and read `ticketDetails[]` for
   `detailType: "WTN"` (the assigned phone number).

---

## B. Refresh an existing phone (`changeDevice` / `changeDeviceAndService`)

Same as a new order, plus old-device handling.

1. **Look up the user** — `GET /api/User?UserName=<email>`.
2. **Look up the device** — `GET /api/Inventory?AssignedUserName=<email>`;
   capture `serviceNumber` (the old phone number) and `inventoryID`.
3. **Choose the scenario:**
   - `changeDevice` — new device, **same** carrier.
   - `changeDeviceAndService` — new device **and** new carrier/plan (supply the
     new `carrier` and a plan add-on SKU).
4. **Old-device decision** — in the `items[]` entry set:
   - `phoneNumbers: ["<old number>"]` (index `[0]`).
   - `existingDeviceDecision`: `keep` (user keeps it; lease charges may apply),
     `trade`, or `depot` (both `trade`/`depot` generate a return label; `depot`
     returns it to the client's depot stock).
5. **Place order** — `POST /api/Ticket/PlaceOrder`. Store `ticketId`.
6. **Track the return** — beyond shipping notes, watch `NewNoteAdded` for:
   - `ReturnLabel` — return shipping label link/QR for the old device.
   - `ReturnOutcome` — `Returned` (good), `Rejected` (poor condition), or
     `Not returned`.

---

## C. Offboard a device (`POST /api/Ticket`, category `WWB`)

Sunsets a device/line (employee offboarding). This is a standard ticket, not a
PlaceOrder.

1. **Gather info** — `GET /api/Inventory?AssignedUserName=<email>` →
   `serviceNumber` + `inventoryID`; then
   `GET /api/Inventory/Attribute?InventoryId=<id>` → `IMEI` (preferred) or serial.
2. **Build the ticket** — `POST /api/Ticket`, `category: "WWB"`, `services[]`
   with the `serviceNumber`, a **Site** contact, and these notes:

   | noteType | Value / meaning |
   | --- | --- |
   | `MTK` | Free-text (e.g. "Employee Offboarding"). |
   | `DDD` | Desired due date `MM/DD/YYYY` (recommend ~1 month out to avoid device/ETF charges). |
   | `RefTicketNumber` | Your system's ticket ID. |
   | `DeviceDecisionId` | `1` Keep It · `2` Return · `3` Trade-In · `4` Depot. |
   | `ServiceDecisionId` | `1` Disconnect · `2` Keep Active · `3` Suspend · `4` Suspend then Disconnect. |
   | `MDMPlatform` | `AWM`, `Checkpoint Harmony Mobile`, `InTune`, `Ivanti`, `Other`, `SOTI`, `Scalefusion`. |
   | `MDMRequestType` | The action (per platform; e.g. Ivanti `Wipe Device`, `Unenroll Device`). |
   | `MDMDeviceIdentifier` | `IMEI` or `SerialNumber` — required for device-level MDM actions. |
   | `IMEI` / `SerialNumber` | The identifier value matching the line above. |

   Return-kit notes (only if `ReturnKIT` = `true`):
   | noteType | Meaning |
   | --- | --- |
   | `ReturnKIT` | `true` / `false`. If `false`, omit the rest. |
   | `ShippingAddress` | Where to send the return kit. |
   | `AttentionTo` | Recipient name. |
   | `ShippingMethod` | e.g. `XXX03` (ground). |

3. **Submit & track** — store `ticketIds[0]`; watch `NewNoteAdded` →
   `ReturnOutcome` (`Returned` / `Rejected` / `Not returned`) and `Closed`.

> MDM request types marked device-specific in the docs require the
> `MDMDeviceIdentifier` + identifier notes. When in doubt, include the IMEI.

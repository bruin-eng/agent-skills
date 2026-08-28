# Workflow: Ordering PIAB (POTS-in-a-Box)

Order a PIAB device with specialty business lines, dispatch(es), and a backboard
kit. Uses `POST /api/Ticket/NewOrder` — see [../ordering.md](../ordering.md) for
the endpoint's structural rules. Related: [../site.md](../site.md),
[../user.md](../user.md), [../webhooks.md](../webhooks.md).

Recommended webhook subscriptions: **NewNoteAdded** + **Closed**.

---

## Steps

1. **Create the site** (new location) — `POST /api/Site`. Store `siteID` and
   `addressID`. A bad address returns `400`; override with
   `CreateIfAddressUnverified`.
2. **Create/confirm contacts** — Site + Ticket contacts must be Bruin users
   (`POST /api/User` if needed). Site contact = someone on-premises (give a
   valid mobile); Ticket contact = usually a PM.
3. **Gather SKUs** — account-specific, from account care / CSE. Typical set:
   - `CDS-90X2-PIABKIT` (device kit)
   - `PIAB-DeviceMonitoringandManagement`, `RESELLER-PIABALLOWANCE-1GB`
   - purchase option `NOCONTRACT` or `36MONTH`
   - `PIAB-SiteSurvey/GoLive`, `PIABOneVisistInstall-OneTime` (dispatches)
   - `SpecialtyBusinessLineAndLicense` (+ `36MONTH`, `BRUIN-LICENSED-SOFTWARE`)
   - `BoardBlack` (backboard kit)
   - `IP-Port` (only if porting numbers in)
4. **Build the order** — `POST /api/Ticket/NewOrder`, `orderType: "PIAB_NewOrder"`.
   Assemble the recursive `items[]` tree and the level-tagged notes (below).
5. **Submit** — ~20s to process. Store the returned `ticketId`.
6. **Track** — `NewNoteAdded` for `ShippingTracking`, `ShipUpdate`,
   `Disp_DateConf` / `Disp_ApptConf` (technician dispatch date/time); then
   `Closed` = install complete / lines working.
7. **Get line data** — on `Closed`, `GET /api/Ticket/{ticketId}/details` and read
   notes matching `FXS<n>Info` (one per line). Parse the `|`-delimited
   `noteValue` for `DID:` (phone number) and `Line Type:`.

---

## The `items[]` tree

- Every item: `{ sku, quantity, subItems, itemNotes }`.
- `subItems` is **recursive** (device → lines/licenses → contract term). Keep
  `quantity` consistent down a parent/child chain (3 lines → 3 licenses → 3
  terms).
- Purchasable services (survey, install, board, lines) usually carry a
  `{ "sku": "Purchase", ... }` sub-item.

### Notes by level (`noteLevel`)

- **`Ticket`** (top-level `notes[]`):
  `RefTicketNumber` (your ref), `MTK` (staff-visible summary — important),
  `BAN` (`noteText` = "BANnumber:MetTel", `noteValue` = BAN ID; from CSE),
  `SubAccount` (`-1` to create a new one), `Hierarchy`, `SiteLabel`,
  `OrderAttributes` (`"{}"`), `OpportunityID`/`ProjectTicket`/`SrvcDlvr*` (leave
  as example/null unless told otherwise).
- **`Subcategory`** (per-SKU `itemNotes[]`, required on kit/survey/install/lines/board):
  `ShippingAddress` (`noteValue` = addressID), `DDD` (`MM/DD/YYYY`, future;
  recommend ~2 weeks out), `AttentionTo`.
- **`Item`** (per-line, keyed by `noteForItemNo` 1..N on
  `SpecialtyBusinessLineAndLicense`):
  `DirectoryListing` (`True`/`False` — `False` for alarm/elevator lines),
  `PhoneNumberType` (`New Phone Number` or `Port Existing Phone Number`),
  `APINPANXX` (desired first 6 digits, or null to auto-pick from site),
  `SeriesCompLine` (hunt group), `SIPLine1` (line type — required for every line).

### `SIPLine1` line-type codes

| Line type | noteValue |
| --- | --- |
| Voice | `231` |
| Fire Alarm | `245` |
| Burglar Alarm | `246` |
| Modem | `247` |
| Elevator | `248` |
| Fax | `250` |
| Elevator Modem | `251` |

---

## Porting numbers in

If any line ports an existing number:

1. Add an **`IP-Port`** item with `Subcategory` itemNotes: `btnofports` (billing
   TN of the ported lines), `portlist` (`"2"` / "I will type the DIDs to be
   ported"), `didtextarea` (comma-separated DIDs), `dirlistingtype` (`"4"` /
   "No Change Needed"), `cnamdetails` (caller name), plus `ShippingAddress`,
   `DDD`, `AttentionTo`.
2. On the ported line's `SpecialtyBusinessLineAndLicense` `Item` notes, set
   `PhoneNumberType` = `Port Existing Phone Number` and add `btnofports` and
   `DIDnums` (the specific TN for that line, keyed by `noteForItemNo`).

---

## Testing (production only)

No client dev server. Before real submission: coordinate with account care/CSE,
set `MTK` to the "TESTING SOFTWARE, DO NOT TAKE ANY ACTION…" warning, and expect
to have the order reviewed/cancelled/simulated. Skipping this can trigger real
shipments, provisioned lines, dispatches, and charges. Lines don't begin billing
until 21 days after order placement.

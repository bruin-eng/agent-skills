# Inventory (read-only)

Read-only visibility into the services, devices, and lines MetTel manages for a
client. Every billable item — a phone, a circuit, an SD-WAN device, a Starlink
terminal — is an inventory record with a unique `inventoryID`.

**Use this to find the `inventoryID` / `ServiceNumber` you need before opening a
ticket, to sync into a CMDB/asset system, or to pull carrier-side identifiers
(BTN, PIC/LPIC, IMEI).**

**Required scope:** `FunctionPermissionInventoryGet`

## Status values (shared)

Every record has a `status`:

- `A` — Active (live and billable)
- `S` — Suspended (temporarily paused, record retained)
- `D` — Disconnected (terminated, record retained for history)

---

## `GET /api/Inventory` — list / filter inventory

Returns `{ "documents": [ … ] }`. All query params are optional filters; combine
them to narrow the set. Most integrations scope by `ClientId`.

> **No pagination.** This endpoint returns the **entire** matching result set in
> one response — there are no `page`/`limit`/`offset` parameters and no cursor.
> Use the filters to keep responses manageable, and expect a single (possibly
> large) `documents` array rather than pages to iterate.

> **Suggested approach — dump first, then decide.** Because a single call returns
> everything, a good pattern is to make one broad request (scoped by `ClientId`),
> **look at the raw `documents` dump**, and then decide what to do with it rather
> than guessing filters up front:
> - See which `productCategory` / `productType` / `status` values actually exist
>   for this client, then re-query (or filter client-side) for the subset you want.
> - Locate the record you need and grab its `inventoryID` / `serviceNumber` before
>   opening a ticket or calling the Attribute endpoint.
> - Decide the downstream action from what you see — sync to a CMDB, reconcile
>   against your own records, or narrow to a site/user.
>
> In short: pull the data, inspect the shape and values, *then* choose filtering
> or the next call — don't over-constrain a query before you've seen what's there.

### Query parameters

| Param | Type | Notes |
| --- | --- | --- |
| `ClientId` | integer | Bruin Client ID; scopes to one client. |
| `Status` | string | `A` / `D` / `S`. |
| `AssignedUserName` | string | First or last name contains this value. |
| `AssignedUserId` | uuid | Exact assigned-user ID. |
| `SiteID` | integer | Inventory at a specific site. |
| `ServiceNumber` | string | Phone number or circuit ID contains this value. |
| `ProductCategory` | string | e.g. `"Wireless"`. |
| `ProductSubCategory` | string | e.g. `"4G LTE"`. |
| `InstallDateFrom` / `InstallDateTo` | `yyyy-MM-dd` | Install-date window. |
| `DisconnectDateFrom` / `DisconnectDateTo` | `yyyy-MM-dd` | Disconnect-date window. |
| `UpdatedDateFrom` / `UpdatedDateTo` | `yyyy-MM-dd` | Updated-date window. |

### Example

```bash
curl -X GET "https://api.bruin.com/api/Inventory?ClientId=53206&Status=A&ProductCategory=Wireless" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

### Response shape (per `documents[]` item)

Key fields (not exhaustive):

| Field | Meaning |
| --- | --- |
| `inventoryID` | Unique Bruin inventory ID (string). |
| `serviceNumber` | Phone number / circuit ID — the `ServiceNumber` for a ticket. |
| `clientID`, `clientName` | Owning client. |
| `status` | `A` / `S` / `D`. |
| `productCategory`, `productType`, `productName` | e.g. `Wireless` / `Smart Phone` / `Apple iPhone 15`. |
| `siteId`, `siteLabel`, `address{}` | Where it's installed. |
| `assignee`, `assigneeEmail`, `assigneeUserId` | Assigned user (`assigneeUserId` is the Bruin user GUID). |
| `vendor`, `accountNumber`, `subAccountNumber` | Upstream vendor + account. |
| `installDate`, `disconnectDate`, `updatedDate` | Lifecycle dates. |
| `items[]` | `{ itemName, primaryIndicator }` — item-level pointers (full values via the Attribute endpoint). |
| `longitude`, `latitude` | Site coordinates. |

```json
{
  "documents": [
    {
      "clientID": 53206,
      "inventoryID": "11868690",
      "serviceNumber": "3059473030",
      "status": "A",
      "productCategory": "Wireless",
      "productType": "Smart Phone",
      "productName": "Apple iPhone 15",
      "siteId": 58,
      "siteLabel": "Downtown Office",
      "assignee": "Jane Doe",
      "assigneeUserId": "66F8AD88-2518-E611-80FB-0050568529C0",
      "items": [ { "itemName": "IMEI", "primaryIndicator": "Y" } ]
    }
  ]
}
```

---

## `GET /api/Inventory/Attribute` — item-level attributes

Returns the key/value attributes (BTN, PIC/LPIC, IMEI, etc.) for **one**
inventory record.

Identify the record **either**:
- by `InventoryId` alone, **or**
- by all three of `ClientId` + `Status` + `ServiceNumber` (used to resolve a
  single record when you don't have the ID).

### Example

```bash
curl -X GET "https://api.bruin.com/api/Inventory/Attribute?InventoryId=11868690" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

### Response

```json
{
  "inventoryId": "11868690",
  "serviceNumber": "3059473030",
  "attributes": [
    { "key": "BTN",  "value": "9724424432" },
    { "key": "LPIC", "value": "0555" },
    { "key": "PIC",  "value": "0555" }
  ]
}
```

Use these carrier-side identifiers (e.g. BTN, PIC/LPIC) when filing ports or
change requests.

---

## Errors

Standard codes: `400` malformed params, `401` missing/expired token, `403`
missing `FunctionPermissionInventoryGet`, `500` server error (returns the
standard error schema with a `traceId`). For the Attribute endpoint, a `400`
usually means you supplied neither `InventoryId` nor a complete
`ClientId`+`Status`+`ServiceNumber` triple.

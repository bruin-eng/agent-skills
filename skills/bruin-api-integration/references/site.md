# Sites (locations)

A **site** is the physical location a service is installed at — an office,
retail store, warehouse. Every site has a unique `siteID` and belongs to exactly
one client. Inventory and tickets reference sites by `siteID`.

**Use this to find a `siteID` before provisioning/ticketing, to look up a site
by address, or to sync locations from a facilities/HR system.**

## Site identifiers

- `siteID` — Bruin's internal numeric ID. The one Inventory and Tickets use.
- `siteIdentifier` — the client's own external ID (e.g. `"NYC-DOWNTOWN-001"`).
- `siteLabel` / `siteName` — human-readable name in the portal.

---

## `GET /api/Site` — list / filter sites

Returns `{ "documents": [ … ] }`.

**Required scope:** `FunctionPermissionSiteGet`

> **No pagination.** Returns the **entire** matching result set in one response —
> no `page`/`limit`/`offset` parameters, no cursor. Filter to narrow it; expect a
> single `documents` array, not pages.

> **Suggested approach — dump first, then decide.** Since one call returns every
> site, make a broad request (scoped by `ClientID`), **look at the raw `documents`
> dump**, and then decide:
> - Find the location you need and grab its `siteID` / `addressID` for ordering,
>   provisioning, or attaching a ticket.
> - Match Bruin sites to your own system via `siteIdentifier` before syncing.
> - Confirm whether a location already exists before calling `POST /api/Site` to
>   create a new one (avoids duplicates).
>
> Pull the list, inspect what's actually there, *then* pick the record or the
> next action.

### Query parameters (all optional filters)

| Param | Type | Notes |
| --- | --- | --- |
| `ClientID` | integer | Scope to one client's sites. |
| `SiteID` | integer | A specific site label ID. |
| `Address` | string | Address contains this substring. |
| `AddressID` | integer | A specific Bruin address ID. |
| `SiteName` | string | Filter by site name. |
| `SiteIdentifier` | string | Filter by the client-supplied identifier. |

```bash
curl -X GET "https://api.bruin.com/api/Site?ClientID=9994&Address=1%20Beacon" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

### Response shape (per `documents[]` item)

Key fields: `siteID`, `siteLabel` / `siteName`, `siteIdentifier`, `clientID`,
`address{ address1, address2, city, state, zip, country }`, `longitude`,
`latitude`, `businessHours`, `timeZone`, `primaryContactName` /
`primaryContactPhone` / `primaryContactEmail`, `structureType`,
`businessPurpose`, `hid`, `hierarchy`, `subAccounts`, `siteAddDate`.

```json
{
  "documents": [
    {
      "clientID": 9994,
      "siteID": 58,
      "siteLabel": "Downtown Office",
      "siteIdentifier": "NYC-DOWNTOWN-001",
      "address": { "address1": "1 Beacon St", "address2": "Suite 200", "city": "Boston", "state": "MA", "zip": "02108", "country": "USA" },
      "timeZone": "America/New_York",
      "hierarchy": "|Region|East|NYC|"
    }
  ]
}
```

---

## `POST /api/Site` — create a site

**Required scope:** `FunctionPermissionSiteUpdate`

### Body fields

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `ClientID` | integer | **Yes** | Bruin Client ID. |
| `SiteName` | string | **Yes** | Display name. |
| `SiteIdentifier` | string | **Yes** | Client-supplied unique ID. |
| `Address` | object | **Yes** | See below. |
| `Address.AddressLine` | string | **Yes** | Street address. |
| `Address.City` | string | **Yes** | |
| `Address.State` | string | **Yes** | |
| `Address.Zip` | string | **Yes** | |
| `Address.Country` | string | **Yes** | ISO 3166-1 alpha-3 (e.g. `"USA"`). |
| `Hierarchy` | string | **Yes** | Pipe-delimited path, e.g. `"|Region|East|NYC|"`. |
| `BusinessHours` | string | No | e.g. `"MONDAY|ALL DAY,TUESDAY|ALL DAY"`. |
| `PrimaryContactEmail` | string | No | Site primary contact. |
| `CreateIfAddressUnverified` | string | No | Force-create an unverified address. Default `"true"`. |

```bash
curl -X POST "https://api.bruin.com/api/Site" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "ClientID": 123,
    "SiteName": "Downtown Office",
    "SiteIdentifier": "NYC-DOWNTOWN-001",
    "Address": { "AddressLine": "123 Main Street", "City": "New York", "State": "NY", "Zip": "10001", "Country": "USA" },
    "Hierarchy": "|Region|East|NYC|"
  }'
```

### Response

```json
{ "SiteID": 456, "AddressID": 789, "Success": true, "Message": "Site created successfully." }
```

---

## Errors

`400` invalid/missing fields, `401` missing/expired token, `403` missing the
required scope (`FunctionPermissionSiteGet` for read, `FunctionPermissionSiteUpdate`
for create), `500` server error (standard error schema + `traceId`).

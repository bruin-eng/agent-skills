# Querying Tickets

Three read endpoints give you visibility into tickets that already exist:
list/filter, pull one ticket's detail lines + notes, and check PON
(Purchase/Provisioning Order Number) status. Use these to check on work that's
already been opened — for creating a ticket, see
[ticket-model.md](ticket-model.md) and the per-product references instead.

**Required scope for all three:** `FunctionPermissionTicketGet`

| Task | Endpoint |
| --- | --- |
| Check a ticket's status before following up with a customer | `GET /api/Ticket` |
| See the detail lines and notes on a ticket (what's been worked, current tasks, history) | `GET /api/Ticket/{ticketId}/details` |
| Get a PON or its status to track a provisioning request | `GET /api/Ticket/{ticketId}/pon` |
| Sync ticket data into an external system for reporting/dashboards | `GET /api/Ticket` (poll) or [webhooks.md](webhooks.md) (push) |

## `GET /api/Ticket`: list / filter tickets

**This is the one paginated list endpoint** in the API (unlike Inventory/Site/
User, which return everything in one shot — see the skill overview). Page with
`Start` (default `0`) and `Rows` (default/max `10000`).

```bash
curl -X GET "https://api.bruin.com/api/Ticket?ClientId=53206&TicketStatus=Resolved" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

### Query parameters (all optional filters)

| Param | Notes |
| --- | --- |
| `ClientId` | Bruin Client ID. |
| `ProductCategory` | e.g. `"Wireless"`. |
| `TicketId` | Look up a single ticket. |
| `SiteID` | Tickets at a specific site. |
| `Address` | Substring match. |
| `InventoryId` | Tickets tied to one inventory record. |
| `ServiceNumber` | Requires `ClientId`; ignored if `InventoryId` is set. |
| `TicketStatus` | `New` / `InProgress` / `InReview` / `Resolved` / `Closed` / `Draft`. |
| `TicketTopic` | Full topic name or topic key. |
| `StartDate` / `EndDate` | ISO 8601 UTC creation-date window (`yyyy-MM-ddTHH:mm:ssZ`). |
| `OpportunityID` | Salesforce Opportunity ID. |
| `ReferenceTicketNumber` | Filter by reference ticket number. |
| `Start` / `Rows` | Paging: start offset (default `0`) / page size (default & max `10000`). |

### Response

```json
{
  "responses": [
    {
      "clientID": 53206,
      "clientName": "Sam & Su's Retail Shop 5",
      "ticketID": 3521039,
      "category": "017",
      "topic": "Add Cloud PBX User License",
      "referenceTicketNumber": null,
      "ticketStatus": "Resolved",
      "address": { "addressID": 169159, "address": "69 Blanchard St", "city": "Newark", "state": "NJ", "zip": "07105-4701", "country": "USA" },
      "createDate": "4/23/2019 7:59:50 PM",
      "createdBy": "Amulya Bidar Nataraj",
      "creationNote": "Test",
      "resolveDate": "4/23/2019 8:00:35 PM",
      "mostRecentNote": " ",
      "nextScheduledDate": "4/23/2019 4:00:00 AM",
      "flags": "",
      "severity": "4",
      "specialty": ["DS0 Repair"],
      "specialties": "DS0 Repair",
      "products": [{ "productName": "Apple iPhone 15", "sku": "Test03271" }]
    }
  ],
  "row": 10000,
  "start": 0
}
```

`row`/`start` echo the page size/offset you requested (not a total count) —
keep paging with `Start += Rows` until a page comes back with fewer than
`Rows` entries.

### Ticket status values

| Status | Meaning |
| --- | --- |
| `New` | Just created. |
| `InProgress` | Actively being worked. |
| `InReview` | Under review. |
| `Resolved` | Issue resolved. |
| `Closed` | Closed out. |
| `Draft` | Saved but not yet submitted. |

## `GET /api/Ticket/{ticketId}/details`: ticket details

Returns the detail lines (the individual inventory/service records that make
up the ticket) and the ticket's notes. Call this after an order/ticket
completes (e.g. on the `Closed` webhook — see [webhooks.md](webhooks.md)) to
pull provisioned line data.

```bash
curl -X GET "https://api.bruin.com/api/Ticket/3204082/details" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

Optional `api-version` header: `"1.0"` (default) or `"2.0"`. Under `2.0`,
each `ticketDetails[]` entry also includes `SkillName` alongside
`subcategoryName`.

### Response

```json
{
  "ticketDetails": [
    {
      "detailID": 2746938,
      "detailType": "Repair_WTN",
      "detailStatus": "O",
      "detailValue": "3059473030",
      "assignedToName": "0",
      "currentTaskID": 7526950,
      "currentTaskName": "COM",
      "currentTaskStatus": null,
      "lastUpdatedBy": 166019,
      "lastUpdatedAt": "2017-06-29T16:12:29.237+02:00",
      "subcategoryName": null
    }
  ],
  "ticketNotes": [
    {
      "noteId": 41894040,
      "noteType": null,
      "noteValue": "test",
      "serviceNumber": ["397_7af21b17-13b7-45c6-ad88-96a2cbaeb6de"],
      "createdDate": "2016-03-30T17:59:31.237-04:00",
      "creator": ""
    }
  ]
}
```

| Field | Notes |
| --- | --- |
| `ticketDetails[].detailID` | Unique ID of the detail line — pass as `DetailId` to the PON endpoint below. |
| `ticketDetails[].detailType` | e.g. `"WTN"` (assigned phone number), `"Repair_WTN"`, `"BillGrpID"`. |
| `ticketDetails[].detailValue` | The value for that detail type, e.g. the phone number itself. |
| `ticketDetails[].currentTaskName` / `currentTaskStatus` | Where the detail line is in its workflow. |
| `ticketNotes[].serviceNumber` | Array — a note can be tied to multiple service numbers. |

**PIAB tip:** look for `ticketNotes[]` entries whose `noteType` matches
`FXS<n>Info` (e.g. `FXS1Info`) — `noteValue` is a `|`-delimited string
containing `DID:` (assigned phone number) and `Line Type:` among other
fields.

## `GET /api/Ticket/{ticketId}/pon`: PON status

Resolve a detail line **either** by `DetailId` **or** by `ServiceNumber` —
provide one of the two as a query param.

```bash
curl -X GET "https://api.bruin.com/api/Ticket/3604522/pon?DetailId=2873611" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

### Response

```json
{
  "ticketId": 3604522,
  "originalRequest": { "serviceNumber": null, "detailId": 2873611 },
  "detailPONStatuses": [
    {
      "detailId": 2873611,
      "serviceNumber": "3043425967",
      "ponAndStatus": [
        { "pon": "LT23724077", "ponStatus": null, "requestType": "LineTest_Voice", "requestStatus": null, "scheduleDate": null },
        { "pon": "IW23724078", "ponStatus": "CA (PON not needed)", "requestType": "MetTel_InsideWire", "requestStatus": "Cancelled", "scheduleDate": null }
      ]
    }
  ]
}
```

`detailPONStatuses[]` has one entry per detail line resolved; each carries a
`ponAndStatus[]` array since a single detail line can have multiple PONs
(e.g. a line test PON and an inside-wire PON).

## Errors

All three endpoints return the standard error schema on failure (`400` malformed
request, `401` missing/expired token, `403` missing the
`FunctionPermissionTicketGet` scope, `500` with the validation body below):

```json
{
  "message": "An error occurred while processing your request.",
  "messageDetail": "…",
  "data": { "ticketId": ["ticketId is required."] },
  "type": "ValidationError",
  "code": 500,
  "traceId": "0HMV4T9P1G2K5:00000001"
}
```

See [ticket-model.md](ticket-model.md) for the same shape on write endpoints.

# Users (assignees & contacts)

A **Bruin user** is anyone provisioned to view the portal, receive ticket
notifications, be assigned inventory, or be referenced as a ticket contact.
Every user has a unique `userID` (GUID) scoped to one `ClientID`. When inventory
or a ticket has an "assignee", that assignee lives here.

**Use this to sync an HR system into Bruin, look up a `userID` before assigning
work, change roles/contact info, or manage ticket-notification subscriptions.**

## Identifiers (pick the one you have)

- `userID` — Bruin's internal GUID; matches `assigneeUserId` on inventory/tickets.
- `username` — login (typically email); accepted as a `PUT` lookup key.
- `employeeID` — the client's external HR ID; accepted as a `PUT` lookup key
  (case-sensitive exact match).

## Status values

- `A` — Active · `D` — Disabled · `L` — Locked

## Scopes

- `FunctionPermissionUserGet` — read (`GET`)
- `FunctionPermissionUserUpdate` — create/update (`POST` / `PUT`)

---

## `GET /api/User` — list / filter users

Returns `{ "documents": [ … ] }`.

> **No pagination.** Returns the **entire** matching result set in one response —
> no `page`/`limit`/`offset` parameters, no cursor. Filter to narrow it; expect a
> single `documents` array, not pages.

> **Suggested approach — dump first, then decide.** Since one call returns every
> user, make a broad request (scoped by `ClientID`), **look at the raw `documents`
> dump**, and then decide:
> - Check whether a person already exists (by `username`/email or `employeeID`)
>   before `POST /api/User` — avoids duplicate accounts (email must be unique).
> - Grab a `userID` / `username` to assign inventory, set a contact, or target a
>   `PUT /api/User` update.
> - Reconcile against your HR system — spot who to create, update, or disable
>   (`PUT` with `Status: "D"`) in bulk.
>
> Pull the list, inspect the roles/status/identifiers present, *then* choose the
> create/update/assign action.

### Query parameters (all optional filters)

| Param | Match behavior |
| --- | --- |
| `ClientID` | Bruin Client ID. |
| `UserID` | Exact, case-sensitive (GUID). |
| `UserName` | Substring, case-sensitive. |
| `FirstName` / `LastName` | Substring, case-sensitive. |
| `Address` | Substring, **case-insensitive**. |
| `RoleName` | Substring, **case-insensitive**. |
| `Status` | `A` / `D` / `L`. |
| `EmployeeID` | Exact, case-sensitive. |
| `DirID` | Directory ID. |
| `PhoneNumber` / `MobileNumber` | Exact. |

```bash
curl -X GET "https://api.bruin.com/api/User?ClientID=53206&Status=A&RoleName=admin" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

Response `documents[]` fields: `userID`, `username`, `dirID`, `firstName`,
`lastName`, `clientID`, `title`, `phoneNumber`, `mobileNumber`, `role`,
`address` (single string), `status`, `employeeID`.

---

## `POST /api/User` — create a user

### Body fields

| Field | Required | Notes |
| --- | --- | --- |
| `ClientID` | **Yes** | Bruin Client ID (with `ClientRoleName`, resolves the role). |
| `ClientRoleName` | **Yes** | Role name, e.g. `"ContactOnly"` (max 50). |
| `FirstName` | **Yes** | max 50. |
| `LastName` | **Yes** | max 50. |
| `Email` | **Yes** | max 100, unique; used as the username. |
| `Title` | No | max 50. |
| `Phone` | No | max 10. |
| `Extension` | No | max 6. |
| `Mobile` | No | max 10. |
| `EmployeeID` | No | max 100, unique if provided. |
| `Department` | No | max 50. |
| `Manager` | No | max 100. |
| `Address1` / `Address2` | No | max 50 each. |
| `City` | No | max 30. |
| `State` | No | max 2. |
| `Zip` | No | max 10. |
| `Country` | No | max 3. |
| `HierarchyValue` | No | Pipe-delimited path, max 300. |

```bash
curl -X POST "https://api.bruin.com/api/User" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "ClientID": 53206,
    "ClientRoleName": "ContactOnly",
    "FirstName": "Rnk",
    "LastName": "Cnt",
    "Email": "rnk.cnt@example.com"
  }'
```

Response: `{ "statusCode": 200, "message": "User successfully created", "isSuccess": true, "dirId": 123456 }`

---

## `PUT /api/User` — update a user

Identify the user with a **query parameter**: `userName` **or** `employeeId`
(if both are passed, `userName` wins). The **body** must include at least one of
`fields` or `ticketSubscriptionSettings`.

```bash
curl -X PUT "https://api.bruin.com/api/User?userName=jsmith@example.com" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": { "Title": "Senior Engineer", "Department": "Platform" },
    "ticketSubscriptionSettings": { "repair": { "subscriptionType": "Notes" } }
  }'
```

### `fields` (any subset; omitted keys unchanged)

`Status` (`A`/`D`/`L`), `Title`, `FirstName`, `LastName`, `Email`, `Phone`,
`Extension`, `Mobile`, `ManagerUserName`, `Department`, `Address1`, `Address2`,
`City`, `State`, `Zip`, `Country`, `Hierarchy`.

> To **disable** an offboarded user, `PUT` with `fields: { "Status": "D" }`.

### `ticketSubscriptionSettings` (email notification prefs)

Keys: `newOrder`, `repair`, `serviceChange` — each an object with a
`subscriptionType`:

| Value | Behavior |
| --- | --- |
| `None` | No emails. |
| `Milestones` | Milestones only. |
| `Notes` | Notes only. |
| `Both` | Milestones + notes. |

---

## Errors

`400` invalid/missing fields, `401` missing/expired token, `403` missing the
required scope, `500` server error (standard schema + `traceId`).

> Note: the Swagger only formally declares `500` for `PUT /api/User`; the API
> still returns normal `401`/`403`/success codes.

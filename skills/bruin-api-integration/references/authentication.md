# Authentication & Environments

Bruin uses **OAuth 2.0 client credentials**. You exchange a Client ID + Secret
for a bearer token, then send that token on every API call.

## Prerequisites (client-side, one-time)

- An **active Bruin account with admin permissions** (created by the client's
  Customer Software Engineer or project manager).
- **OAuth credentials** generated in the Bruin portal:
  My Company → Developer Configuration → API Access → **Generate Credentials**.
- The credentials are scoped to specific endpoints. To add an endpoint not
  listed on the API Access page, the client contacts their CSE.

> Credential generation happens in the portal, not via API. If a client hasn't
> generated credentials yet, direct them there — you can't do it for them.

## OAuth Client ID vs. Bruin Client ID

| | OAuth Client ID | Bruin Client ID |
| --- | --- | --- |
| Looks like | opaque string + secret | a number, e.g. `9994` |
| Used for | requesting a bearer token **only** | a parameter/body field in nearly every call |
| Source | portal "Generate Credentials" | provided by the CSE |

When any endpoint asks for `clientId` / `ClientID`, it wants the **Bruin Client
ID**.

## Endpoints by environment

| Environment | Auth endpoint (token) | API base URL (calls) |
| --- | --- | --- |
| Commercial | `https://apigw.bruin.com/authorize/token` | `https://api.bruin.com` |
| Federal (EIS) | `https://id-fed.mettel.net/identity/connect/token` | `https://fedapi.mettel.net` |
| Federal (Non-EIS) | `https://id-federal.mettel.net/identity/connect/token` | `https://federalapi.mettel.net` |

Pick one environment and use its auth endpoint **and** API base together.

## Request a bearer token

`POST` to the auth endpoint with `application/x-www-form-urlencoded` body:

- `grant_type=client_credentials`
- `client_id=OAUTH_CLIENT_ID`
- `client_secret=OAUTH_CLIENT_SECRET`
- `scope=public_api`

```bash
curl -X POST https://apigw.bruin.com/authorize/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=OAUTH_CLIENT_ID" \
  -d "client_secret=OAUTH_CLIENT_SECRET" \
  -d "scope=public_api"
```

```python
import requests

resp = requests.post(
    "https://apigw.bruin.com/authorize/token",
    headers={"Content-Type": "application/x-www-form-urlencoded"},
    data={
        "grant_type": "client_credentials",
        "client_id": "OAUTH_CLIENT_ID",
        "client_secret": "OAUTH_CLIENT_SECRET",
        "scope": "public_api",
    },
)
token = resp.json()["access_token"]
```

### Token response

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "public_api"
}
```

- Tokens expire after **3600 seconds (1 hour)**.
- Refresh **before** expiry rather than reacting to a `401`. A common pattern:
  cache the token with its issue time and re-request at ~55 minutes.

## Use the token

Send it on every call:

```bash
curl -X GET "https://api.bruin.com/api/Inventory?ClientId=9994" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

## Scopes (permissions)

Each credential carries `FunctionPermission*` scopes that gate specific
endpoints:

| Scope | Grants |
| --- | --- |
| `FunctionPermissionInventoryGet` | `GET /api/Inventory`, `GET /api/Inventory/Attribute` |
| `FunctionPermissionSiteGet` | `GET /api/Site` |
| `FunctionPermissionSiteUpdate` | `POST /api/Site` |
| `FunctionPermissionUserGet` | `GET /api/User` |
| `FunctionPermissionUserUpdate` | `POST /api/User`, `PUT /api/User` |

(Ticket endpoints have their own scopes provisioned per account.)

## Auth error cheat-sheet

| Status | Meaning | Fix |
| --- | --- | --- |
| `401 Unauthorized` | Missing/expired/invalid token | Request a fresh token; check you're using the right environment's auth endpoint |
| `403 Forbidden` | Token is valid but the credential lacks the scope for this endpoint | Add the scope via portal/CSE — not a code fix |
| `400 Bad Request` | Malformed token request or missing form fields | Check `grant_type`, `scope=public_api`, and content type |

> Security: store the Client Secret in a secret manager or env var, never in
> source or client-side code. Each organization has unique credentials.

# Cable Internet — Ticket Reference

Cable internet circuit ticket operations. Every ticket is a `POST /api/Ticket`
using the universal body in [../ticket-model.md](../ticket-model.md) — only the
`category` code and the product-specific `notes` below differ. Contacts,
services, and top-level fields follow the ticket model.

## Operations

| Operation | `category` | Purpose |
| --- | --- | --- |
| Add Change Delete Feature | `025` | Add, change, or delete a feature on a cable service (e.g. static IPs, IP version, modem settings). |
| Circuit Sunset Project | `OOO` | Decommission a cable circuit as part of a sunset project. |
| Disconnect | `020` | Disconnect a cable service and record the disconnect reason. |
| Enable/Disable WiFi | `444` | Enable or disable WiFi on the service's modem. |
| New Order | `010` | Order a new cable service. |
| Replace/Move Modem | `VV6` | Replace or move the modem for a cable service. |
| Service Affecting Trouble | `VAS` | Report a service-affecting cable trouble (degraded but not fully down). |
| Service Outage Trouble | `VOO` | Report a full cable service outage. |
| Upgrade Existing Circuit | `088` | Upgrade an existing cable circuit. |

All operations use Subcategory ID `23`. In the note objects below, `MTK` (problem
description) and `DDD` (desired due date, `MM/DD/YYYY`) are universal ticket-model
notes and are shown in examples but not documented as product-specific notes.

## Add Change Delete Feature — `025`

Add, change, or delete a feature on a cable service. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "025",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "dslrequesttype",
            "noteValue": "Add Static IP Address(es)"
        },
        {
            "noteType": "modemsetting",
            "noteValue": "Sample value"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`dslrequesttype`** **(Required)** — Broadband request type. Changes are subject to service availability based on product type and service address. Allowed values:
  - `"Add Static IP Address(es)"`
  - `"Change from IPV4 to IPV6"`
  - `"Convert from Static to Dynamic"`
  - `"Other"` → **`OtherMDMRequest`** **(Conditional — Required)** — Other request; describe the request being made. Free-text.
  - `"Remove Static IP Address(es)"`
  - `"TSP Update"` → **`TSP Code`** **(Conditional — Required)** — TSP code to apply to this service. Free-text.
- **`modemsetting`** **(Optional)** — Changes needed to modem settings. Some changes may be limited by modem make/model, and some may require a technician dispatch (dispatch fee). Free-text.

## Circuit Sunset Project — `OOO`

Decommission a cable circuit as part of a sunset project. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "OOO",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "ReplaceDecomCircuit",
            "noteValue": "No"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`ReplaceDecomCircuit`** **(Required)** — Replacing decommissioned circuit? Allowed values:
  - `"No"`
  - `"Yes"` reveals:
    - **`DecomissionProject`** **(Conditional — Required)** — Name or ID of the decommission project. Free-text.
    - **`ProjectDeadline`** **(Conditional — Required)** — Project deadline. Date (`MM/DD/YYYY`).
    - **`CustomerWAN?`** **(Conditional — Required)** — Whether the customer is providing their own WAN. Free-text.

## Disconnect — `020`

Disconnect a cable service and record the disconnect reason. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "020",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "DRN",
            "noteValue": "1"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`DRN`** **(Required)** — Disconnect reason. Allowed values:
  - `"1"` — Downsizing Services at Location
  - `"2"` — Closing Location
  - `"3"` — Technology Upgrade
  - `"4"` — Moving to New Location → **`moveserviceoption`** **(Conditional — Required)** — What should happen to the service at the new location. Free-text.
  - `"5"` — Non-Payment
  - `"6"` — Other → **`descother`** **(Conditional — Required)** — Description of the disconnect reason. Free-text.
- **`discauthdoc`** **(Optional)** — Disconnection authorization paperwork; upload the signed authorization paperwork if applicable. Accepts a file upload.

## Enable/Disable WiFi — `444`

Enable or disable WiFi on the service's modem. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "444",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "WiFiRequest",
            "noteValue": "Disable"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`WiFiRequest`** **(Required)** — WiFi request type. Allowed values:
  - `"Disable"`
  - `"Enable"` reveals:
    - **`SSIDUsername`** **(Conditional — Required)** — WiFi network name (SSID). Free-text.
    - **`SSIDPwd`** **(Conditional — Required)** — WiFi network (SSID) password. Free-text.

## New Order — `010`

Order a new cable service. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "010",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "Migration",
            "noteValue": "No"
        },
        {
            "noteType": "ReplacementOrder",
            "noteValue": "false"
        },
        {
            "noteType": "SpecialCarrierNotes",
            "noteValue": "Sample value"
        },
        {
            "noteType": "SpecialSurveyNotes",
            "noteValue": "Sample value"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`Migration`** **(Required)** — Migrating service. Allowed values:
  - `"No"`
  - `"Yes"`
- **`ReplacementOrder`** **(Optional)** — Cancel and replace order; indicate if this ticket replaces a previous order. Allowed values:
  - `"true"` — This ticket cancels and replaces a previous order → **`OriginalTicketorPON`** **(Conditional — Required)** — Original ticket number or PON being replaced. Free-text.
  - `"false"` — This is a new order, not a replacement.
- **`SpecialCarrierNotes`** **(Optional)** — Special carrier notes. Free-text.
- **`SpecialSurveyNotes`** **(Optional)** — Special survey notes. Free-text.

## Replace/Move Modem — `VV6`

Replace or move the modem for a cable service. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "VV6",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "ModemRequest",
            "noteValue": "Move"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`ModemRequest`** **(Required)** — Modem request type. Allowed values:
  - `"Move"` reveals:
    - **`CurrentAddress`** **(Conditional — Optional)** — Current service address of the modem. Free-text.
    - **`MoveAddress`** **(Conditional — Optional)** — New address the modem is being moved to. Free-text.
  - `"Replace"`

## Service Affecting Trouble — `VAS`

Report a service-affecting cable trouble (degraded but not fully down). Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "VAS",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "IntrTest",
            "noteValue": "false"
        },
        {
            "noteType": "Light Status",
            "noteValue": "true"
        },
        {
            "noteType": "cableproblem",
            "noteValue": "1"
        },
        {
            "noteType": "fieldtech",
            "noteValue": "false"
        },
        {
            "noteType": "rebootmodem",
            "noteValue": "1"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`IntrTest`** **(Optional)** — Intrusive testing authorized. Allowed values:
  - `"true"` — Intrusive testing is authorized.
  - `"false"` — Intrusive testing is not authorized.
- **`Light Status`** **(Optional)** — Can you provide modem light status? Allowed values:
  - `"true"` — Yes, can provide the modem light status. Reveals:
    - **`powerstatus`** **(Conditional — Optional)** — Power status of the modem. Free-text.
    - **`dslsynclight`** **(Conditional — Optional)** — Status of the DSL sync light on the modem. Free-text.
  - `"false"` — No, cannot provide the modem light status.
- **`cableproblem`** **(Required)** — Nature of the cable problem. Allowed values:
  - `"1"` — Slow connection / intermittent connection
  - `"2"` — Internet completely down
- **`fieldtech`** **(Optional)** — MetTel field tech assist? Selecting Yes agrees to associated fees (invoiced at contracted ISW rates). Allowed values:
  - `"true"` — Request a MetTel field technician dispatch. Reveals:
    - **`HoursTechAssist`** **(Conditional — Required)** — Number of hours of technician assistance requested. Free-text.
    - **`TechHours`** **(Conditional — Required)** — Technician hours needed. Free-text.
    - **`timetechdispatch`** **(Conditional — Required)** — Requested dispatch time. Free-text.
    - **`techsow`** **(Conditional — Required)** — Scope of work for the technician. Free-text.
  - `"false"` — Do not request a field technician dispatch.
- **`rebootmodem`** **(Required)** — Have you rebooted the modem? Rebooting is a prerequisite to submitting a trouble ticket; fees may apply if skipped. Allowed values:
  - `"1"` — Yes, modem has been rebooted
  - `"2"` — No, modem has not been rebooted

## Service Outage Trouble — `VOO`

Report a full cable service outage. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "VOO",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "IntrTest",
            "noteValue": "false"
        },
        {
            "noteType": "Light Status",
            "noteValue": "true"
        },
        {
            "noteType": "cableproblem",
            "noteValue": "1"
        },
        {
            "noteType": "fieldtech",
            "noteValue": "false"
        },
        {
            "noteType": "rebootmodem",
            "noteValue": "1"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`IntrTest`** **(Optional)** — Intrusive testing authorized. Allowed values:
  - `"true"` — Intrusive testing is authorized.
  - `"false"` — Intrusive testing is not authorized.
- **`Light Status`** **(Optional)** — Can you provide modem light status? Allowed values:
  - `"true"` — Yes, can provide the modem light status. Reveals:
    - **`powerstatus`** **(Conditional — Optional)** — Power status of the modem. Free-text.
    - **`dslsynclight`** **(Conditional — Optional)** — Status of the DSL sync light on the modem. Free-text.
  - `"false"` — No, cannot provide the modem light status.
- **`cableproblem`** **(Required)** — Nature of the cable problem. Allowed values:
  - `"1"` — Slow connection / intermittent connection
  - `"2"` — Internet completely down
- **`fieldtech`** **(Optional)** — MetTel field tech assist? Selecting Yes agrees to associated fees (invoiced at contracted ISW rates). Allowed values:
  - `"true"` — Request a MetTel field technician dispatch. Reveals:
    - **`HoursTechAssist`** **(Conditional — Required)** — Number of hours of technician assistance requested. Free-text.
    - **`TechHours`** **(Conditional — Required)** — Technician hours needed. Free-text.
    - **`timetechdispatch`** **(Conditional — Required)** — Requested dispatch time. Free-text.
    - **`techsow`** **(Conditional — Required)** — Scope of work for the technician. Free-text.
  - `"false"` — Do not request a field technician dispatch.
- **`rebootmodem`** **(Required)** — Have you rebooted the modem? Rebooting is a prerequisite to submitting a trouble ticket; fees may apply if skipped. Allowed values:
  - `"1"` — Yes, modem has been rebooted
  - `"2"` — No, modem has not been rebooted

## Upgrade Existing Circuit — `088`

Upgrade an existing cable circuit. Subcategory ID: `23`.

### Example body

```json
{
    "clientId": 9994,
    "category": "088",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "SpecialCarrierNotes",
            "noteValue": "Sample value"
        },
        {
            "noteType": "SpecialSurveyNotes",
            "noteValue": "Sample value"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "NoteType": "DDD",
            "NoteValue": "04/22/2026"
        }
    ],
    "contacts": [
        {
            "email": "contact@example.com",
            "phone": "9827783377",
            "name": "Site Contact",
            "type": "Site"
        },
        {
            "email": "ticket@example.com",
            "phone": "9827783377",
            "name": "Ticket Contact",
            "type": "Ticket"
        }
    ]
}
```

### Notes

Legend: **(Required)** must be present · **(Optional)** may be omitted · **(Conditional)** required/optional only under a specific parent value.

- **`SpecialCarrierNotes`** **(Optional)** — Special carrier notes. Free-text.
- **`SpecialSurveyNotes`** **(Optional)** — Special survey notes. Free-text.

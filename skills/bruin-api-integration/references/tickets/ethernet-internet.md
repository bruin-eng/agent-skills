# Ethernet Internet — Ticket Reference

Ethernet internet circuit ticket operations. Every ticket is a `POST /api/Ticket`
using the universal body in [../ticket-model.md](../ticket-model.md) — only the
`category` code and the product-specific `notes` below differ. Contacts,
services, and top-level fields follow the ticket model.

## Operations

| Operation | `category` | Purpose |
| --- | --- | --- |
| Add Change Delete Feature | `025` | Request a MAC (move/add/change/delete) feature change on an existing circuit. |
| Circuit Sunset Project | `OOO` | Record a circuit as part of a decommission/sunset project. |
| Disconnect | `020` | Disconnect an existing circuit. |
| New Order | `010` | Order a new Ethernet internet circuit. |
| Service Affecting Trouble | `VAS` | Report a service-affecting (degraded) trouble condition. |
| Service Outage Trouble | `VOO` | Report a full service outage trouble condition. |
| Upgrade Existing Circuit | `088` | Upgrade an existing Ethernet internet circuit. |

## Add Change Delete Feature — `025`

Request a MAC feature change (move/add/change/delete) on an existing Ethernet internet circuit. Subcategory ID: `66`.

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
            "noteType": "datamacdfeature",
            "noteValue": "Demarc Extension"
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

- **`datamacdfeature`** **(Required)** — Data Request Type. Requested changes are subject to service availability based on product type and service address. Allowed values:
  - value `"Demarc Extension"` — Demarc Extension
  - value `"IP Change"` — IP Change
    - → **`ipchangereq`** **(Conditional — Required)** — IP Change Request; describe the IP change being requested. Free-text.
  - value `"Modem/Router Reconfiguration Request"` — Modem/Router Reconfiguration Request
    - → **`configdetails`** **(Conditional — Required)** — Configuration Details; provide the modem/router reconfiguration details. Free-text.
  - value `"Order New Modem/Router - Hardware Upgrade"` — Order New Modem/Router - Hardware Upgrade
    - → **`newhardwarereq`** **(Conditional — Required)** — New Hardware Request; describe the new modem/router hardware being requested. Free-text.
  - value `"Other"` — Other
    - → **`OtherMDMRequest`** **(Conditional — Required)** — Other Request; describe the request being made. Free-text.
  - value `"TSP Update"` — TSP Update
    - → **`TSP Code`** **(Conditional — Required)** — TSP Code; enter the TSP code to apply to this service. Free-text.

## Circuit Sunset Project — `OOO`

Record an Ethernet internet circuit as part of a decommission/sunset project. Subcategory ID: `66`.

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

- **`ReplaceDecomCircuit`** **(Required)** — Replacing Decommissioned Circuit? Allowed values:
  - value `"No"` — No
  - value `"Yes"` — Yes
    - → **`DecomissionProject`** **(Conditional — Required)** — Decommission Project; provide the name or ID of the decommission project. Free-text.
    - → **`ProjectDeadline`** **(Conditional — Required)** — Project Deadline; enter the project deadline. Date (format: `MM/DD/YYYY`).
    - → **`CustomerWAN?`** **(Conditional — Required)** — Customer WAN?; indicate whether the customer is providing their own WAN. Free-text.

## Disconnect — `020`

Disconnect an existing Ethernet internet circuit. Subcategory ID: `66`.

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

- **`DRN`** **(Required)** — Disconnect Reason. Allowed values:
  - value `"1"` — Downsizing Services at Location
  - value `"2"` — Closing Location
  - value `"3"` — Technology Upgrade
  - value `"4"` — Moving to New Location
    - → **`moveserviceoption`** **(Conditional — Required)** — Move Service Option; specify what should happen to the service at the new location. Free-text.
  - value `"5"` — Non-Payment
  - value `"6"` — Other
    - → **`descother`** **(Conditional — Required)** — Description; provide a description of the disconnect reason. Free-text.
- **`discauthdoc`** **(Optional)** — Disconnection Authorization Paperwork; upload the signed authorization paperwork if applicable. Accepts a file upload.

## New Order — `010`

Order a new Ethernet internet circuit. Subcategory ID: `66`.

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
            "noteType": "ConnectorType",
            "noteValue": "LC"
        },
        {
            "noteType": "DMARC",
            "noteValue": "Sample value"
        },
        {
            "noteType": "EthernetHandoffType",
            "noteValue": "Carrier Preference (Electrical/Optical)"
        },
        {
            "noteType": "Migration",
            "noteValue": "No"
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
            "noteType": "TSP",
            "noteValue": "false"
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

- **`ConnectorType`** **(Optional)** — Connector Type. Allowed values:
  - value `"LC"` — LC
  - value `"SC"` — SC
- **`DMARC`** **(Required)** — DEMARC. Failure to provide the correct BLDG/FL/STE/RM will result in the carrier delivering the service to the incorrect MPOE (Minimum Point of Entry); this can result in a reissue and a new SLA will apply. Free-text.
- **`EthernetHandoffType`** **(Required)** — Handoff Type. No guarantee the specified handoff is available or can be delivered. Allowed values:
  - value `"Carrier Preference (Electrical/Optical)"` — Carrier Preference (Electrical/Optical)
  - value `"Electrical - RJ45"` — Electrical - RJ45
  - value `"Multimode Fiber (MMF) - LC"` — Multimode Fiber (MMF) - LC
  - value `"Singlemode Fiber (SMF) - LC"` — Singlemode Fiber (SMF) - LC
- **`Migration`** **(Required)** — Migrating Service. Allowed values:
  - value `"No"` — No
  - value `"Yes"` — Yes
- **`SpecialCarrierNotes`** **(Optional)** — Special Carrier Notes. Free-text.
- **`SpecialSurveyNotes`** **(Optional)** — Special Survey Notes. Free-text.
- **`TSP`** **(Optional)** — TSP. Allowed values:
  - value `"true"` — This service requires a TSP code.
    - → **`TSP Code`** **(Conditional — Required)** — TSP Code; enter the TSP code to apply to this service. Free-text.
  - value `"false"` — No TSP code is required.

## Service Affecting Trouble — `VAS`

Report a service-affecting (degraded) trouble condition on an Ethernet internet circuit. Subcategory ID: `66`.

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
            "noteType": "fieldtech",
            "noteValue": "false"
        },
        {
            "noteType": "mplsinternetproblem",
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

- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? Selecting "Yes"/true agrees to associated fees; MetTel can dispatch an independent technician to assist troubleshooting, invoiced at contracted ISW rates. Allowed values:
  - value `"true"` — Request a MetTel field technician dispatch.
    - → **`HoursTechAssist`** **(Conditional — Required)** — Hours of Tech Assist; enter the number of hours of technician assistance requested. Free-text.
    - → **`TechHours`** **(Conditional — Required)** — Tech Hours; specify the technician hours needed. Free-text.
    - → **`timetechdispatch`** **(Conditional — Required)** — Time of Tech Dispatch; specify the requested dispatch time. Free-text.
    - → **`techsow`** **(Conditional — Required)** — Tech Scope of Work; describe the scope of work for the technician. Free-text.
  - value `"false"` — Do not request a field technician dispatch.
- **`mplsinternetproblem`** **(Required)** — Nature of the internet connection problem. Allowed values:
  - value `"1"` — Slow connection / intermittent connection
    - → **`intrusive testing`** **(Conditional — Required)** — Intrusive Testing; indicate whether intrusive testing is permitted. Free-text.
  - value `"2"` — Internet completely down
    - → **`powertomodem`** **(Conditional — Required)** — Power to Modem; indicate whether there is currently power to the modem. Free-text.

## Service Outage Trouble — `VOO`

Report a full service outage trouble condition on an Ethernet internet circuit. Subcategory ID: `66`.

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
            "noteType": "fieldtech",
            "noteValue": "false"
        },
        {
            "noteType": "mplsinternetproblem",
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

- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? Selecting "Yes"/true agrees to associated fees; MetTel can dispatch an independent technician to assist troubleshooting, invoiced at contracted ISW rates. Allowed values:
  - value `"true"` — Request a MetTel field technician dispatch.
    - → **`HoursTechAssist`** **(Conditional — Required)** — Hours of Tech Assist; enter the number of hours of technician assistance requested. Free-text.
    - → **`TechHours`** **(Conditional — Required)** — Tech Hours; specify the technician hours needed. Free-text.
    - → **`timetechdispatch`** **(Conditional — Required)** — Time of Tech Dispatch; specify the requested dispatch time. Free-text.
    - → **`techsow`** **(Conditional — Required)** — Tech Scope of Work; describe the scope of work for the technician. Free-text.
  - value `"false"` — Do not request a field technician dispatch.
- **`mplsinternetproblem`** **(Required)** — Nature of the internet connection problem. Allowed values:
  - value `"1"` — Slow connection / intermittent connection
    - → **`intrusive testing`** **(Conditional — Required)** — Intrusive Testing; indicate whether intrusive testing is permitted. Free-text.
  - value `"2"` — Internet completely down
    - → **`powertomodem`** **(Conditional — Required)** — Power to Modem; indicate whether there is currently power to the modem. Free-text.

## Upgrade Existing Circuit — `088`

Upgrade an existing Ethernet internet circuit. Subcategory ID: `66`.

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
            "noteType": "ConnectorType",
            "noteValue": "LC"
        },
        {
            "noteType": "DMARC",
            "noteValue": "Sample value"
        },
        {
            "noteType": "EthernetHandoffType",
            "noteValue": "Carrier Preference (Electrical/Optical)"
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

- **`ConnectorType`** **(Optional)** — Connector Type. Allowed values:
  - value `"LC"` — LC
  - value `"SC"` — SC
- **`DMARC`** **(Required)** — DEMARC. Failure to provide the correct BLDG/FL/STE/RM will result in the carrier delivering the service to the incorrect MPOE (Minimum Point of Entry); this can result in a reissue and a new SLA will apply. Free-text.
- **`EthernetHandoffType`** **(Required)** — Handoff Type. No guarantee the specified handoff is available or can be delivered. Allowed values:
  - value `"Carrier Preference (Electrical/Optical)"` — Carrier Preference (Electrical/Optical)
  - value `"Electrical - RJ45"` — Electrical - RJ45
  - value `"Multimode Fiber (MMF) - LC"` — Multimode Fiber (MMF) - LC
  - value `"Singlemode Fiber (SMF) - LC"` — Singlemode Fiber (SMF) - LC
- **`SpecialCarrierNotes`** **(Optional)** — Special Carrier Notes. Free-text.
- **`SpecialSurveyNotes`** **(Optional)** — Special Survey Notes. Free-text.

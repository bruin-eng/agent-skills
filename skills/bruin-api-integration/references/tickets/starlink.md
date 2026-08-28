# Starlink — Ticket Reference

Starlink ticket operations. Every ticket is a `POST /api/Ticket`
using the universal body in [../ticket-model.md](../ticket-model.md) — only the
`category` code and the product-specific `notes` below differ. Contacts,
services, and top-level fields follow the ticket model.

All operations target the **Satellite** product. Subcategory IDs: `315`, `412`.

## Operations

| Operation | `category` | Purpose |
| --- | --- | --- |
| Add Change Delete Feature | `025` | Request broadband feature or modem setting changes on a service. |
| Disconnect | `020` | Disconnect a service and record the disconnect reason. |
| Equipment RMA | `125` | Return/replace defective equipment and ship a replacement device. |
| Return Device | `058` | Return a device and record the return/RMA reason. |
| Service Affecting Trouble | `VAS` | Report a service-affecting internet problem (degraded service). |
| Service Outage Trouble | `VOO` | Report a full internet outage or connection problem. |
| Update Management Status | `JJJ` | Update the management status of a Starlink device by serial number. |
| Upgrade Starlink Circuit | `005` | Upgrade a Starlink circuit, optionally with carrier/survey notes. |
| Suspend | `SUS` | Suspend a service (no category-specific notes). |
| Unsuspend Service | `022` | Unsuspend a service (no category-specific notes). |

## Add Change Delete Feature — `025`

Request broadband feature or modem setting changes on a Satellite service. Subcategory IDs: `315`, `412`.

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
            "noteValue": "N/A"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`dslrequesttype`** **(Required)** — Broadband Request Type. Requested changes are subject to service availability based on product type and service address. Allowed values:
  - `"Add Static IP Address(es)"` — Request to add one or more static IP addresses to this service.
  - `"Change from IPV4 to IPV6"` — Request to convert the IP version from IPv4 to IPv6.
  - `"Convert from Static to Dynamic"` — Request to convert this service from a static IP to a dynamic IP assignment.
  - `"Other"` — Request for a broadband feature change not listed above.
    - value `"Other"` → **`OtherMDMRequest`** **(Conditional — Required)** — Other Request. Describe the broadband feature change being requested. Free-text input.
  - `"Remove Static IP Address(es)"` — Request to remove one or more static IP addresses from this service.
  - `"TSP Update"` — Request to update the Telephone Service Priority (TSP) code for this service.
    - value `"TSP Update"` → **`TSP Code`** **(Conditional — Required)** — TSP Code to apply to this service. Free-text input.
- **`modemsetting`** **(Required)** — Change Needed to Modem Settings? Indicate any changes needed to the modem settings. Some changes may be limited by modem make/model, and some may require a technician dispatch (dispatch fee applies). Free-text input.

## Disconnect — `020`

Disconnect a Satellite service and record the disconnect reason. Subcategory IDs: `315`, `412`.

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
            "noteValue": "2"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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
  - `"1"` — Downsizing Services at Location.
  - `"2"` — Closing Location.
  - `"3"` — Technology Upgrade.
  - `"4"` — Moving to New Location.
    - value `"4"` → **`moveserviceoption`** **(Conditional — Required)** — Move Service Option. Specify what should happen to the service at the new location. Free-text input.
  - `"5"` — Non-Payment.
  - `"6"` — Other.
    - value `"6"` → **`descother`** **(Conditional — Required)** — Description. Provide a description of the disconnect reason. Free-text input.
- **`discauthdoc`** **(Optional)** — Disconnection Authorization Paperwork. Upload the signed authorization paperwork if applicable. Accepts a file upload attachment.

## Equipment RMA — `125`

Return/replace defective equipment and ship a replacement device. Subcategory IDs: `315`, `412`.

### Example body

```json
{
    "clientId": 9994,
    "category": "125",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "AttentionTo",
            "noteValue": "JOHN SMITH"
        },
        {
            "noteType": "ShippingAddress",
            "noteValue": "123 Main St, Springfield, IL 62701"
        },
        {
            "noteType": "Warranty Reasons",
            "noteValue": "7"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`AttentionTo`** **(Required)** — Attention To. Name of the person the shipment should be addressed to. Free-text input.
- **`RefRepairTicket`** **(Optional)** — Reference Repair Ticket Number. Ticket number of a related repair ticket if applicable. Accepts a dropdown selection.
- **`ShippingAddress`** **(Required)** — Shipping Address. Address where the replacement device should be shipped. Accepts a structured address.
- **`Warranty Reasons`** **(Required)** — Return/RMA Reason. Allowed values:
  - `"1"` — Device Not Charging
  - `"4"` — Screen Not Working
  - `"5"` — Other
  - `"6"` — Damaged In Transit
  - `"7"` — Defective Device
  - `"8"` — Incorrect Address
  - `"10"` — Missing Component
  - `"11"` — Lost/Stolen
  - `"12"` — Closed Location

## Return Device — `058`

Return a device and record the return/RMA reason. Subcategory IDs: `315`, `412`.

### Example body

```json
{
    "clientId": 9994,
    "category": "058",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "Warranty Reasons",
            "noteValue": "7"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`Warranty Reasons`** **(Required)** — Return/RMA Reason. Allowed values:
  - `"1"` — Device Not Charging
  - `"4"` — Screen Not Working
  - `"5"` — Other
  - `"6"` — Damaged In Transit
  - `"7"` — Defective Device
  - `"8"` — Incorrect Address
  - `"10"` — Missing Component
  - `"11"` — Lost/Stolen
  - `"12"` — Closed Location

## Service Affecting Trouble — `VAS`

Report a service-affecting internet problem (degraded service) on a Satellite service. Subcategory IDs: `315`, `412`.

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
            "noteType": "fttiproblem",
            "noteValue": "1"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? Selecting `true` agrees to associated fees; MetTel can dispatch an independent technician (invoiced at contracted ISW rates). Allowed values:
  - `"true"` — Request a MetTel field technician dispatch.
    - value `"true"` → **`HoursTechAssist`** **(Conditional — Required)** — Hours of Tech Assist. Number of hours of technician assistance requested. Free-text input.
    - value `"true"` → **`TechHours`** **(Conditional — Required)** — Tech Hours. Technician hours needed. Free-text input.
    - value `"true"` → **`timetechdispatch`** **(Conditional — Required)** — Time of Tech Dispatch. Requested dispatch time. Free-text input.
    - value `"true"` → **`techsow`** **(Conditional — Required)** — Tech Scope of Work. Describe the scope of work for the technician. Free-text input.
  - `"false"` — Do not request a field technician dispatch.
- **`fttiproblem`** **(Conditional — Required for subcategory 412)** — Nature of the Fiber Internet Problem. Use instead of `mplsinternetproblem` for subcategory `412`. Allowed values:
  - `"1"` — Slow connection or intermittent connection.
  - `"2"` — Internet completely down.
- **`mplsinternetproblem`** **(Conditional — Required for subcategory 315)** — Nature of the Internet Connection Problem. Use instead of `fttiproblem` for subcategory `315`. Allowed values:
  - `"1"` — Slow connection or intermittent connection.
    - value `"1"` → **`intrusive testing`** **(Conditional — Required)** — Intrusive Testing. Indicate whether intrusive testing is authorized. Free-text input.
  - `"2"` — Internet completely down.
    - value `"2"` → **`powertomodem`** **(Conditional — Required)** — Power to Modem. Indicate whether there is currently power to the modem. Free-text input.

## Service Outage Trouble — `VOO`

Report a full internet outage or connection problem on a Satellite service. Subcategory IDs: `315`, `412`.

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
            "noteType": "intrusive testing",
            "noteValue": "Yes"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? Selecting `true` agrees to associated fees; MetTel can dispatch an independent technician (invoiced at contracted ISW rates). Allowed values:
  - `"true"` — Request a MetTel field technician dispatch.
    - value `"true"` → **`HoursTechAssist`** **(Conditional — Required)** — Hours of Tech Assist. Number of hours of technician assistance requested. Free-text input.
    - value `"true"` → **`TechHours`** **(Conditional — Required)** — Tech Hours. Technician hours needed. Free-text input.
    - value `"true"` → **`timetechdispatch`** **(Conditional — Required)** — Time of Tech Dispatch. Requested dispatch time. Free-text input.
    - value `"true"` → **`techsow`** **(Conditional — Required)** — Tech Scope of Work. Describe the scope of work for the technician. Free-text input.
  - `"false"` — Do not request a field technician dispatch.
- **`mplsinternetproblem`** **(Required)** — Nature of the Internet Connection Problem. Allowed values:
  - `"1"` — Slow connection or intermittent connection.
    - value `"1"` → **`intrusive testing`** **(Conditional — Required)** — Intrusive Testing. Indicate whether intrusive testing is authorized. Free-text input.
  - `"2"` — Internet completely down.
    - value `"2"` → **`powertomodem`** **(Conditional — Required)** — Power to Modem. Indicate whether there is currently power to the modem. Free-text input.

## Update Management Status — `JJJ`

Update the management status of a Starlink device identified by serial number. Subcategory IDs: `315`, `412`.

### Example body

```json
{
    "clientId": 9994,
    "category": "JJJ",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "SerialNumber",
            "noteValue": "STARLINK-SN-12345"
        },
        {
            "noteType": "ManagementStatus",
            "noteValue": "STATUS VALUE HERE"
        },
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`SerialNumber`** **(Required)** — Serial Number of the Starlink device. Free-text input.
- **`ManagementStatus`** **(Required)** — Management Status to apply to the device. Accepts a dropdown selection.

## Upgrade Starlink Circuit — `005`

Upgrade a Starlink circuit, optionally with special carrier and survey notes. Subcategory IDs: `315`, `412`.

### Example body

```json
{
    "clientId": 9994,
    "category": "005",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

- **`SpecialCarrierNotes`** **(Optional)** — Special Carrier Notes. Any special notes for the carrier regarding this circuit upgrade. Free-text input.
- **`SpecialSurveyNotes`** **(Optional)** — Special Survey Notes. Any special notes relevant to the site survey for this circuit upgrade. Free-text input.

## Suspend — `SUS`

Suspend a Satellite service. Subcategory IDs: `315`, `412`.

### Example body

```json
{
    "clientId": 9994,
    "category": "SUS",
    "Services": [
        {
            "ServiceNumber": "12.rbcb.123456"
        }
    ],
    "notes": [
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

This ticket type does not require any category-specific note types. Submit the request with only the universal `MTK` and `DDD` notes (see the example body).

## Unsuspend Service — `022`

Unsuspend a Satellite service. Subcategory IDs: `315`, `412`.

### Example body

```json
{
    "clientId": 9994,
    "category": "022",
    "Services": [
        {
            "ServiceNumber": "12.rbcb.123456"
        }
    ],
    "notes": [
        {
            "noteType": "MTK",
            "noteValue": "PROBLEM DESCRIPTION HERE"
        },
        {
            "noteType": "DDD",
            "noteValue": "04/22/2026"
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

This ticket type does not require any category-specific note types. Submit the request with only the universal `MTK` and `DDD` notes (see the example body).

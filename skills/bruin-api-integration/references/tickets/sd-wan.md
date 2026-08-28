# SD-WAN — Ticket Reference

SD-WAN ticket operations. Every ticket is a `POST /api/Ticket`
using the universal body in [../ticket-model.md](../ticket-model.md) — only the
`category` code and the product-specific `notes` below differ. Contacts,
services, and top-level fields follow the ticket model.

## Operations

| Operation | `category` | Purpose |
| --- | --- | --- |
| Equipment RMA | `125` | Request a replacement device (RMA) shipped to a specified address. |
| Return Device | `058` | Return an SD-WAN device to MetTel. |
| Service Affecting Trouble | `VAS` | Report a service-affecting (degraded, not down) SD-WAN trouble. |
| Service Outage Trouble | `VOO` | Report a full SD-WAN service outage. |
| Update Management Status | `JJJ` | Update the management status of an SD-WAN device. |

## Equipment RMA — `125`

Request a replacement device (RMA) shipped to a specified address. Subcategory IDs: `96`, `343`.

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

- **`AttentionTo`** **(Required)** — Attention To. Name of the person the shipment should be addressed to. Accepts free-text input.
- **`RefRepairTicket`** **(Optional)** — Reference Repair Ticket Number. Ticket number of a related repair ticket if applicable. Accepts a dropdown selection.
- **`ShippingAddress`** **(Required)** — Shipping Address. Address where the replacement device should be shipped. Accepts a structured address.
- **`Warranty Reasons`** **(Required)** — Return/RMA Reason. Allowed values:
  - `1` = Device Not Charging
  - `4` = Screen Not Working
  - `5` = Other
  - `6` = Damaged In Transit
  - `7` = Defective Device
  - `8` = Incorrect Address
  - `10` = Missing Component
  - `11` = Lost/Stolen
  - `12` = Closed Location

## Return Device — `058`

Return an SD-WAN device to MetTel. Subcategory IDs: `96`, `343`.

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

- **`CPON`** **(Optional)** — Conexus Purchase Order Number. Conexus purchase order number if applicable. Accepts free-text input.
- **`Warranty Reasons`** **(Required)** — Return/RMA Reason. Allowed values:
  - `1` = Device Not Charging
  - `4` = Screen Not Working
  - `5` = Other
  - `6` = Damaged In Transit
  - `7` = Defective Device
  - `8` = Incorrect Address
  - `10` = Missing Component
  - `11` = Lost/Stolen
  - `12` = Closed Location

## Service Affecting Trouble — `VAS`

Report a service-affecting (degraded, not down) SD-WAN trouble. Subcategory IDs: `96`, `343`.

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
            "noteType": "descsdwanissue",
            "noteValue": "1"
        },
        {
            "noteType": "sdwanhaspower",
            "noteValue": "Yes"
        },
        {
            "noteType": "permissiontoreboot",
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

- **`descsdwanissue`** **(Required)** — Description of SD-WAN Issue. Allowed values:
  - value `"1"` (SD-WAN hardware issue) →
    - **`sdwanhaspower`** **(Conditional — Required)** — SD-WAN Has Power. Indicate whether the SD-WAN device currently has power. Accepts free-text input.
    - **`permissiontoreboot`** **(Conditional — Required)** — Permission to Reboot. Indicate whether MetTel has permission to reboot the SD-WAN device. Accepts free-text input.
  - value `"2"` (SD-WAN traffic issue) →
    - **`desctrafficissue`** **(Conditional — Required)** — Description of Traffic Issue. Describe the nature of the traffic issue being experienced. Accepts free-text input.
    - **`permissiontofailover`** **(Conditional — Required)** — Permission to Fail Over. Indicate whether MetTel has permission to fail over the SD-WAN traffic to an alternate path. Accepts free-text input.
    - **`permissiontoreboot`** **(Conditional — Required)** — Permission to Reboot. Indicate whether MetTel has permission to reboot the SD-WAN device. Accepts free-text input.

## Service Outage Trouble — `VOO`

Report a full SD-WAN service outage. Subcategory IDs: `96`, `343`.

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
            "noteType": "descsdwanissue",
            "noteValue": "1"
        },
        {
            "noteType": "sdwanhaspower",
            "noteValue": "Yes"
        },
        {
            "noteType": "permissiontoreboot",
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

- **`descsdwanissue`** **(Required)** — Description of SD-WAN Issue. Allowed values:
  - value `"1"` (SD-WAN hardware issue) →
    - **`sdwanhaspower`** **(Conditional — Required)** — SD-WAN Has Power. Indicate whether the SD-WAN device currently has power. Accepts free-text input.
    - **`permissiontoreboot`** **(Conditional — Required)** — Permission to Reboot. Indicate whether MetTel has permission to reboot the SD-WAN device. Accepts free-text input.
  - value `"2"` (SD-WAN traffic issue) →
    - **`desctrafficissue`** **(Conditional — Required)** — Description of Traffic Issue. Describe the nature of the traffic issue being experienced. Accepts free-text input.
    - **`permissiontofailover`** **(Conditional — Required)** — Permission to Fail Over. Indicate whether MetTel has permission to fail over the SD-WAN traffic to an alternate path. Accepts free-text input.
    - **`permissiontoreboot`** **(Conditional — Required)** — Permission to Reboot. Indicate whether MetTel has permission to reboot the SD-WAN device. Accepts free-text input.

## Update Management Status — `JJJ`

Update the management status of an SD-WAN device. Subcategory IDs: `96`, `343`.

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
            "noteValue": "FTX1234ABCD"
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

- **`SerialNumber`** **(Required)** — Serial Number. Serial number of the SD-WAN device. Accepts free-text input.
- **`ManagementStatus`** **(Required)** — Management Status. Management status to apply to the device. Accepts a dropdown selection (allowed values not enumerated in source docs).

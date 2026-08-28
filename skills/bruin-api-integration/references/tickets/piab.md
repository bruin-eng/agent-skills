# PIAB (Phone-in-a-Box) — Ticket Reference

PIAB ticket operations, split into **Device** operations and **Lines**
operations. Every ticket is a `POST /api/Ticket` using the universal body in
[../ticket-model.md](../ticket-model.md) — only the `category` code and the
product-specific `notes` below differ. Contacts, services, and top-level fields
follow the ticket model.

## Operations

| Group | Operation | `category` | Purpose |
| --- | --- | --- | --- |
| Device | Equipment RMA | `125` | Request an RMA/replacement for a PIAB device. |
| Device | Return Device | `058` | Return a PIAB device. |
| Device | Service Affecting Trouble | `VAS` | Report service-affecting (degraded) trouble on a PIAB device. |
| Device | Service Outage Trouble | `VOO` | Report a full service outage on a PIAB device. |
| Device | Update Management Status | `JJJ` | Update the management status of a PIAB device. |
| Lines | Add Additional VoIP Line | `141` | Add an additional VoIP line to a PIAB service. |
| Lines | Add Change Delete Feature | `025` | Add, change, or delete a feature on a PIAB line. |
| Lines | Change CNAM on PIAB Line | `160` | Change the Caller ID Name (CNAM)/listed name on a line. |
| Lines | Change PIAB Line Type | `143` | Change the line type of an existing PIAB line. |
| Lines | Change PIAB Phone Number | `144` | Change the phone number assigned to a PIAB line. |
| Lines | Disconnect | `020` | Disconnect a PIAB line. |
| Lines | Loss of Line Investigation | `061` | Investigate a lost/ported-out line. |
| Lines | Service Affecting Trouble | `VAS` | Report service-affecting (degraded) voice trouble on a line. |
| Lines | Service Outage Trouble | `VOO` | Report a full voice outage on a line. |
| Lines | Suspend | `SUS` | Suspend a PIAB line. |
| Lines | Unsuspend Service | `022` | Unsuspend a previously suspended PIAB line. |

Note: `MTK` (problem description) and `DDD` (desired due date, `MM/DD/YYYY`) are
universal notes carried on every ticket per the ticket model; they are shown in
each example body but are not repeated as product-specific notes below.

# Device

PIAB Device operations use the **POTS in a Box** product. Subcategory IDs: `231`, `364`.

## Equipment RMA — `125`

Request an RMA/replacement for a PIAB device. Subcategory IDs: `231`, `364`.

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

- **`AttentionTo`** **(Required)** — Attention To; name the shipment should be addressed to. Free-text.
- **`RefRepairTicket`** **(Optional)** — Reference Repair Ticket Number; ticket number of a related repair ticket. Dropdown selection.
- **`ShippingAddress`** **(Required)** — Shipping Address for the replacement device. Structured address.
- **`Warranty Reasons`** **(Required)** — Return/RMA Reason. Allowed values:
  - `1` Device Not Charging
  - `4` Screen Not Working
  - `5` Other
  - `6` Damaged In Transit
  - `7` Defective Device
  - `8` Incorrect Address
  - `10` Missing Component
  - `11` Lost/Stolen
  - `12` Closed Location

## Return Device — `058`

Return a PIAB device. Subcategory IDs: `231`, `364`.

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
            "noteType": "IMEI",
            "noteValue": "123456789012345"
        },
        {
            "noteType": "RequestDeinstall",
            "noteValue": "false"
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

- **`CPON`** **(Optional)** — Conexus Purchase Order Number, if applicable. Free-text.
- **`IMEI`** **(Required)** — IMEI number of the device being returned. Free-text.
- **`RequestDeinstall`** **(Required)** — Request IW/Deinstallation.
  - `true` — Request inside wiring deinstallation.
  - `false` — Do not request inside wiring deinstallation.
- **`Warranty Reasons`** **(Required)** — Return/RMA Reason. Allowed values:
  - `1` Device Not Charging
  - `4` Screen Not Working
  - `5` Other
  - `6` Damaged In Transit
  - `7` Defective Device
  - `8` Incorrect Address
  - `10` Missing Component
  - `11` Lost/Stolen
  - `12` Closed Location

## Service Affecting Trouble — `VAS`

Report service-affecting (degraded) trouble on a PIAB device. Subcategory IDs: `231`, `364`.

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
            "noteType": "Require ISW",
            "noteValue": "B"
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

- **`Require ISW`** **(Required)** — Require Inside Wiring.
  - `A` — Yes, require inside wiring.
  - `B` — No, do not require inside wiring.

## Service Outage Trouble — `VOO`

Report a full service outage on a PIAB device. Subcategory IDs: `231`, `364`.

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
            "noteType": "Require ISW",
            "noteValue": "B"
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

- **`Require ISW`** **(Required)** — Require Inside Wiring.
  - `A` — Yes, require inside wiring.
  - `B` — No, do not require inside wiring.

## Update Management Status — `JJJ`

Update the management status of a PIAB device. Subcategory IDs: `231`, `364`.

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
            "noteType": "IMEI",
            "noteValue": "123456789012345"
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

- **`IMEI`** **(Required)** — The device's IMEI number. Free-text.
- **`ManagementStatus`** **(Required)** — Management status to apply to the device. Dropdown selection.

# Lines

PIAB Lines operations use the **Specialty Business Line** product. Subcategory ID: `238`.

## Add Additional VoIP Line — `141`

Add an additional VoIP line to a PIAB service. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "141",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "AvailablePIABPort",
            "noteValue": "1"
        },
        {
            "noteType": "DirectoryListing",
            "noteValue": "TRUE"
        },
        {
            "noteType": "LNAME",
            "noteValue": "SMITH JOHN"
        },
        {
            "noteType": "SIPLine1",
            "noteValue": "231"
        },
        {
            "noteType": "PhoneNumberType",
            "noteValue": "New Phone Number"
        },
        {
            "noteType": "APINPANXX",
            "noteValue": "305847"
        },
        {
            "noteType": "SecondaryE911",
            "noteValue": "123 Main St, Springfield, IL 62701"
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

- **`AvailablePIABPort`** **(Required)** — Available Port. Valid values are fetched at runtime from the Bruin API template field `BruinTemplateField_AvailablePIABPorts`.
- **`DirectoryListing`** **(Required)** — Directory Listing.
  - `FALSE` — Do not add this line to the directory listing.
  - `TRUE` — Add this line to the directory listing.
    - **`LNAME`** **(Conditional — Required)** — Listed Name; max 15 characters, also used for emergency registration. Free-text.
- **`SIPLine1`** **(Required)** — PIAB Line Type. Allowed values:
  - `231` Voice
  - `245` Fire Alarm
  - `246` Burglar Alarm
  - `247` Modem
  - `248` Elevator
  - `250` Fax
  - `251` Elevator Modem
- **`PhoneNumberType`** **(Required)** — Phone Number Type. Required for all PIAB line types.
  - `New Phone Number` — Request a new phone number for this line.
    - **`APINPANXX`** **(Conditional — Required)** — Desired Area Code; preferred area code and exchange (NPANXX). Free-text.
  - `Port Existing Phone Number` — Port an existing phone number to this line.
    - **`btnofports`** **(Conditional — Required)** — Number of Ports; number of phone numbers being ported. Free-text.
    - **`DIDnums`** **(Conditional — Required)** — DID Numbers to be ported. Free-text.
  - `Use Existing Enterprise Phone Number` — Use a phone number already provisioned within your enterprise.
    - **`ExistEntPhoneNumber`** **(Conditional — Required)** — Existing Enterprise Phone Number to assign to this line. Free-text.
- **`SecondaryE911`** **(Optional)** — Secondary E911. Free-text.

## Add Change Delete Feature — `025`

Add, change, or delete a feature on a PIAB line. Subcategory ID: `238`.

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
            "noteType": "Feature Request Type",
            "noteValue": "Account Codes"
        },
        {
            "noteType": "acctcodedetails",
            "noteValue": "ACCOUNT CODE DETAILS HERE"
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

- **`Feature Request Type`** **(Required)** — Feature Request Type. Each value below triggers its own additional note(s):
  - `Account Codes` — Add, change, or delete account codes.
    - **`acctcodedetails`** **(Conditional — Required)** — Account Code Details; describe the account code changes. Free-text.
  - `CNAM` — Add, change, or delete Caller ID Name (CNAM).
    - **`cnamchange`** **(Conditional — Optional)** — CNAM Change; desired CNAM value or change description. Free-text.
  - `Changing DNIS Digits` — Change the number of DNIS digits.
    - **`numdnisdigits`** **(Conditional — Required)** — Number of DNIS Digits desired. Free-text.
  - `Circuit Trunk Group Configuration` — Modify circuit trunk group configuration.
    - **`fasnfas`** **(Conditional — Required)** — FAS/Non-FAS Configuration details. Free-text.
  - `Convert to RCF` — Convert this line to Remote Call Forwarding. (No additional note.)
  - `DTO/Trunk Group Overflow` — Add, change, or delete DTO or trunk group overflow settings.
    - **`newdtoorchange`** **(Conditional — Required)** — New DTO or Change; whether this is a new DTO or a change to an existing one. Free-text.
    - **`dtopointonum`** **(Conditional — Required)** — DTO Point-To Number; where overflow calls should be directed. Free-text.
  - `Directory Listing` — Add, change, or delete a directory listing.
    - **`dirlistingtype`** **(Conditional — Required)** — Directory Listing Type (add, update, remove). Free-text.
  - `E911` — Add, change, or delete E911 registration information.
    - **`e911change`** **(Conditional — Required)** — E911 Change; describe the E911 address or configuration change. Free-text.
  - `International Dialing Enable or Disable` — Enable or disable international dialing.
    - **`enableordisable`** **(Conditional — Required)** — Enable or Disable international dialing. Free-text.
  - `Move Services` — Move services to a new address.
    - **`NewAddress`** **(Conditional — Required)** — New Address; full new service address. Free-text.
  - `New DIDs` — Add new Direct Inward Dialing numbers.
    - **`NDID`** **(Conditional — Required)** — New DID Numbers to be added. Free-text.
    - **`diffnpanxx`** **(Conditional — Required)** — Different NPANXX; desired area code and exchange if different from existing number. Free-text.
  - `Other` — A feature change not listed above.
    - **`RequestDescription`** **(Conditional — Required)** — Request Description; describe the feature change. Free-text.
  - `PBX control to out-pulse individual DIDs` — Enable PBX control to out-pulse individual DID numbers.
    - **`yesorno`** **(Conditional — Required)** — Yes or No; confirm whether to enable PBX out-pulsing. Free-text.
  - `Port Request` — Port phone numbers to or from this line.
    - **`btnofports`** **(Conditional — Required)** — Number of Ports; total number of phone numbers being ported. Free-text.
    - **`portlist`** **(Conditional — Required)** — Port List; the phone numbers to be ported. Free-text.
    - **`dirlistingtype`** **(Conditional — Required)** — Directory Listing Type for the ported numbers. Free-text.
    - **`cnamdetails`** **(Conditional — Required)** — CNAM Details; Caller ID Name for the ported numbers. Free-text.
  - `Request for Activation` — Activate a circuit or feature on this line.
    - **`mettelcircuitbilling`** **(Conditional — Required)** — MetTel Circuit Billing information for the activation. Free-text.

## Change CNAM on PIAB Line — `160`

Change the Caller ID Name (CNAM)/listed name on a line. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "160",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "LNAME",
            "noteValue": "SMITH JOHN"
        },
        {
            "noteType": "SecondaryE911",
            "noteValue": "123 Main St, Springfield, IL 62701"
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

- **`LNAME`** **(Required)** — Listed Name; max 15 characters, also used for emergency registration. Free-text.
- **`SecondaryE911`** **(Optional)** — Secondary E911. Free-text.

## Change PIAB Line Type — `143`

Change the line type of an existing PIAB line. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "143",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "NewLineType",
            "noteValue": "231"
        },
        {
            "noteType": "LNAME",
            "noteValue": "SMITH JOHN"
        },
        {
            "noteType": "SecondaryE911",
            "noteValue": "123 Main St, Springfield, IL 62701"
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

- **`LNAME`** **(Optional)** — Listed Name; max 15 characters, also used for emergency registration. Free-text.
- **`NewLineType`** **(Required)** — New Line Type. Allowed values:
  - `231` Voice
  - `245` Fire Alarm
  - `246` Burglar Alarm
  - `247` Modem
  - `248` Elevator
  - `250` Fax
  - `251` Elevator Modem
- **`SecondaryE911`** **(Optional)** — Secondary E911. Free-text.

## Change PIAB Phone Number — `144`

Change the phone number assigned to a PIAB line. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "144",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "DirectoryListing",
            "noteValue": "TRUE"
        },
        {
            "noteType": "LNAME",
            "noteValue": "SMITH JOHN"
        },
        {
            "noteType": "PhoneNumberType",
            "noteValue": "New Phone Number"
        },
        {
            "noteType": "APINPANXX",
            "noteValue": "305847"
        },
        {
            "noteType": "SecondaryE911",
            "noteValue": "123 Main St, Springfield, IL 62701"
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

- **`DirectoryListing`** **(Required)** — Directory Listing.
  - `FALSE` — Do not add this line to the directory listing.
  - `TRUE` — Add this line to the directory listing.
    - **`LNAME`** **(Conditional — Required)** — Listed Name; max 15 characters, also used for emergency registration. Free-text.
- **`PhoneNumberType`** **(Required)** — Phone Number Type.
  - `New Phone Number` — Request a new phone number for this line.
    - **`APINPANXX`** **(Conditional — Required)** — Desired Area Code; preferred area code and exchange (NPANXX). Free-text.
  - `Port Existing Phone Number` — Port an existing phone number to this line.
    - **`btnofports`** **(Conditional — Required)** — Number of Ports; number of phone numbers being ported. Free-text.
    - **`DIDnums`** **(Conditional — Required)** — DID Numbers to be ported. Free-text.
  - `Use Existing Enterprise Phone Number` — Use a phone number already provisioned within your enterprise.
    - **`ExistEntPhoneNumber`** **(Conditional — Required)** — Existing Enterprise Phone Number to assign to this line. Free-text.
- **`SecondaryE911`** **(Optional)** — Secondary E911. Free-text.

## Disconnect — `020`

Disconnect a PIAB line. Subcategory ID: `238`.

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
            "noteType": "VOIPDisconnect",
            "noteValue": "Disconnect"
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

- **`DRN`** **(Required)** — Disconnect Reason.
  - `1` — Downsizing Services at Location.
  - `2` — Closing Location.
  - `3` — Technology Upgrade.
  - `4` — Moving to New Location.
    - **`moveserviceoption`** **(Conditional — Required)** — Move Service Option; what should happen to the service at the new location. Free-text.
  - `5` — Non-Payment.
  - `6` — Other.
    - **`descother`** **(Conditional — Required)** — Description of the disconnect reason. Free-text.
- **`VOIPDisconnect`** **(Required)** — VOIP Disconnect.
  - `Disconnect` — Fully disconnect the line.
  - `Intercept` — Route the line to an intercept message.
    - **`PointTo`** **(Conditional — Required)** — Point To; the intercept message destination. Free-text.
  - `RCF` — Forward the line via SIP Remote Call Forwarding.
    - **`PointTo`** **(Conditional — Required)** — Point To; the forwarding destination number. Free-text.
- **`discauthdoc`** **(Optional)** — Disconnection Authorization Paperwork; upload signed authorization paperwork if applicable. File upload attachment.

## Loss of Line Investigation — `061`

Investigate a lost/ported-out line. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "061",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "portoutdate",
            "noteValue": "05/15/2026"
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

- **`portoutdate`** **(Required)** — Port Out Date. Date value (`MM/DD/YYYY`).

## Service Affecting Trouble — `VAS`

Report service-affecting (degraded) voice trouble on a line. Subcategory ID: `238`.

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
            "noteType": "mplsvoiceproblem",
            "noteValue": "1"
        },
        {
            "noteType": "FTN",
            "noteValue": "3059473030"
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

- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? Selecting `true` agrees to associated fees (invoiced at contracted ISW rates).
  - `true` — Request a MetTel field technician dispatch.
    - **`HoursTechAssist`** **(Conditional — Required)** — Hours of Tech Assist requested. Free-text.
    - **`TechHours`** **(Conditional — Required)** — Tech Hours needed. Free-text.
    - **`timetechdispatch`** **(Conditional — Required)** — Time of Tech Dispatch requested. Free-text.
    - **`techsow`** **(Conditional — Required)** — Tech Scope of Work. Free-text.
  - `false` — Do not request a field technician dispatch.
- **`mplsvoiceproblem`** **(Required)** — Nature of the Voice Problem.
  - `1` — Circuit down; no inbound or outbound calls.
    - **`FTN`** **(Conditional — Required)** — Failing Telephone Number experiencing the outage. Free-text.
  - `2` — Cannot receive inbound calls.
    - **`FTN`** **(Conditional — Required)** — Failing Telephone Number experiencing the issue. Free-text.
    - **`Call Example`** **(Conditional — Required)** — Example of a call that failed to connect. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.
  - `3` — Cannot make outbound calls.
    - **`Call Example`** **(Conditional — Required)** — Example of a call that failed to connect. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.
  - `4` — Dropped calls.
    - **`callexamfrom`** **(Conditional — Required)** — Call Example From; originating number of a dropped call. Free-text.
    - **`callexamto`** **(Conditional — Required)** — Call Example To; destination number of a dropped call. Free-text.
    - **`callexamdate`** **(Conditional — Required)** — Call Example Date the dropped call occurred. Date value (`MM/DD/YYYY`).
    - **`callexamtime`** **(Conditional — Required)** — Call Example Time the dropped call occurred. Free-text.
    - **`Time Zone`** **(Conditional — Required)** — Time Zone for the call example. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.
  - `5` — Static or choppy calls.
    - **`callexamfrom`** **(Conditional — Required)** — Call Example From; originating number of an affected call. Free-text.
    - **`callexamto`** **(Conditional — Required)** — Call Example To; destination number of an affected call. Free-text.
    - **`callexamdate`** **(Conditional — Required)** — Call Example Date the affected call occurred. Date value (`MM/DD/YYYY`).
    - **`callexamtime`** **(Conditional — Required)** — Call Example Time the affected call occurred. Free-text.
    - **`Time Zone`** **(Conditional — Required)** — Time Zone for the call example. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.

## Service Outage Trouble — `VOO`

Report a full voice outage on a line. Subcategory ID: `238`.

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
            "noteType": "mplsvoiceproblem",
            "noteValue": "1"
        },
        {
            "noteType": "FTN",
            "noteValue": "3059473030"
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

- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? Selecting `true` agrees to associated fees (invoiced at contracted ISW rates).
  - `true` — Request a MetTel field technician dispatch.
    - **`HoursTechAssist`** **(Conditional — Required)** — Hours of Tech Assist requested. Free-text.
    - **`TechHours`** **(Conditional — Required)** — Tech Hours needed. Free-text.
    - **`timetechdispatch`** **(Conditional — Required)** — Time of Tech Dispatch requested. Free-text.
    - **`techsow`** **(Conditional — Required)** — Tech Scope of Work. Free-text.
  - `false` — Do not request a field technician dispatch.
- **`mplsvoiceproblem`** **(Required)** — Nature of the Voice Problem.
  - `1` — Circuit down; no inbound or outbound calls.
    - **`FTN`** **(Conditional — Required)** — Failing Telephone Number experiencing the outage. Free-text.
  - `2` — Cannot receive inbound calls.
    - **`FTN`** **(Conditional — Required)** — Failing Telephone Number experiencing the issue. Free-text.
    - **`Call Example`** **(Conditional — Required)** — Example of a call that failed to connect. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.
  - `3` — Cannot make outbound calls.
    - **`Call Example`** **(Conditional — Required)** — Example of a call that failed to connect. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.
  - `4` — Dropped calls.
    - **`callexamfrom`** **(Conditional — Required)** — Call Example From; originating number of a dropped call. Free-text.
    - **`callexamto`** **(Conditional — Required)** — Call Example To; destination number of a dropped call. Free-text.
    - **`callexamdate`** **(Conditional — Required)** — Call Example Date the dropped call occurred. Date value (`MM/DD/YYYY`).
    - **`callexamtime`** **(Conditional — Required)** — Call Example Time the dropped call occurred. Free-text.
    - **`Time Zone`** **(Conditional — Required)** — Time Zone for the call example. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.
  - `5` — Static or choppy calls.
    - **`callexamfrom`** **(Conditional — Required)** — Call Example From; originating number of an affected call. Free-text.
    - **`callexamto`** **(Conditional — Required)** — Call Example To; destination number of an affected call. Free-text.
    - **`callexamdate`** **(Conditional — Required)** — Call Example Date the affected call occurred. Date value (`MM/DD/YYYY`).
    - **`callexamtime`** **(Conditional — Required)** — Call Example Time the affected call occurred. Free-text.
    - **`Time Zone`** **(Conditional — Required)** — Time Zone for the call example. Free-text.
    - **`intrusive testing`** **(Conditional — Optional)** — Whether intrusive testing is permitted. Free-text.

## Suspend — `SUS`

Suspend a PIAB line. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "SUS",
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

No category-specific note types. Submit with only the universal `MTK` and `DDD` notes.

## Unsuspend Service — `022`

Unsuspend a previously suspended PIAB line. Subcategory ID: `238`.

### Example body

```json
{
    "clientId": 9994,
    "category": "022",
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

No category-specific note types. Submit with only the universal `MTK` and `DDD` notes.

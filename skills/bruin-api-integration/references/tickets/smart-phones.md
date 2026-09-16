# Smart Phones — Ticket Reference

Wireless / smart phone ticket operations. Every ticket is a `POST /api/Ticket`
using the universal body in [../ticket-model.md](../ticket-model.md) — only the
`category` code and the product-specific `notes` below differ. Contacts,
services, and top-level fields follow the ticket model.

## Operations

| Operation | `category` | Purpose |
| --- | --- | --- |
| Add/Return Depot Inventory | `RRR` | Return a device to depot inventory, optionally keeping the service active. |
| CallerID Update | `888` | Update the outbound CallerID name for a line. |
| Change Mobile Number | `004` | Request a new number or restore a previous number for a line. |
| Device Trade-In | `PP2` | Trade in / recycle a device for a user. |
| Disconnect | `020` | Disconnect a wireless line of service. |
| MDM Enroll/Unenroll | `086` | Enroll or un-enroll a device in mobile device management. |
| MDM Support | `016` | Request MDM support for a managed device by platform. |
| Return Device | `058` | Return a device, optionally disconnecting the line. |
| Suspend | `SUS` | Suspend a wireless line of service. |
| Swap Service | `11F` | Swap/port a service line onto a new account. |
| Unsuspend Service | `022` | Restore a suspended wireless line. |
| Voicemail Box Reset | `013` | Reset the voicemail box for a line. |
| Voicemail Password Reset | `014` | Reset the voicemail password for a line. |
| Warranty/Protect Replacement | `008` | Ship a warranty/protection replacement device. |
| Wireless Device Not Working | `017` | Troubleshoot a non-working wireless device. |
| Wireless Service Not Working | `019` | Troubleshoot non-working wireless service. |

## Add/Return Depot Inventory — `RRR`

Return a device to depot inventory, optionally keeping the service active. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "RRR",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "DeviceAddress",
            "noteValue": "123 Main St, Springfield, IL 62701"
        },
        {
            "noteType": "DeviceReturnOnly",
            "noteValue": "true"
        },
        {
            "noteType": "IMEI",
            "noteValue": "352999001234567"
        },
        {
            "noteType": "ReturnKIT",
            "noteValue": "false"
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

- **`DeviceAddress`** **(Required)** — Device from address. Accepts a structured address.
- **`DeviceReturnOnly`** **(Required)** — Keep service active. `"true"` = keep service active, return device only; `"false"` = disconnect the service along with the device return.
- **`IMEI`** **(Required)** — Device IMEI. Free-text input.
- **`ReturnKIT`** **(Required)** — Purchase return kit? `"true"` = purchase a return kit; `"false"` = do not purchase.

## CallerID Update — `888`

Update the outbound CallerID name for a line. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "888",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "ChangeCallerID",
            "noteValue": "John Smith"
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

- **`ChangeCallerID`** **(Required)** — Change CallerID to. Accepts free-text input.

## Change Mobile Number — `004`

Request a new number or restore a previous number for a line. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "004",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "PhoneNumberChange",
            "noteValue": "Request New Number"
        },
        {
            "noteType": "DSNPANXX",
            "noteValue": "305847"
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

- **`Assignee`** **(Optional)** — Who the line is assigned to.
  - value `"Hot Spare"` (assign line to an unassigned spare device) → **`LDSCR`** **(Conditional — Optional)** — Line Description; descriptive label for this line. Free-text input.
  - value `"Other"` (line not assigned to a specific person) → **`LDSCR`** **(Conditional — Required)** — Line Description; descriptive label for this line. Free-text input.
  - value `"Person"` (assign line to a specific user) → **`User`** **(Conditional — Required)** — Bruin user to assign this line to. Accepts a Bruin user selection.
  - value `"Person"` → **`LDSCR`** **(Conditional — Optional)** — Line Description; descriptive label for this line. Free-text input.
- **`PhoneNumberChange`** **(Required)** — Phone number change.
  - value `"Request New Number"` (request a brand new number) → **`DSNPANXX`** **(Conditional — Required)** — Desired Area Code; preferred area code and exchange (NPANXX). Free-text input.
  - value `"Restore Previous Number"` (restore a previously held number) → **`NumbertoRestore`** **(Conditional — Required)** — Number to Restore; the phone number to restore. Free-text input.

## Device Trade-In — `PP2`

Trade in / recycle a device for a user. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "PP2",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "RecycleDevice",
            "noteValue": "true"
        },
        {
            "noteType": "User",
            "noteValue": "jsmith"
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

- **`RecycleDevice`** **(Required)** — Recycling your device? `"true"` = recycle the device; `"false"` = do not recycle.
- **`User`** **(Required)** — User. Accepts a Bruin user selection.

## Disconnect — `020`

Disconnect a wireless line of service. Subcategory ID: `1`.

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
            "noteType": "DeviceDecisionId",
            "noteValue": "1"
        },
        {
            "noteType": "DisconnectPending",
            "noteValue": "false"
        },
        {
            "noteType": "WL Disconnect Reason",
            "noteValue": "C"
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

- **`DeviceDecisionId`** **(Required)** — Existing device decision. Valid values fetched at runtime from Bruin API template field `BruinTemplateField_DeviceDecisions`.
- **`DisconnectPending`** **(Required)** — Disconnect pending port out? `"true"` = pending a port-out; `"false"` = not pending a port-out.
- **`WL Disconnect Reason`** **(Required)** — Wireless disconnect reason:
  - `"B"` = 1. Customer is Cutting Back
  - `"C"` = 2. Employee Left Company
  - `"D"` = 3. Phone Returned
  - `"E"` = 4. Port-Out
  - `"F"` = 5. Lost/Stolen
  - `"G"` = 6. Billing Issue
  - `"H"` = 7. Support Issue
  - `"I"` = 8. Move to Competition - Other
  - `"J"` = 9. No Install/Equip Never Delivered
  - `"K"` = 10. Customer Request Cancel - No Reason
  - `"L"` = 11. Zero Usage

## MDM Enroll/Unenroll — `086`

Enroll or un-enroll a device in mobile device management. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "086",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "DEPOrderType",
            "noteValue": "Enrollment"
        },
        {
            "noteType": "MDMDeviceIdentifier",
            "noteValue": "IMEI"
        },
        {
            "noteType": "IMEI",
            "noteValue": "352999001234567"
        },
        {
            "noteType": "MDMType",
            "noteValue": "Apple DEP"
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

- **`DEPOrderType`** **(Required)** — MDM enrollment type. `"Enrollment"` = enroll a device into MDM; `"Un-enrollment"` = remove a device from MDM.
- **`MDMDeviceIdentifier`** **(Required)** — Identify device by IMEI or Serial Number.
  - value `"IMEI"` (identify by IMEI) → **`IMEI`** **(Conditional — Required)** — IMEI of the device to enroll/unenroll.
  - value `"Serial Number"` (identify by serial number) → **`SerialNumber`** **(Conditional — Required)** — Serial Number of the device to enroll/unenroll.
- **`MDMType`** **(Required)** — MDM vendor. `"Android Zero-Touch"`, `"Apple DEP"` (Apple Device Enrollment Program), or `"Samsung Knox"`.

## MDM Support — `016`

Request MDM support for a managed device by platform. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "016",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "MDMPlatform",
            "noteValue": "AWM"
        },
        {
            "noteType": "Type",
            "noteValue": "Device Enrollment"
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

- **`MDMPlatform`** **(Required)** — MDM platform.
  - value `"AWM"` (AirWatch) → **`Type`** **(Conditional — Required)** — Type of MDM support request. Free-text input.
  - value `"Checkpoint Harmony Mobile"` → **`CheckpointHarmonyReq`** **(Conditional — Required)** — Checkpoint Harmony Mobile support request. Free-text input.
  - value `"Intune"` (Microsoft Intune) → **`Type`** **(Conditional — Required)** — Type of MDM support request. Free-text input.
  - value `"Ivanti"` → **`MDMRequestType`** **(Conditional — Required)** — Type of Ivanti MDM support request. Free-text input.
  - value `"Other"` (platform not listed) → **`OtherMDMPlatform`** **(Conditional — Required)** — Name of the MDM platform being used. Free-text input.
  - value `"Other"` → **`Type`** **(Conditional — Required)** — Type of MDM support request. Free-text input.
  - value `"SOTI"` (SOTI MobiControl) → **`Type`** **(Conditional — Required)** — Type of MDM support request. Free-text input.
  - value `"Scalefusion"` — no additional note.

## Return Device — `058`

Return a device, optionally disconnecting the line. Subcategory ID: `1`.

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
            "noteValue": "352999001234567"
        },
        {
            "noteType": "LineOfServDisco",
            "noteValue": "FALSE"
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

- **`IMEI`** **(Required)** — Device IMEI. Free-text input.
- **`LineOfServDisco`** **(Required)** — Disconnect service? `"FALSE"` = return device only; `"TRUE"` = return device and disconnect the associated line of service.
- **`Warranty Reasons`** **(Required)** — Return/RMA reason:
  - `"1"` = Device Not Charging
  - `"4"` = Screen Not Working
  - `"5"` = Other
  - `"6"` = Damaged In Transit
  - `"7"` = Defective Device
  - `"8"` = Incorrect Address
  - `"10"` = Missing Component
  - `"11"` = Lost/Stolen
  - `"12"` = Closed Location

## Suspend — `SUS`

Suspend a wireless line of service. Subcategory ID: `1`.

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

This ticket type requires no category-specific note types. Submit with only the universal `MTK` and `DDD` notes.

## Swap Service — `11F`

Swap/port a service line onto a new account. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "11F",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "IMEI",
            "noteValue": "352999001234567"
        },
        {
            "noteType": "OSPAccount",
            "noteValue": "ACC-9876543"
        },
        {
            "noteType": "PORTLINES",
            "noteValue": "3059473030"
        },
        {
            "noteType": "User",
            "noteValue": "jsmith"
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

- **`IMEI`** **(Required)** — Device IMEI. Free-text input.
- **`OSPAccount`** **(Required)** — Old account number. Valid values fetched at runtime from Bruin API template field `BruinTemplateField_OSPAccounts`.
- **`PORTLINES`** **(Required)** — Existing phone number. Free-text input.
- **`User`** **(Required)** — User. Accepts a Bruin user selection.

## Unsuspend Service — `022`

Restore a suspended wireless line. Subcategory ID: `1`.

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
            "noteType": "User",
            "noteValue": "jsmith"
        },
        {
            "noteType": "VoicemailBoxReset",
            "noteValue": "false"
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

- **`User`** **(Required)** — User. Accepts a Bruin user selection.
- **`VoicemailBoxReset`** **(Required)** — Reset voicemail box? `"true"` = reset the voicemail box; `"false"` = do not reset.

## Voicemail Box Reset — `013`

Reset the voicemail box for a line. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "013",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "VoicemailPassword",
            "noteValue": "NewP@ssw0rd!"
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

- **`VoicemailPassword`** **(Optional)** — Voicemail password. Free-text input.

## Voicemail Password Reset — `014`

Reset the voicemail password for a line. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "014",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "VoicemailPassword",
            "noteValue": "NewP@ssw0rd!"
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

- **`VoicemailPassword`** **(Optional)** — Voicemail password. Free-text input.

## Warranty/Protect Replacement — `008`

Ship a warranty/protection replacement device. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "008",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "AttentionTo",
            "noteValue": "John Smith"
        },
        {
            "noteType": "IMEI",
            "noteValue": "352999001234567"
        },
        {
            "noteType": "ShippingAddress",
            "noteValue": "123 Main St, Springfield, IL 62701"
        },
        {
            "noteType": "ShippingMethod",
            "noteValue": "XXX01"
        },
        {
            "noteType": "User",
            "noteValue": "jsmith"
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

- **`AttentionTo`** **(Required)** — Attention to. Free-text input.
- **`IMEI`** **(Required)** — Device IMEI. Free-text input.
- **`ShippingAddress`** **(Required)** — Shipping address. Accepts a structured address.
- **`ShippingMethod`** **(Required)** — Shipping method. `"XXX01"` = Next Day Air; `"XXX02"` = 2nd Day Air; `"XXX03"` = Ground.
- **`User`** **(Required)** — User. Accepts a Bruin user selection.
- **`Warranty Reasons`** **(Required)** — Return/RMA reason:
  - `"1"` = Device Not Charging
  - `"4"` = Screen Not Working
  - `"5"` = Other
  - `"6"` = Damaged In Transit
  - `"7"` = Defective Device
  - `"8"` = Incorrect Address
  - `"10"` = Missing Component
  - `"11"` = Lost/Stolen
  - `"12"` = Closed Location

## Wireless Device Not Working — `017`

Troubleshoot a non-working wireless device. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "017",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "ICCID",
            "noteValue": "8901260123456789012"
        },
        {
            "noteType": "IMEI",
            "noteValue": "352999001234567"
        },
        {
            "noteType": "IssueStartDate",
            "noteValue": "05/01/2026"
        },
        {
            "noteType": "IsthisBYOD",
            "noteValue": "No"
        },
        {
            "noteType": "NetworkSettingReset",
            "noteValue": "Yes"
        },
        {
            "noteType": "RecentlyPorted",
            "noteValue": "No"
        },
        {
            "noteType": "SIMReseat",
            "noteValue": "Yes"
        },
        {
            "noteType": "ServiceBars",
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

- **`ICCID`** **(Required)** — ICCID. Free-text input.
- **`IMEI`** **(Required)** — Device IMEI. Free-text input.
- **`IssueStartDate`** **(Required)** — Date issue started. Format `MM/DD/YYYY`.
- **`IsthisBYOD`** **(Required)** — Is this device BYOD? `"Yes"` = Bring Your Own Device; `"No"` = not BYOD.
- **`NetworkSettingReset`** **(Required)** — Has the user tried a network settings reset? `"Yes"` = attempted; `"No"` = not attempted.
- **`RecentlyPorted`** **(Required)** — Was this line recently ported?
  - value `"Yes"` (recently ported) → **`PortTicketNumber`** **(Conditional — Required)** — Ticket number associated with the recent port. Free-text input.
  - value `"No"` — not recently ported; no additional note.
- **`SIMReseat`** **(Required)** — SIM reseated? `"Yes"` = reseated; `"No"` = not reseated.
- **`ServiceBars`** **(Required)** — How many bars of service does the user have? `"0"`–`"5"` (number of bars).

## Wireless Service Not Working — `019`

Troubleshoot non-working wireless service. Subcategory ID: `1`.

### Example body

```json
{
    "clientId": 9994,
    "category": "019",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "ICCID",
            "noteValue": "8901260123456789012"
        },
        {
            "noteType": "IMEI",
            "noteValue": "352999001234567"
        },
        {
            "noteType": "IssueStartDate",
            "noteValue": "05/01/2026"
        },
        {
            "noteType": "IsthisBYOD",
            "noteValue": "No"
        },
        {
            "noteType": "NetworkSettingReset",
            "noteValue": "Yes"
        },
        {
            "noteType": "RecentlyPorted",
            "noteValue": "No"
        },
        {
            "noteType": "SIMReseat",
            "noteValue": "Yes"
        },
        {
            "noteType": "ServiceBars",
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

- **`ICCID`** **(Required)** — ICCID. Free-text input.
- **`IMEI`** **(Required)** — Device IMEI. Free-text input.
- **`IssueStartDate`** **(Required)** — Date issue started. Format `MM/DD/YYYY`.
- **`IsthisBYOD`** **(Required)** — Is this device BYOD? `"Yes"` = Bring Your Own Device; `"No"` = not BYOD.
- **`NetworkSettingReset`** **(Required)** — Has the user tried a network settings reset? `"Yes"` = attempted; `"No"` = not attempted.
- **`RecentlyPorted`** **(Required)** — Was this line recently ported?
  - value `"Yes"` (recently ported) → **`PortTicketNumber`** **(Conditional — Required)** — Ticket number associated with the recent port. Free-text input.
  - value `"No"` — not recently ported; no additional note.
- **`SIMReseat`** **(Required)** — SIM reseated? `"Yes"` = reseated; `"No"` = not reseated.
- **`ServiceBars`** **(Required)** — How many bars of service does the user have? `"0"`–`"5"` (number of bars).

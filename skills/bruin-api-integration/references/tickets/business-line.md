# Business Line — Ticket Reference

Business Line (voice) ticket operations. Every ticket is a `POST /api/Ticket`
using the universal body in [../ticket-model.md](../ticket-model.md) — only the
`category` code and the product-specific `notes` below differ. Contacts,
services, and top-level fields follow the ticket model.

All operations use Subcategory ID: `9`. Every example also carries the universal
`MTK` (problem description) and `DDD` (desired due date, `MM/DD/YYYY`) notes from
the ticket model; they are not repeated in the per-operation Notes sections below.

## Operations

| Operation | `category` | Purpose |
| --- | --- | --- |
| Add Additional Lines | `053` | Add new voice lines/features to existing business line service. |
| Add Change Delete Feature | `025` | Add, change, or delete a voice feature on a business line. |
| Circuit Sunset Project | `OOO` | Decommission a circuit as part of a sunset project. |
| Disconnect | `020` | Disconnect a business line. |
| Loss of Line Investigation | `061` | Investigate an unexpected loss/port-out of a line. |
| New Order | `010` | Place a new business line order. |
| Restore Service | `023` | Restore previously disconnected business line service. |
| Service Affecting Trouble | `VAS` | Report a trouble that is degrading (but not fully out) voice service. |
| Service Outage Trouble | `VOO` | Report a full voice service outage. |
| Swap TN | `032` | Swap a telephone number, keeping one WTN active and disconnecting another. |

## Add Additional Lines — `053`

Add new voice lines and/or optional features to existing business line service. Subcategory ID: `9`.

### Example body

```json
{
    "clientId": 9994,
    "category": "053",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "addafeature",
            "noteValue": "1"
        },
        {
            "noteType": "addfeat2",
            "noteValue": "1"
        },
        {
            "noteType": "addfeat3",
            "noteValue": "1"
        },
        {
            "noteType": "addfeat4",
            "noteValue": "1"
        },
        {
            "noteType": "iswifapplic",
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

- **`addafeature`** **(Optional)** — Add a Feature? Select the feature to add.
  - value `"1"` → Call Forwarding → **`typecallforwarding`** **(Conditional — Required)** — Type of Call Forwarding. Free-text.
  - value `"2"` → Caller ID → **`calleridtype`** **(Conditional — Required)** — Caller ID Type. Free-text.
  - value `"3"` → Call Waiting (no additional note)
  - value `"4"` → Voicemail → **`qtyvoicemailboxes`** **(Conditional — Required)** — Quantity of Voicemail Boxes. Free-text. · **`voiceaccessnum`** **(Conditional — Required)** — Voicemail Access Number. Free-text.
  - value `"5"` → Hunt Group → **`huntsequence`** **(Conditional — Required)** — Hunt Sequence. Free-text.
  - value `"6"` → Blocking Feature(s) → **`blockingfeatures`** **(Conditional — Required)** — Blocking Features. Free-text.
  - value `"7"` → Distinctive Ring → **`distring`** **(Conditional — Required)** — Distinctive Ring. Free-text.
  - value `"8"` → Other → **`RequestDescription`** **(Conditional — Required)** — Request Description. Free-text.
- **`addfeat2`** **(Optional)** — Add Feature 2. Same value set as `addafeature`; the child notes here are Conditional—Optional.
  - value `"1"` → Call Forwarding → **`typecallforwarding`** **(Conditional — Optional)** — Type of Call Forwarding. Free-text.
  - value `"2"` → Caller ID → **`calleridtype`** **(Conditional — Optional)** — Caller ID Type. Free-text.
  - value `"3"` → Call Waiting (no additional note)
  - value `"4"` → Voicemail → **`qtyvoicemailboxes`** **(Conditional — Optional)** — Quantity of Voicemail Boxes. Free-text. · **`voiceaccessnum`** **(Conditional — Optional)** — Voicemail Access Number. Free-text.
  - value `"5"` → Hunt Group → **`huntsequence`** **(Conditional — Optional)** — Hunt Sequence. Free-text.
  - value `"6"` → Blocking Feature(s) → **`blockingfeatures`** **(Conditional — Optional)** — Blocking Features. Free-text.
  - value `"7"` → Distinctive Ring → **`distring`** **(Conditional — Optional)** — Distinctive Ring. Free-text.
  - value `"8"` → Other → **`RequestDescription`** **(Conditional — Optional)** — Request Description. Free-text.
- **`addfeat3`** **(Optional)** — Add Feature 3. Same value set and Conditional—Optional child notes as `addfeat2`.
  - value `"1"` → Call Forwarding → **`typecallforwarding`** **(Conditional — Optional)** — Type of Call Forwarding. Free-text.
  - value `"2"` → Caller ID → **`calleridtype`** **(Conditional — Optional)** — Caller ID Type. Free-text.
  - value `"3"` → Call Waiting (no additional note)
  - value `"4"` → Voicemail → **`qtyvoicemailboxes`** **(Conditional — Optional)** — Quantity of Voicemail Boxes. Free-text. · **`voiceaccessnum`** **(Conditional — Optional)** — Voicemail Access Number. Free-text.
  - value `"5"` → Hunt Group → **`huntsequence`** **(Conditional — Optional)** — Hunt Sequence. Free-text.
  - value `"6"` → Blocking Feature(s) → **`blockingfeatures`** **(Conditional — Optional)** — Blocking Features. Free-text.
  - value `"7"` → Distinctive Ring → **`distring`** **(Conditional — Optional)** — Distinctive Ring. Free-text.
  - value `"8"` → Other → **`RequestDescription`** **(Conditional — Optional)** — Request Description. Free-text.
- **`addfeat4`** **(Optional)** — Add Feature 4. Same value set and Conditional—Optional child notes as `addfeat2`.
  - value `"1"` → Call Forwarding → **`typecallforwarding`** **(Conditional — Optional)** — Type of Call Forwarding. Free-text.
  - value `"2"` → Caller ID → **`calleridtype`** **(Conditional — Optional)** — Caller ID Type. Free-text.
  - value `"3"` → Call Waiting (no additional note)
  - value `"4"` → Voicemail → **`qtyvoicemailboxes`** **(Conditional — Optional)** — Quantity of Voicemail Boxes. Free-text. · **`voiceaccessnum`** **(Conditional — Optional)** — Voicemail Access Number. Free-text.
  - value `"5"` → Hunt Group → **`huntsequence`** **(Conditional — Optional)** — Hunt Sequence. Free-text.
  - value `"6"` → Blocking Feature(s) → **`blockingfeatures`** **(Conditional — Optional)** — Blocking Features. Free-text.
  - value `"7"` → Distinctive Ring → **`distring`** **(Conditional — Optional)** — Distinctive Ring. Free-text.
  - value `"8"` → Other → **`RequestDescription`** **(Conditional — Optional)** — Request Description. Free-text.
- **`iswifapplic`** **(Optional)** — Inside Wiring Scope of Work (if applicable). Details on the scope of work for inside line wiring. Free-text.

## Add Change Delete Feature — `025`

Add, change, or delete a voice feature on a business line. Subcategory ID: `9`.

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
            "noteType": "DropDownList",
            "noteValue": "1"
        },
        {
            "noteType": "addchgdel",
            "noteValue": "Add"
        },
        {
            "noteType": "additionalfeature",
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

- **`DropDownList`** **(Required)** — Voice Feature Request Type. Select the feature.
  - value `"1"` → Call Forwarding → **`typecallforwarding`** **(Conditional — Required)** — Type of Call Forwarding. Free-text.
  - value `"2"` → Caller ID → **`calleridtype`** **(Conditional — Required)** — Caller ID Type. Free-text.
  - value `"3"` → Call Waiting (no additional note)
  - value `"4"` → Voicemail → **`qtyvoicemailboxes`** **(Conditional — Required)** — Quantity of Voicemail Boxes. Free-text. · **`voiceaccessnum`** **(Conditional — Required)** — Voicemail Access Number. Free-text.
  - value `"5"` → Hunt Group → **`huntsequence`** **(Conditional — Required)** — Hunt Sequence. Free-text.
  - value `"6"` → Blocking Feature(s) → **`blockingfeatures`** **(Conditional — Required)** — Blocking Features. Free-text.
  - value `"7"` → CNAM / Directory Listing → **`listingtype`** **(Conditional — Required)** — Directory Listing Type. Free-text. · **`cnamdetails`** **(Conditional — Required)** — CNAM Details (Caller ID Name for the listing). Free-text.
  - value `"8"` → Distinctive Ring → **`distring`** **(Conditional — Required)** — Distinctive Ring. Free-text.
  - value `"9"` → Convert to RCF → **`onnetoroffnet`** **(Conditional — Required)** — On-Net or Off-Net. Free-text. · **`PointTo`** **(Conditional — Required)** — number calls should be pointed to. Free-text. · **`numcallpaths`** **(Conditional — Required)** — Number of Call Paths. Free-text. · **`servofptn`** **(Conditional — Required)** — Serving Office / PTN. Free-text.
  - value `"10"` → Other → **`RequestDescription`** **(Conditional — Required)** — Request Description. Free-text.
- **`addchgdel`** **(Required)** — Add, Change, or Delete? Allowed values: `"Add"`, `"Change"`, `"Delete"`.
- **`additionalfeature`** **(Optional)** — Second Feature Request?
  - value `"true"` → submit a second feature request → **`addchgdel`** **(Conditional — Required)** — Add, Change, or Delete? for the second feature. Free-text. · **`voicefeat2`** **(Conditional — Required)** — Second Voice Feature. Free-text.
  - value `"false"` → single feature request only (no additional note)

## Circuit Sunset Project — `OOO`

Decommission a circuit as part of a sunset project. Subcategory ID: `9`.

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

- **`ReplaceDecomCircuit`** **(Required)** — Replacing Decommissioned Circuit? Allowed values: `"No"`, `"Yes"`.
  - value `"No"` → No (no additional note)
  - value `"Yes"` → Yes → **`DecomissionProject`** **(Conditional — Required)** — name or ID of the decommission project. Free-text. · **`ProjectDeadline`** **(Conditional — Required)** — Project Deadline. Date (`MM/DD/YYYY`). · **`CustomerWAN?`** **(Conditional — Required)** — whether the customer is providing their own WAN. Free-text.

## Disconnect — `020`

Disconnect a business line. Subcategory ID: `9`.

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
            "noteType": "DiscTNinHuntGroup",
            "noteValue": "No"
        },
        {
            "noteType": "DisconnectingBTN",
            "noteValue": "False"
        },
        {
            "noteType": "discoreferral",
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

- **`DRN`** **(Required)** — Disconnect Reason.
  - value `"1"` → Downsizing Services at Location (no additional note)
  - value `"2"` → Closing Location (no additional note)
  - value `"3"` → Technology Upgrade (no additional note)
  - value `"4"` → Moving to New Location → **`moveserviceoption`** **(Conditional — Required)** — what should happen to the service at the new location. Free-text.
  - value `"5"` → Non-Payment (no additional note)
  - value `"6"` → Other → **`descother`** **(Conditional — Required)** — description of the disconnect reason. Free-text.
- **`DiscTNinHuntGroup`** **(Required)** — Disconnecting TN in Hunt Group? Allowed values: `"No"`, `"Yes"`.
  - value `"No"` → No (no additional note)
  - value `"Yes"` → Yes → **`New Hunt Sequence`** **(Conditional — Required)** — new hunt group sequence. Free-text.
- **`DisconnectingBTN`** **(Required)** — Disconnecting BTN? Allowed values: `"False"` (No), `"True"` (Yes).
  - value `"False"` → No (no additional note)
  - value `"True"` → Yes → **`NewBTNAfterDisc`** **(Conditional — Optional)** — new Billing Telephone Number (BTN) to use after disconnect. Free-text.
- **`discauthdoc`** **(Optional)** — Disconnection Authorization Paperwork. Accepts a file upload.
- **`discoreferral`** **(Optional)** — Disconnection Referral Number? Temporary referral message on the line after disconnect (typically up to 30 days). Allowed values: `"true"`, `"false"`.
  - value `"true"` → place a temporary referral message → **`referralnumber`** **(Conditional — Required)** — number to place on the temporary referral message. Free-text.
  - value `"false"` → do not place a referral message (no additional note)

## Loss of Line Investigation — `061`

Investigate an unexpected loss or port-out of a line. Subcategory ID: `9`.

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
            "noteValue": "04/22/2026"
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

- **`portoutdate`** **(Required)** — Port Out Date. Date (`MM/DD/YYYY`).

## New Order — `010`

Place a new business line order. Subcategory ID: `9`.

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
            "noteType": "LNAME",
            "noteValue": "Sample value"
        },
        {
            "noteType": "iswifapplic",
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

- **`LNAME`** **(Optional)** — Listed Name. Directory listing name; max 15 characters. Also used for emergency registration. Free-text.
- **`iswifapplic`** **(Optional)** — Inside Wiring Scope of Work (if applicable). Details on the scope of work for inside line wiring. Free-text.

## Restore Service — `023`

Restore previously disconnected business line service. Subcategory ID: `9`.

### Example body

```json
{
    "clientId": 9994,
    "category": "023",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "Disconnect Ticket",
            "noteValue": "Sample value"
        },
        {
            "noteType": "METTELLD",
            "noteValue": "true"
        },
        {
            "noteType": "METTELREG",
            "noteValue": "true"
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

- **`Disconnect Ticket`** **(Optional)** — Disconnect Ticket. Free-text.
- **`METTELLD`** **(Optional)** — MetTel LD Service. Designate MetTel as Long Distance provider (sets PIC code to MetTel). Allowed values: `"true"`, `"false"`.
  - value `"true"` → designate MetTel as Long Distance provider (no additional note)
  - value `"false"` → do not designate MetTel → **`PIC`** **(Conditional — Required)** — Primary Interexchange Carrier (PIC) to designate for long distance. Free-text.
- **`METTELREG`** **(Optional)** — MetTel Regional Service. Designate MetTel as Regional provider (sets LPIC code to MetTel). Allowed values: `"true"`, `"false"`.
  - value `"true"` → designate MetTel as Regional provider (no additional note)
  - value `"false"` → do not designate MetTel → **`LPIC`** **(Conditional — Required)** — Local Primary Interexchange Carrier (LPIC) to designate. Free-text.

## Service Affecting Trouble — `VAS`

Report a trouble that is degrading (but not fully out) voice service. Subcategory ID: `9`.

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
            "noteType": "BHGR",
            "noteValue": "1"
        },
        {
            "noteType": "CFN",
            "noteValue": "Sample value"
        },
        {
            "noteType": "dispatchauth",
            "noteValue": "1"
        },
        {
            "noteType": "fieldtech",
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

- **`BHGR`** **(Required)** — Busy Out Impacted Line(s)?
  - value `"1"` → Yes, please busy out impacted line(s)
  - value `"2"` → Customer does not wish to busy out line(s)
  - value `"3"` → Not applicable as line(s) is not in a hunt group
- **`CFN`** **(Required)** — Temporary Call Forward (TCF)? Free-text.
  - **`PointTo`** **(Conditional — Required)** — number that calls should be pointed to. Free-text.
- **`dispatchauth`** **(Required)** — Carrier Dispatch Authorization. Dispatch preference based on line test results. (Note: if a carrier technician is dispatched and no trouble is found at the minimum point of entry, a $175.00/line fee applies.)
  - value `"1"` → Dispatch only if testing indicates a carrier issue
  - value `"2"` → Dispatch regardless of testing results
  - value `"3"` → Line test only - no dispatch
  - value `"4"` → Request a vendor meet → **`vendormeet`** **(Conditional — Required)** — confirm the vendor meet request. Free-text. · **`vendortech`** **(Conditional — Required)** — vendor technician details. Free-text. · **`datevendormeet`** **(Conditional — Required)** — requested date for the vendor meet. Date (`MM/DD/YYYY`). · **`timevendormeet`** **(Conditional — Required)** — requested time for the vendor meet. Free-text.
- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? (Fees apply, invoiced at contracted ISW rates.) Allowed values: `"true"`, `"false"`.
  - value `"true"` → request a MetTel field technician dispatch → **`HoursTechAssist`** **(Conditional — Required)** — number of hours of technician assistance requested. Free-text. · **`TechHours`** **(Conditional — Required)** — technician hours needed. Free-text. · **`timetechdispatch`** **(Conditional — Required)** — requested dispatch time. Free-text. · **`techsow`** **(Conditional — Required)** — scope of work for the technician. Free-text.
  - value `"false"` → do not request a field technician dispatch (no additional note)

## Service Outage Trouble — `VOO`

Report a full voice service outage. Subcategory ID: `9`.

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
            "noteType": "BHGR",
            "noteValue": "1"
        },
        {
            "noteType": "CFN",
            "noteValue": "Sample value"
        },
        {
            "noteType": "dispatchauth",
            "noteValue": "1"
        },
        {
            "noteType": "fieldtech",
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

- **`BHGR`** **(Required)** — Busy Out Impacted Line(s)?
  - value `"1"` → Yes, please busy out impacted line(s)
  - value `"2"` → Customer does not wish to busy out line(s)
  - value `"3"` → Not applicable as line(s) is not in a hunt group
- **`CFN`** **(Required)** — Temporary Call Forward (TCF)? Free-text.
  - **`PointTo`** **(Conditional — Required)** — number that calls should be pointed to. Free-text.
- **`dispatchauth`** **(Required)** — Carrier Dispatch Authorization. Dispatch preference based on line test results. (Note: if a carrier technician is dispatched and no trouble is found at the minimum point of entry, a $175.00/line fee applies.)
  - value `"1"` → Dispatch only if testing indicates a carrier issue
  - value `"2"` → Dispatch regardless of testing results
  - value `"3"` → Line test only - no dispatch
  - value `"4"` → Request a vendor meet → **`vendormeet`** **(Conditional — Required)** — confirm the vendor meet request. Free-text. · **`vendortech`** **(Conditional — Required)** — vendor technician details. Free-text. · **`datevendormeet`** **(Conditional — Required)** — requested date for the vendor meet. Date (`MM/DD/YYYY`). · **`timevendormeet`** **(Conditional — Required)** — requested time for the vendor meet. Free-text.
- **`fieldtech`** **(Optional)** — MetTel Field Tech Assist? (Fees apply, invoiced at contracted ISW rates.) Allowed values: `"true"`, `"false"`.
  - value `"true"` → request a MetTel field technician dispatch → **`HoursTechAssist`** **(Conditional — Required)** — number of hours of technician assistance requested. Free-text. · **`TechHours`** **(Conditional — Required)** — technician hours needed. Free-text. · **`timetechdispatch`** **(Conditional — Required)** — requested dispatch time. Free-text. · **`techsow`** **(Conditional — Required)** — scope of work for the technician. Free-text.
  - value `"false"` → do not request a field technician dispatch (no additional note)

## Swap TN — `032`

Swap a telephone number: keep one WTN active and disconnect another. Subcategory ID: `9`.

### Example body

```json
{
    "clientId": 9994,
    "category": "032",
    "Services": [
        {
            "ServiceNumber": "3059473030"
        }
    ],
    "notes": [
        {
            "noteType": "Address4ActiveWTN",
            "noteValue": "123 Main St, Springfield, IL 62701"
        },
        {
            "noteType": "Disconnected TN",
            "noteValue": "3059473030"
        },
        {
            "noteType": "New Hunt Sequence",
            "noteValue": "Sample value"
        },
        {
            "noteType": "SWAPBTN",
            "noteValue": "No"
        },
        {
            "noteType": "WTNKeepActive",
            "noteValue": "3059473030"
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

- **`Address4ActiveWTN`** **(Required)** — Address for Kept Active WTN After Swap. Accepts a structured address.
- **`Disconnected TN`** **(Required)** — WTN to be Disconnected. Free-text.
- **`New Hunt Sequence`** **(Optional)** — New Hunt Sequence. Free-text.
- **`SWAPBTN`** **(Required)** — Is the Number Swapping the BTN? Allowed values: `"No"`, `"Yes"`.
- **`WTNKeepActive`** **(Required)** — WTN to Keep Active. Free-text.

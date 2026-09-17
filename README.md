# bruin-plugin

A [Claude Code plugin](https://docs.claude.com/en/docs/claude-code/plugins) that
teaches Claude to help clients build and debug integrations against the **Bruin
Public API** — MetTel's REST API for tickets, inventory, sites, users, and
webhooks.

## What's inside

| Component | Purpose |
| --- | --- |
| [`skills/bruin-api-integration`](skills/bruin-api-integration) | Agent skill covering OAuth 2.0 auth, the universal ticket model, per-product note-type schemas (Smart Phones, Cable, Ethernet, Business Line, Starlink, SD-WAN, PIAB), inventory/site/user endpoints, and webhook payloads. |

The skill uses **progressive disclosure**: only `SKILL.md` is always in
context; per-product references and workflows load on demand. See the
[skill README](skills/bruin-api-integration/README.md) for the internal layout.

## Install

### Option A — load a local checkout

```bash
git clone <this-repo> bruin-plugin
claude --plugin-dir ./bruin-plugin
```

The skill activates automatically based on the user's request (auth, ticket
creation, inventory lookup, webhooks, etc.). It will not appear in `/plugin`,
which only lists marketplace installs.

### Option B — install from a marketplace

If this repo is published to a marketplace:

```
/plugin marketplace add <marketplace-url>
/plugin install bruin-plugin
```

## When Claude uses the skill

Claude triggers `bruin-api-integration` when a task involves:

- OAuth 2.0 client-credentials auth against Bruin, or a `401` / `403` on the API
- Building a `POST /api/Ticket` payload for a specific product (correct
  `category` code, required and conditional notes)
- Placing device, PIAB, or Specialty Business Line orders (`/PlaceOrder`)
- Listing or looking up inventory, reading `BTN` / `IMEI` / `PIC` attributes
- Creating or reading sites and users
- Subscribing to and parsing ticket webhooks
- Questions about **Bruin Client IDs** vs. **OAuth Client IDs**, scopes, or
  ticket category codes

## Repo layout

```
bruin-plugin/
├── .claude-plugin/
│   └── plugin.json                # plugin manifest
├── skills/
│   └── bruin-api-integration/
│       ├── SKILL.md               # entry point (always in context when active)
│       └── references/            # loaded on demand
└── README.md
```

## Contributing

The skill's content is distilled from MetTel's published Bruin Public API docs.
When those docs change (new products, note types, category codes), re-distill
the affected `references/tickets/*.md` file from its source. Do not hand-edit
note types or category codes that aren't in the source docs — the skill's
value is fidelity to the published API.

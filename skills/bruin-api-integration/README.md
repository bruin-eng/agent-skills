# Bruin Public API Integration — Agent Skill

A self-contained [Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)
that teaches an AI agent to help clients build and debug integrations against
the **Bruin Public API** (MetTel's REST API for tickets, inventory, sites,
users, and webhooks). Built from MetTel's published web docs.

## What's inside

```
bruin-api-integration/
├── SKILL.md                       # entry point: core concepts + navigation map
└── references/                    # loaded on demand
    ├── authentication.md          # OAuth 2.0, environments, scopes, token lifecycle
    ├── ticket-model.md            # universal POST /api/Ticket body + note conventions
    ├── ordering.md                # PlaceOrder (devices), NewOrder (PIAB), topics, details
    ├── inventory.md  site.md  user.md  webhooks.md
    ├── tickets/                   # per-product note-type schemas (73 operations)
    │   ├── smart-phones.md  cable-internet.md  ethernet-internet.md
    │   ├── business-line.md  starlink.md  sd-wan.md  piab.md
    └── workflows/                 # multi-call playbooks
        ├── cell-phone-lifecycle.md  ordering-piab.md  repair-tickets.md
```

The design follows **progressive disclosure**: `SKILL.md` is always in context;
everything under `references/` is opened only when the task calls for it. The
per-product ticket files preserve the API's conditional note logic (a child note
that is required only when a parent note takes a specific value).

## How to use it

- **Claude Code / claude.ai:** copy the `bruin-api-integration/` folder into a
  skills directory (e.g. `.claude/skills/`). The agent triggers it based on the
  `description` in `SKILL.md`'s frontmatter.
- **Agent SDK / custom app:** load `SKILL.md` into the system prompt (or as a
  tool the model can invoke), and give the model file access to `references/` so
  it can pull the relevant file on demand. The `description` frontmatter is the
  routing signal for when to engage the skill.

## Maintenance

Content is derived from the Docusaurus docs under `Bruin-Public-API-Webdocs/`.
When those docs change (new products, note types, or category codes),
re-distill the affected `references/tickets/*.md` file from its source folder
under `docs/Tickets/`. Never hand-edit note types or category codes that aren't
in the source docs — the skill's value is fidelity to the published API.

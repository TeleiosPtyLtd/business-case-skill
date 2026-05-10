# business-case (Claude Code skill)

Author interactive financial business cases by talking to Claude Code. The
skill clones the [CBAgent template][tpl], writes `project.config.js` from
your description, and tells you how to run and share it.

[tpl]: https://github.com/TeleiosPtyLtd/business-case

## Install

In Claude Code:

```
/plugin marketplace add TeleiosPtyLtd/business-case-skill
/plugin install business-case@teleios
```

## Use

Once installed, just describe a decision:

> "Build a business case for replacing our annotation pipeline with an
> AI-assisted workflow. $250k engagement, 3-year horizon, expect to save
> ~$700k/yr in vendor fees once Phase 2 lands."

Claude will:

1. Clone the template into `./<short-slug>/`.
2. Author `project.config.js` — assumptions, items (costs/benefits), three
   scenarios (Conservative / Central / Optimistic), and per-assumption
   sensitivity ranges.
3. Start `live-server` so the browser opens with hot reload.
4. Tell you how to click Share to upload the snapshot to a hosted backend
   and get a password-protected viewer URL.

## What's inside the model

- **NPV / BCR / IRR** with a 5-stage value waterfall per item: gross →
  overlap → phase delivery probability → counterfactual capture → cash/soft
  split.
- **Three scenarios** with per-item parameter overrides.
- **Sensitivity tornado** ranking the top 5 levers by NPV swing.
- **Hover-aware stacked bar charts**, timeline view, full data tables, and
  Excel-compatible CSV exports.
- **Sandboxed formula compiler** — every assumption is a numeric input you
  edit live; every cost/benefit value is a JS expression referencing those
  inputs.
- **Config validator** — surfaces a banner if probabilities fall out of
  range, identifiers don't resolve, or required fields are missing.

## What's outside

This is decision-support, not an audit-grade financial model. The engine
**doesn't** model: tax shields, depreciation/amortisation, inflation (real
vs nominal), working capital, capex/opex distinction, or Monte Carlo
simulation. If your CFO needs any of those, drop the CSV export into Excel
for the final pass.

## Repo layout

```
business-case-skill/
├── .claude-plugin/
│   ├── plugin.json          # plugin metadata
│   └── marketplace.json     # marketplace manifest (teleios)
├── skills/
│   └── business-case/
│       └── SKILL.md         # the skill itself (frontmatter + authoring guide)
└── README.md
```

## Self-hosting the share backend

The skill wires uploads to `https://models.teleios.au/api/share` by default
(Teleios's hosted instance of [`business-case-server`][srv]). To point at
your own deployment, edit `share.config.js` in any cloned business case:

```js
window.CBAGENT_SHARE_ENDPOINT = "https://your-host.example.com/api/share";
```

[srv]: https://github.com/TeleiosPtyLtd/business-case-server

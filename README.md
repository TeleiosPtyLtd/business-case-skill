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
> AI-assisted workflow. $250k upfront cost, we expect to save ~$60k/month
> on vendor fees once Phase 2 lands six months in. Model it monthly over
> 18 months."

Claude will:

1. Clone the template into `./<short-slug>/`.
2. Pick a **granularity** (day / week / month / quarter / year) and **horizon**
   (integer count of those periods) that matches how you think about the
   decision.
3. Author `project.config.js` — assumptions, items (costs/benefits), three
   scenarios (Conservative / Central / Optimistic), per-assumption
   sensitivity ranges, plain-language risk titles.
4. Start `live-server` so the browser opens with hot reload.
5. Tell you how to click Share to upload the snapshot to a hosted backend
   and get a password-protected viewer URL.

## What's inside the model

- **Net cumulative + payback period** as the headline. The page is nominal
  cashflows over your chosen timescale: "you net $X over the next {horizon}",
  "payback at period N".
- **Granularity-native authoring** — every assumption and every benefit/cost
  formula is expressed per-period in the case's chosen unit. No
  annualising, no monthly-vs-yearly fudging mid-case.
- **Item deferral** — a benefit that kicks in five months late uses
  `startPeriod: 6`. No ramp curves to tune; the timing is explicit.
- **Three scenarios** with per-assumption overrides — Conservative /
  Central / Optimistic.
- **Sensitivity tornado** ranking the top 5 assumptions by net-cumulative
  swing.
- **Hover-aware stacked bar charts**, timeline view, full data tables, and
  per-section drill-downs.
- **Sandboxed formula compiler** — every assumption is a numeric input you
  edit live; every cost/benefit value is a JS expression referencing those
  inputs.
- **Config validator** — surfaces a banner when identifiers don't resolve,
  required fields are missing, or legacy schema fields are present.

## What's outside

This is decision-support, not an audit-grade financial model. The engine
**doesn't** model: NPV / IRR / discounted cashflow (deliberate — see below),
tax shields, depreciation/amortisation, inflation (real vs nominal),
working capital, capex/opex distinction, or Monte Carlo simulation. If
your CFO needs any of those, take the snapshot into Excel for the final
pass.

**Why no NPV?** Most small/medium-business decisions are read in nominal
dollars over a horizon the buyer already has in mind. The discount rate
adds a knob nobody wants to defend and an answer that's harder to talk
about. The page's job is to be readable end-to-end on one pass, not to
satisfy a finance textbook. If you genuinely need NPV, model in your CFO's
spreadsheet.

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

---
name: business-case
description: Use this skill whenever the user wants to build, edit, or share a CBAgent business case — i.e. an interactive financial model rendered in the CBAgent UI (NPV, BCR, IRR, value waterfall, timeline, sensitivity). Triggers include "build a business case for X", "model the financial impact of Y", "turn this proposal into a business case", "update the costs/benefits/assumptions", "regenerate the business case after this change", or any request that ends with the user wanting an interactive view of a decision's financial impact.
---

# CBAgent Business Case skill

## What you'll do

The user has a public template repo (referenced as `TEMPLATE_REPO` below).
Your job, when they describe a project or decision, is to:

1. **Clone the template** into a working directory in the user's current
   workspace (default: `./<short-project-slug>/`).
2. **Author `project.config.js`** — the single source of truth for the
   business case. Every cost, benefit, assumption, and scenario lives there.
3. **Validate** — read the file back, run a mental sanity check against the
   schema, fix obvious errors (e.g. unreferenced assumption ids).
4. **Tell the user how to run and share it.**

Don't touch `src/`, `src2/`, `server/`, or `index.html` unless the user
explicitly asks for new visual capability — the engine is already generic.

## TEMPLATE_REPO

```
git@github.com:TeleiosPtyLtd/business-case.git
```

If the user's environment can't auth via SSH, fall back to:
`https://github.com/TeleiosPtyLtd/business-case.git` (requires the repo to
be public, or a credential helper to be configured).

## Step 1 — Clone

If the user hasn't already specified a directory, derive a short slug from
the project name (`acme-pricing-tool`, `gpu-cluster-2026`, etc.) and run:

```sh
git clone --depth 1 git@github.com:TeleiosPtyLtd/business-case.git ./<slug>
cd ./<slug>
rm -rf .git           # detach from the template; this is now the user's repo
```

If the directory already exists with a `project.config.js`, **don't re-clone**
— assume the user is iterating on an existing case and just edit the config.

## Step 2 — Author `project.config.js`

The schema lives in the repo at `project.config.js` (a placeholder template
with generic assumption/item names) and `examples/minimal.config.js` (a
1-cost + 1-benefit skeleton). Read either before writing.

### Required structure

```js
window.PROJECT_CONFIG = {
  meta: { name, shortName, description, eyebrow },
  horizon: 7,                     // display horizon in years (5–10 typical)
  defaultScenario: "central",
  scenarios: {
    central:      { label, desc, overrides: {}, counterfactualShift: 0 },
    conservative: { label, desc, overrides: { ... }, counterfactualShift: 0.15,
                    itemOverrides: { item_id: { cashRealisation: 0.5 } } },
    optimistic:   { label, desc, overrides: { ... }, counterfactualShift: -0.10 },
  },
  categoryColors: {
    // category id -> CSS variable. Use existing palette only:
    //   benefits:  --c-mint, --c-blue, --c-green, --c-purple, --c-mintlight
    //   costs:     --c-red, --c-yellow, --c-orange
  },
  assumptions: [ /* see below */ ],
  items:       [ /* see below */ ],
};
```

### `assumptions[]`

```js
{
  id: "snake_case_id",            // referenced in formula strings
  label: "Human-readable label",
  group: "Group name",            // groups together in the rail
  value: 1500000,                 // numeric default (sensible & conservative)
  unit: "$" | "$/yr" | "%" | "events" | "yrs" | ...,
  step: 50000,
  icon: "IconDollar",             // see icon list below
  domain: "internal" | "<vendor>.com",
  source: "Short attribution",
  description: "Plain-language definition. What is this and what does it mean?",
  rationale:   "Modelling justification — why this number.",
  sensitivityRange: { lo: 0.5, hi: 1.5 },   // optional, multipliers on `value`
}
```

Both `description` AND `rationale` are required for every assumption — the
user has corrected this before. Plain-language description first; modelling
justification second.

Available icons: `IconDollar`, `IconUsers`, `IconPercent`, `IconBuilding`,
`IconClock`, `IconBolt`, `IconShield`, `IconTrend`, `IconLeaf`, `IconCube`.

Probabilities: use ids ending in `_prob` and put values in 0..100 (the
validator enforces this).

### `items[]`

```js
{
  id:   "unique_id",
  name: "Item name",
  kind: "cost" | "benefit",
  category: "ops" | "capacity" | "commercial" | "reuse" | "cost_pri" | "cost_run" | "cost_int",

  lump:      false,        // true → one-off in startYear; false → annuity over horizon
  startYear: 1,            // 1-indexed
  phase:     2,            // 0..4. Cumulative phase delivery probability is applied.
                           // Costs use phase 0 (always realise).

  // The cash value as a JavaScript expression *as a string*. References any
  // assumption id and standard math helpers (pow, min, max, abs, log, sqrt,
  // exp, floor, ceil, round, PI, E). One-off items: $ value. Recurring items: $/yr.
  //
  // The formula compiler is sandboxed: only identifiers in {assumption_ids,
  // math helpers}, numeric literals, and operators + - * / ( ) , . are allowed.
  // Assignments, semicolons, brackets, function bodies, and unknown
  // identifiers will fail validation.
  gross: "events_per_year * hours_per_event * loaded_rate + exposure_pool * recovery_fraction",

  overlap:        0.10,    // 0..1, fraction already counted by other initiatives
  counterfactual: 0.20,    // 0..1, fraction the user would capture without this work
  cashRealisation: 0.70,   // 0..1, fraction realised as cash (vs soft / freed time)

  horizonOverride: "contract_remaining_years",  // optional assumption id capping annuity duration

  desc: "1-2 sentence narrative — shown in the drill-down.",
  uses: ["assumption_ids", "that", "drive", "this"],   // for UI badges
}
```

The waterfall:
`gross × (1 - overlap) × cumulativePhaseProb × (1 - counterfactual) → split into cash + soft`.

### Scenarios

`overrides` maps `assumption_id → number`, applied on top of defaults.
`counterfactualShift` adds (clamped to 0..1) to every item's `counterfactual`.
`itemOverrides[item_id]` patches per-item parameters (overlap,
counterfactual, cashRealisation, phase, startYear) for a specific scenario.

## Authoring loop

When the user describes a project:

1. **Identify the decision.** Action, counterfactual, who pays, who benefits,
   over what time horizon.
2. **Sketch the items.** 2–4 costs + 3–8 benefits. Cap at ~8 benefits — the
   user has previously said too many stacked benefits look cluttered. Merge
   granular wins into themed items (e.g. "Operational efficiency" can roll
   up several small ops gains).
3. **Identify the levers.** For each item, what numeric inputs drive its
   value? Those become assumptions.
4. **Group assumptions.** Sensible groups: "Engagement", "Financial",
   "Scale", "Operations", "Delivery Confidence". Phase probabilities
   (`p1_prob` … `p4_prob`) are conventional.
5. **Pick reasonable, conservative defaults.** The user has said: "the best
   estimates are reasonable and conservative."
6. **Write `description` AND `rationale` for every assumption.**
7. **Pick categories and colors.** Costs use `cost_pri` / `cost_run` /
   `cost_int`. Benefits get one of the benefit categories above.
8. **Define three scenarios.** central + conservative + optimistic. Override
   only the parameters that meaningfully differ. Conservative
   `counterfactualShift` should be positive (the user captures more on their
   own); optimistic negative.

## Constraints from prior conversations

- **Keep total benefits ≤ ~8** so the stacked chart stays readable.
- **Costs and benefits should look symmetric** in the drill-down — engine
  handles this; don't try to inject custom UI.
- **Every assumption needs `description` (plain language) AND `rationale`
  (modelling justification).** Both, not one.
- **Charts hover-tooltip per year** — already built. Don't reinvent.
- **Everything is exportable to Excel** — already built.

## Step 3 — Validate

The runtime validator surfaces a banner at the top of the page if the config
is broken. Mentally pre-check:

- Every `item.category` exists in `categoryColors`.
- Every identifier in every `gross` formula is either an assumption id or a
  whitelisted math helper.
- All `*_prob` assumption values are between 0 and 100.
- `overlap`, `counterfactual`, `cashRealisation` are between 0 and 1.
- Item ids are unique; both costs and benefits exist.

## Step 4 — Run and share

The editor is a static page — no build step. Run it with `live-server` so the
browser auto-opens and reloads on every save. Tell the user:

```sh
cd <slug>
npx live-server@1.2.2          # http://localhost:8080, opens browser
```

Save any file (`project.config.js`, a JSX file, …) → browser reloads → new
numbers. JSX is transpiled in-browser by Babel-standalone, so hot reload
works without any build pipeline.

After cloning, **start `live-server` for the user** as a background process
(don't block the conversation), then tell them where to look. A pattern that
works well: run it under a backgrounded shell so it stays alive across turns,
e.g. `npx live-server@1.2.2 > /tmp/live-server-<slug>.log 2>&1 &` (or use
your harness's background-process mechanism). The browser opens
automatically; the user starts editing immediately.

Sharing is pre-wired to the Teleios hosted backend at `https://models.teleios.au`
via `share.config.js`. Click Share → set a password → upload — the backend
returns a `/view/{id}` URL. Override `CBAGENT_SHARE_ENDPOINT` in
`share.config.js` if pointing at a different deployment of
[business-case-server](https://github.com/TeleiosPtyLtd/business-case-server).

## Examples

- `project.config.js` — placeholder template with generic assumption and
  item names. Always present in a fresh clone; overwrite with project-specific
  content when authoring.
- `examples/minimal.config.js` — smallest viable config (1 cost, 1 benefit).

When in doubt, copy `examples/minimal.config.js` over `project.config.js` and
grow from there.

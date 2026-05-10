---
name: business-case
description: Use this skill whenever the user wants to build, edit, or share a CBAgent business case — i.e. an interactive financial model rendered in the CBAgent UI (NPV, BCR, IRR, value waterfall, timeline, sensitivity). Triggers include "build a business case for X", "model the financial impact of Y", "turn this proposal into a business case", "update the costs/benefits/assumptions", "regenerate the business case after this change", or any request that ends with the user wanting an interactive view of a decision's financial impact.
---

# CBAgent Business Case skill

## What you'll do

The user has a public template repo. Your job, when they describe a project
or decision, is to:

1. **Clone the template** into a working directory.
2. **Author `project.config.js`** through the six-phase loop in Step 2 —
   every cost, benefit, assumption, and scenario lives in this file.
3. **Pre-flight** — run the validator + critique gates in Step 3.
4. **Run and share** — start `live-server`, point the user at the URL.

Don't touch `src/`, `src2/`, or `index.html` unless the user explicitly asks
for new visual capability — the engine is already generic.

## TEMPLATE_REPO

```
git@github.com:TeleiosPtyLtd/business-case.git
```

Fallback (HTTPS, public clone):
`https://github.com/TeleiosPtyLtd/business-case.git`.

## Step 1 — Clone

If the user hasn't specified a directory, derive a short slug from the
project name (`acme-pricing-tool`, `gpu-cluster-2026`, etc.):

```sh
git clone --depth 1 git@github.com:TeleiosPtyLtd/business-case.git ./<slug>
cd ./<slug>
rm -rf .git
```

If the directory already exists with a `project.config.js`, **don't
re-clone** — the user is iterating, just edit.

## Step 2 — Author `project.config.js` (six-phase loop)

Most BCs fail at the early phases, not at the math. Don't skip them.

### Phase 1 — Frame the decision

Before any numbers, establish *what is being chosen between* — value is
*delta*, not absolute. Five fields, all required:

- **The counterfactual.** Not "doing nothing" — usually "the cheapest
  credible alternative" (a different vendor, an internal team, a partial
  scope). The honest comparator.
- **Decision audience.** A CFO needs different rigor than a champion's
  internal slide deck.
- **Time horizon, with a reason.** Why 7 years? Contract length? Tech
  lifecycle? Strategic plan? The number must come from somewhere.
- **Who pays vs who captures.** Agency mismatches (org A pays, org B
  benefits) often kill an otherwise "good" project.
- **What "yes" actually unlocks.** Often the early phases are infrastructure
  and the later phases unlock the bigger value but are conditional.

If any of these five are unclear from the user's brief, **ask them in a
single combined message** before writing the config — don't five-question
them. Then write the answers into `meta.description` so the framing is
visible at the top of the page.

### Phase 2 — Decompose each benefit (first principles)

For every candidate benefit, derive a four-step value chain:

```
project action → what changes in the world → how that becomes $ → who captures it
```

Example (better baggage data):
```
better evidence per dispute  →  more disputes contested successfully
                             →  fewer rebate $ paid + faster resolution time
                             →  PA opex (rebates) + AOC capacity (hours)
```

The chain is the **mechanism**. Every item's `desc` field must name it
(1-2 sentences). Then `gross` implements *that* chain — not a freelance
formula. Models that collapse the chain into a single magic number lose
the audit trail and can't be defended.

Rules of thumb:
- Cap total benefits at **≈8** so the stacked chart reads cleanly. Merge
  granular wins into themed items ("Operational efficiency" rolls up
  multiple small ops gains).
- Costs and benefits should look symmetric in the drill-down — the engine
  handles this; don't try to inject custom UI.

### Phase 3 — Source & calibrate every belief

Every numeric input gets *one* of these sources, in priority order:

| Source | When | What you do |
|---|---|---|
| **Internal data** | The user's org has it | **Ask them.** "Do you have FY24 dispute count? If not, an order-of-magnitude is fine — I'll flag." |
| **Authoritative external** | Public benchmark exists | **Run a web search.** Industry reports, regulator filings, peer-reviewed sources. Cite the URL in `source`. |
| **Vendor proposal** | Vendor stated the number | Cite verbatim, flag bias: `source: "Teleios proposal v2 (vendor-stated)"`. |
| **First-principles Fermi** | No data, but decomposition works | Show the working in `rationale`: `30 incidents × 4 hrs × $80/hr` — not just `9600`. |
| **`[CONFIRM]`** | Nobody knows yet | Flag explicitly: `source: "[CONFIRM] needs ops budget review"`. The validator surfaces these. |

Online research is **encouraged** for industry benchmarks, regulator
filings, and macro inputs (discount rates, inflation, sector cost
structures). Use the `WebSearch` tool. Search template:

> "Find a 2023+ benchmark for `<metric>` in `<industry/geography>`. I need:
> (1) the typical value, (2) the range across comparable orgs, (3) a source
> I can cite — peer-reviewed or industry-standard preferred."

Cite the source in the assumption's `source` field. If results are thin or
contradictory, prefer the lower estimate and note the uncertainty in
`rationale`.

When asking the user, batch your questions. One message with three precise
questions beats three rounds of one-question asks. Template:

> "To make these estimates solid I need three things you'll know: (1)
> `<datum 1>`, (2) `<datum 2>`, (3) `<datum 3>`. If you don't have one, I'll
> proceed with my Fermi estimate and flag `[CONFIRM]`."

**Range calibration.** For each belief, set `sensitivityRange: { lo, hi }`
based on a *coherent low/high case*, not a generic ±25%. Lo is the value
a sceptical CFO would defend; hi is the value the champion would defend.
Probability beliefs (ids ending `_prob`) get tighter ranges that respect
0–100.

### Phase 4 — Compose items with mandatory critique

Each item carries:

- `gross` — the formula implementing the value chain from Phase 2.
- `overlap`, `counterfactual`, `cashRealisation` — each justified in
  `desc` or in inline reasoning, not just numbers.
- `critiques: [str, str, str]` — **at least three specific attacks** drawn
  from the lenses below.

The 10 critique lenses (pick the 3–5 most apt per item):

1. **Counterfactual capture** — does a cheaper alt do most of this anyway?
2. **Overlap** — is another initiative already booking part of this benefit?
3. **Cash vs soft** — does the dollar leave a budget line, or is it freed
   time that gets reabsorbed?
4. **Adoption** — built ≠ used. What drives uptake?
5. **Phase risk** — how conditional is this on prior phases landing?
6. **Time-of-arrival** — what if it lands a year later than modelled?
7. **Behavioural absorption** — gains spent on more activity, not banked.
8. **Selection bias** — modelling on the average case or the marginal case?
9. **Dependency** — what if the vendor / champion / sponsor leaves?
10. **Externalities** — costs imposed on others (training, disruption,
    opportunity).

Critiques must be **specific**. Bad: "this might not deliver as expected".
Good: "Code 87 dispute success depends more on legal precedent in airline
contracts than on data quality — better evidence is necessary but not
sufficient without legal-team capacity to act on it."

The critiques live as comments above the item in `project.config.js` (the
schema doesn't currently have a critiques field; comments are the carrier).

### Phase 5 — Pressure-test the model

**Scenarios as narratives, not knob-twists.** Three scenarios:

- **Conservative** — a steelman bear case. Internally consistent: lower
  scale + lower phase delivery + higher counterfactual capture.
- **Central** — best evidence right now.
- **Optimistic** — a steelman bull case. Likewise coherent.

Write a one-sentence story for each in `desc` describing the world it
lives in.

**Self-critique pass.** Before declaring done, read your own model and
answer in writing — produce a `CRITIQUE.md` next to `project.config.js`:

1. *What's the weakest belief?* (Lowest source quality × biggest NPV
   impact — top of the sensitivity tornado with weakest `source`.)
2. *What's the most likely double-count?*
3. *Where is "soft" smuggled into "cash"?*
4. *What single dependency, if it slipped, would change the conclusion?*

**Sensitivity sanity check.** When the user opens the page, check that
the top 5 levers in the Summary tornado match what theory predicts. If
they don't (e.g. discount rate dominating an annuity-heavy model is
expected; an obscure ops parameter dominating is suspicious), the model
has a structural problem — fix it before sharing.

### Phase 6 — Pre-flight checklist

Hard gates before declaring done:

- Every assumption has `source`, `rationale`, `description`, and
  `sensitivityRange`. None empty.
- Every item has a value-chain `desc` and ≥3 specific critiques (as
  comments above the item).
- `CRITIQUE.md` exists and names the weakest belief by id.
- The runtime validator banner is clean.
- Top 5 sensitivity levers reviewed and match theory.

If a gate fails, fix it before telling the user it's done.

## Schema reference

### `assumptions[]`

```js
{
  id: "snake_case_id",            // referenced in formula strings
  label: "Human-readable label",
  group: "Group name",            // groups together in the rail
  value: 1500000,                 // numeric default
  unit: "$" | "$/yr" | "%" | "events" | "yrs" | ...,
  step: 50000,
  icon: "IconDollar",             // see icon list below
  domain: "internal" | "<vendor>.com",
  source: "Short attribution",    // [CONFIRM] / vendor / URL / Fermi
  description: "Plain-language definition. What is this and what does it mean?",
  rationale:   "Modelling justification — why this number, with working if Fermi.",
  sensitivityRange: { lo: 0.5, hi: 1.5 },   // multipliers on `value`
}
```

Both `description` AND `rationale` are required for every assumption.

Available icons: `IconDollar`, `IconUsers`, `IconPercent`, `IconBuilding`,
`IconClock`, `IconBolt`, `IconShield`, `IconTrend`, `IconLeaf`, `IconCube`.

Probabilities: ids ending in `_prob`, values 0..100 (validator enforces).

### `items[]`

```js
{
  id:   "unique_id",
  name: "Item name",
  kind: "cost" | "benefit",
  category: "ops" | "capacity" | "commercial" | "reuse"
          | "cost_pri" | "cost_run" | "cost_int",

  lump:      false,        // true → one-off in startYear; false → annuity
  startYear: 1,            // 1-indexed
  phase:     2,            // 0..4. Costs use phase 0 (always realise).

  // Formula string, sandboxed: only assumption ids, math helpers (pow, min,
  // max, abs, log, sqrt, exp, floor, ceil, round, PI, E), numeric literals,
  // and operators + - * / ( ) , . are allowed.
  gross: "events_per_year * hours_per_event * loaded_rate
        + exposure_pool * recovery_fraction",

  overlap:        0.10,    // 0..1, fraction already counted by other initiatives
  counterfactual: 0.20,    // 0..1, fraction captured without this work
  cashRealisation: 0.70,   // 0..1, fraction realised as cash (vs soft / freed time)

  horizonOverride: "contract_remaining_years",  // optional id capping annuity duration

  desc: "1-2 sentence value-chain mechanism (Phase 2). Shown in drill-down.",
  uses: ["assumption_ids", "that", "drive", "this"],   // for UI badges
}
```

Engine math: `gross × (1 - overlap) × cumulativePhaseProb × (1 - counterfactual)
→ split into cash + soft`.

### Scenarios

```js
scenarios: {
  central:      { label, desc, overrides: {}, counterfactualShift: 0 },
  conservative: { label, desc, overrides: { id: value, ... },
                  counterfactualShift: 0.15,
                  itemOverrides: { item_id: { cashRealisation: 0.5, ... } } },
  optimistic:   { label, desc, overrides: { ... }, counterfactualShift: -0.10 },
}
```

`counterfactualShift` adds (clamped 0..1) to every item's `counterfactual`.
`itemOverrides[item_id]` patches per-item parameters per scenario.

## Step 3 — Validate

The runtime validator surfaces a banner at the top of the page when the
config is broken. Pre-check:

- Every `item.category` exists in `categoryColors`.
- Every identifier in every `gross` formula is an assumption id or a
  whitelisted math helper.
- All `*_prob` values are 0..100.
- `overlap`, `counterfactual`, `cashRealisation` are 0..1.
- Item ids unique; both costs and benefits exist.

Also run the Phase 6 critique gates manually before declaring done.

## Step 4 — Run and share

```sh
cd <slug>
npx live-server@1.2.2          # http://localhost:8080, opens browser
```

Save any file → browser reloads → new numbers. JSX is transpiled
in-browser by Babel-standalone; no build pipeline.

**Run `live-server` for the user** as a background process so it stays
alive across turns, e.g.
`npx live-server@1.2.2 > /tmp/live-server-<slug>.log 2>&1 &`. The browser
opens automatically.

Share is pre-wired to `https://models.teleios.au` via `share.config.js`.
Click Share → password → upload → backend returns a `/view/{id}` URL.

## Examples

- `project.config.js` — placeholder template (generic assumption and item
  names). Overwrite when authoring.
- `examples/minimal.config.js` — smallest viable config (1 cost, 1 benefit).

When in doubt, copy `examples/minimal.config.js` over `project.config.js`
and grow from there.

## Constraints from prior conversations

- **≤8 benefits** total — stacked chart readability.
- **`description` AND `rationale`** required on every assumption (both,
  not one).
- **Costs and benefits visually symmetric** — engine handles this.
- **Hover tooltips, Excel export** — already built; don't reinvent.
- **Numbers should be reasonable and conservative** by default — better
  to be defended than to be optimistic.

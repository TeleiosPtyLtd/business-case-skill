---
name: business-case
description: Use this skill when the user wants to build, edit, or share a CBAgent business case — an interactive financial model rendered as a four-section proof (Now / And / Then / Risks). Triggers include "build a business case for X", "model the financial impact of Y", "turn this proposal into a business case", "update the assumptions/benefits/costs", or any request that ends with the user wanting an interactive page that walks a buyer through a decision's projected impact. Not for: static slides, audit-grade financial models (no tax/depreciation/working-capital), Monte Carlo simulation, or multi-stream P&L modelling.
---

# CBAgent Business Case skill

## What you'll do

When the user describes a project or decision:

1. **Clone the template** into a working directory.
2. **Author `project.config.js`** — the only file that carries case data. The engine, UI, and exports are already generic.
3. **Pre-flight** — validator + critique pass.
4. **Run** — `live-server`, point them at the URL.

Don't touch `src/`, `src2/`, `index.html`, or any other source file unless the user explicitly asks for new visual capability.

## TEMPLATE_REPO

```
git@github.com:TeleiosPtyLtd/business-case.git
```

Public-clone fallback: `https://github.com/TeleiosPtyLtd/business-case.git`.

## Step 1 — Clone

Derive a short slug from the project name (`acme-pricing-tool`, `gpu-cluster-2026`):

```sh
git clone --depth 1 git@github.com:TeleiosPtyLtd/business-case.git ./<slug>
cd ./<slug>
rm -rf .git
```

If `<slug>/project.config.js` already exists, **edit instead of re-cloning**.

## Step 2 — Understand the page so you can author for it

The page is a four-section rhetorical proof. Your config drives each section directly. Knowing what each section *does* tells you what each field is *for*.

### Now
The buyer confirms the world they live in. Surfaces the **top 3 world-fact assumptions** (those with `controllable: false`, ranked by scope-1 sensitivity) and a live equation derived from `baseline[]` that shows what those assumptions imply about today (e.g. *"Your annual revenue today"* = proposals × win-rate × fee = $300k/yr). Each row has a *Sounds right* button. Once all three are confirmed, a *Let's proceed* button reveals the rest of the page.

### And
What you commit to change. Surfaces the **top 3 commitment assumptions** (those with `controllable: true`, ranked by scope-1 sensitivity). Each row has an *Okay* button. Beneath, two subtotals derived from the scope-1 quantitative benefits:
- `+X% change to your annual revenue (+$Y/yr)` — sum of `revenue_uplift` items as a % of the `kind: "revenue"` baseline.
- `$Z/yr recurring cost savings` — sum of `cost_saving` items.

A *Show the math* toggle reveals the per-benefit multiplication chains.

### Then
The outcome. Three rows — Benefits / Costs / Net — with the headline pinned at the bottom: **net cumulative ($) over the horizon** and **payback period** (the period where cumulative net first crosses zero). No NPV/IRR/PV — the page is nominal cashflows over the buyer's chosen timescale.

### Risks
Bare-titles disclosure list. Each risk states one thing that could go wrong, grouped by `locus` (`"commitment"` = under our control, `"world"` = shared). No mitigation copy — that's implementation detail and lives in the proposal, not the BC.

## Voice & writing rules

**Read this before writing any field.** The buyer reads the page literally, on one pass, without a finance background. Every word that asks them to translate is a word they won't pay for. This applies to *every* user-facing string — `meta.description`, item `name`, item `desc`, assumption `label`, assumption `description`, baseline `label`, risk `title`. Don't relax it for any field.

### Who reads this

A small-business owner, founder, principal, or department head. They:

- **Know**: their own business. Revenue, costs, deals won and lost, team hours, customers, day-to-day operations. They have language for all of it — usually different from yours.
- **Don't know**: finance vocabulary (NPV, IRR, BCR, payback period, EBITDA, COGS), modelling concepts (sensitivity, attribution, counterfactual capture), consulting jargon (uplift, leverage, value framing, throughput, capacity, synergy), or notation that requires effort (Δ, basis points, multi-clause formulas).

The page's voice should sound like a competent colleague talking to them at lunch. Not a McKinsey deck. Not a research paper. Not the Excel ribbon. The math is auditable behind a *Show the math* toggle, so the surface text doesn't need to prove rigour — it needs to be readable.

### Vocabulary to drop, and what to use instead

| ✗ Don't write | ✓ Write |
|---|---|
| Uplift | Lift, raise, increase, more |
| Optimise | Improve, raise, fix |
| Leverage | Use, lean on, take advantage of |
| Drive (as verb) | Make, cause, produce |
| Capture / accrue | Get, keep, earn |
| Realise (as in "realise value") | Show up, land |
| Operationalise | Put into practice, use it |
| Throughput | Deals won, hours billed, units sold — name the actual unit |
| Capacity | Hours, time, people |
| Engagement | Project, deal, contract — whatever they call it |
| Stakeholder | Owner, manager, client, sponsor — name the role |
| FTE / headcount | People, employees, team members |
| KPI / metric | Number, target, result |
| Workflow / process | The way you do X |
| Pipeline | Deals in progress, future work |
| Cross-functional / matrixed | Across the team |
| Holistic / end-to-end | (don't — they're filler) |
| Scalable | Works as you grow |
| Mission-critical / best-in-class | (don't — they're filler) |
| Buy-in | Support from the people involved |
| Loaded rate | Your hourly cost |
| Cost basis | Hours × rate, what it costs you to do it |
| Value-framing / outcome-anchored | Selling on results, charging for outcomes |
| Adoption | Whether your team actually uses it |
| Counterfactual | What would happen anyway |
| Baseline | Current, today's, where things sit now |
| Synergy | (just don't) |
| Δ, ↑, pp, bps | Spell it out: "change in", "increase of", "percentage points", "hundredths of a percent" |
| NPV / BCR / IRR / discounted cashflow | Don't use these at all. The page is nominal cashflows; the headline is "you net $X over the next {horizon}" plus "payback at period N". |

### Naming patterns

**Item names** read literally — they appear in the Then table, in audit-trail rows, and in subtotal labels. Write a short sentence the buyer would say out loud.

| ✗ | ✓ |
|---|---|
| Pricing uplift from value framing | Higher prices on each deal you win |
| Win-rate uplift from rigour signal | Winning more deals |
| Onboarding time saved on won deals | Less time aligning scope at kickoff |
| Mid-engagement rework avoided | Less mid-project rework |
| Premium inbound from reputation | Better leads coming in |
| Lower marketing cost via referrals | Lower marketing spend |
| Y1 process setup | One-time setup in year 1 |
| BC authoring time per proposal | Time spent on each business case |

**Assumption labels** appear in NOW rows (world facts the buyer confirms) and AND rows (commitments the buyer acknowledges), each next to an editable value field. The label should read as "the thing this number measures" — in the buyer's words.

| ✗ | ✓ |
|---|---|
| Baseline win rate | Current win rate |
| Average engagement fee | Typical deal size |
| Principal loaded rate | Your hourly cost |
| Pricing uplift from value framing | Price lift on each won deal |
| Win-rate uplift (percentage points) | Win-rate increase |
| Onboarding hours saved per won deal | Kickoff hours saved per deal |
| BC hours per proposal | Hours per business case |
| Referral velocity uplift | Increase in referrals |
| Annual marketing cost reduction | Marketing spend saved |
| One-off setup hours (Y1) | Year-1 setup hours |

**Risk titles** state in plain language *what could actually go wrong*. Not what gets falsified. Not what metric drifts. What happens in the buyer's world on a Tuesday morning.

| ✗ | ✓ |
|---|---|
| Implementation effort exceeds the steady-state estimate | Writing business cases keeps taking longer than 4 hours |
| Sophisticated buyers reject value framing | Buyers refuse to pay on outcomes — they want a day rate |
| Reputation effects don't compound | Nobody outside the engagement notices the methodology |
| Selection bias in the BC filter | We use the BC to talk ourselves into a bad engagement |
| Adoption risk on the new workflow | The team doesn't use it after the first month |

The bad versions describe a metric or a hypothesis. The good versions describe an event.

**Descriptions** — `description` on assumptions, `desc` on items — are 1–2 sentences shown on hover/focus and in popovers. Plain language explaining the actual mechanism. Not marketing copy. Not a rationale paragraph.

| ✗ | ✓ |
|---|---|
| Pricing uplift driven by value-anchored conversations leveraging methodological rigour. | Clients pay more when the conversation is about outcomes, not hours. Typical lift on won deals is 12–18%. |
| Optimises proposal authoring throughput via templated frameworks. | Each business case takes ~4 hours once you have a few templates. The first few take 6–8. |
| Engagement value will be captured at higher fee levels post-implementation. | You'll charge more per deal. |

### Active voice, concrete subject

The buyer prefers sentences where someone *does* something.

| ✗ Passive / abstract | ✓ Active / concrete |
|---|---|
| Engagement value will be captured | You'll charge more per deal |
| Operational efficiencies are realised | You spend less time on kickoff |
| Adoption risk threatens benefit accrual | If the team doesn't use it, the savings don't show up |
| Reputation effects compound over time | Word-of-mouth brings in better-quality leads |
| Cash savings are predicated on overlap mitigation | These savings only land if another initiative isn't already booking them |

### Stress test before declaring done

Read every assumption label, item name, and risk title out loud. Ask three questions:

1. **Would a non-financial business owner repeat this exact wording when describing the project to a friend at the pub?** If no, simplify.
2. **Does any word in it require translation?** *Uplift* requires translation. *Lift* doesn't. *Stakeholder* requires translation. *Owner* doesn't. Replace until none do.
3. **Does it describe an event that happens, or a metric that's measured?** Events win. *"Buyers refuse to pay on outcomes"* > *"Pricing elasticity failure"*.

If a string fails any of those three, rewrite before moving on. Don't ship "almost-plain" — it reads as worse than fully technical, because the inconsistency makes the buyer doubt the rest.

## Step 3 — Author `project.config.js`

Most BCs fail at the early steps, not at the math. Work through these in order.

### A. Frame the decision (ask once)

Before any numbers, four facts must be clear. If any are unclear from the user's brief, **ask all of them in a single combined message**:

- **The counterfactual.** Not "doing nothing" — usually "the cheapest credible alternative" (a different vendor, an internal team, a partial scope). The honest comparator.
- **Decision audience.** A CFO needs different rigour than a champion's slide deck. The audience shapes the voice; don't drift mid-document.
- **Who pays vs. who captures.** Agency mismatches (org A pays, org B benefits) kill otherwise-good projects.
- **What "yes" unlocks.** Often the early work is infrastructure and the later phases unlock the bigger value but are conditional.

Write the answers into `meta.description` so the framing is visible. Set `meta.shortName` — it's interpolated as the intervention's name everywhere.

### B. Pick the timescale (granularity + horizon)

The decision's natural cadence sets `granularity` and `horizon` for the whole model. **Every formula, every assumption, every benefit and cost works in this unit** — no annualising, no monthly-vs-yearly fudging mid-case.

| Granularity | When | Typical horizon | Example decisions |
|---|---|---|---|
| `"day"`     | A sprint, a triage exercise, a launch window | 14–60 days | Campaign push, incident response, hiring sprint |
| `"week"`    | A quarter-ish project, a pilot           | 6–26 weeks | Sales coaching pilot, ops re-rostering trial |
| `"month"`   | The default for SaaS / programme rollouts | 6–24 months | Tooling rollout, marketing programme, M&A integration |
| `"quarter"` | Strategic programmes, OKR-aligned bets    | 4–16 quarters | Capability build, platform migration, channel expansion |
| `"year"`    | Infrastructure, M&A, very long cycles     | 3–7 years | Plant build, multi-year contract, brand investment |

Pick the unit that matches **how the buyer thinks about the decision**, not the largest unit the model can fit. A 12-month rollout modelled in years has a single Year-1 bar and tells you nothing about the ramp — model it monthly. A 5-year contract modelled monthly has 60 columns nobody can read — model it quarterly or yearly.

`horizon` is then the *count of those units*. `granularity: "month", horizon: 18` = an 18-month case. Once set, **all flow assumptions are expressed in this unit**: if monthly, `revenue_per_period` is monthly revenue, never annual. The skill's job is to ask the user for figures in the chosen unit, or convert explicitly with an attribution note.

### C. Identify commitments vs. world facts

Every numeric input is one of two things:

- **World fact** (`controllable: false` or absent) — something *about* the buyer's business that the intervention doesn't change. Examples: proposals per year, current win rate, average deal size, hourly cost of staff. The buyer confirms these in Now.
- **Commitment** (`controllable: true`) — an outcome the intervention *moves*, by promise. Examples: pricing lift %, win-rate boost in pp, hours saved per deal, new vendor cost per year. The buyer acknowledges these in And.

Tag every assumption. Misclassifying wrecks the Now/And distinction.

### D. Define the baseline expression

Write at least one `baseline[]` entry — what the world facts imply about the business today. The Now section renders it as a live multiplication chain that fills in as the buyer confirms each input.

```js
{
  label: "Your annual revenue today",
  formula: "proposals_per_year * (baseline_win_rate / 100) * avg_engagement_fee",
  unit: "$/yr",
  kind: "revenue",
}
```

`kind: "revenue"` makes this the denominator for the *% change to annual revenue* subtotal in And. If the case is primarily cost reduction, add a second entry with `kind: "cost"` so the cost-saving subtotal can also render as a percentage.

The formula uses the same syntax as `item.gross` — a product of assumption ids, with the same sandboxed helpers.

### E. Compose benefits and costs

For every item, write the four-step value chain in `desc`:

```
project action → what changes in the world → how that becomes $ → who captures it
```

Example:

```
better evidence per dispute → more disputes contested successfully
                            → fewer rebate $ paid + faster resolution time
                            → PA opex (rebates) + AOC capacity (hours)
```

The chain is the **mechanism**. `gross` implements *that* chain — not a freelance formula. Magic numbers without an audit trail can't be defended.

**Rules of thumb:**

- **≤8 benefits** total — the And breakdown reads cleanly at that count and below.
- **Scope-1 benefits** (`scope: 1`) pay for the project on their own. Each Scope-1 benefit's `gross` must reference a commitment assumption (one with `controllable: true`) — otherwise the And section has nothing to anchor on.
- **Scope-2 benefits** (`scope: 2`) are adjacent, secondary. Real but less directly attributable.
- **Scope-3 benefits** (`scope: 3`) are downstream / strategic. Hardest to attribute.
- **Costs** have no scope. Always counted. Use `lump: true` for one-off setup costs, `lump: false` for recurring.
- Every numeric benefit declares its `benefitKind`: `"revenue_uplift"`, `"cost_saving"`, or `"qualitative"`.
- Qualitative benefits have `gross: "0"`.
- **Plain-language naming.** Every `name`, `label`, `title`, `desc`, and `description` follows the *Voice & writing rules* above. Don't relax for cost items, qualitative benefits, or scope-2/3 items — same rule everywhere.

### F. Source every belief

Every numeric assumption gets *one* of these sources, in priority order:

| Source | When | What you do |
|---|---|---|
| **Internal data** | The user's org has it | Ask them — *"Do you have FY24 dispute count? Order-of-magnitude is fine, I'll flag."* |
| **Authoritative external** | Public benchmark exists | Run `WebSearch`. Industry reports, regulator filings, peer-reviewed. Cite the URL in `source`. |
| **Vendor proposal** | Vendor stated it | Cite verbatim, flag bias: `source: "Teleios proposal v2 (vendor-stated)"`. |
| **Fermi decomposition** | No data but decomposition works | Use the structured `fermi[]` field — `[{ label: "...", value: ..., source: "..." }]` — not a single rolled-up number. The provenance popover renders it. |
| **`[CONFIRM]`** | Nobody knows yet | Flag explicitly: `source: "[CONFIRM] needs ops budget review"`. |

**Batch your questions.** One message with three precise questions beats three rounds of one-question asks. Template:

> *"To make these estimates solid I need three things you'll know: (1) ____, (2) ____, (3) ____. If you don't have one, I'll proceed with my Fermi estimate and flag it."*

**Web search template** for industry benchmarks:

> *"Find a 2023+ benchmark for `<metric>` in `<industry/geography>`. I need: (1) the typical value, (2) the range across comparable orgs, (3) a source I can cite — peer-reviewed or industry-standard preferred."*

If results are thin or contradictory, prefer the lower estimate and note the uncertainty in `description`.

**Sensitivity range.** Set `sensitivityRange: { lo, hi }` per assumption based on a *coherent low/high case*, not a generic ±25%. Lo is what a sceptical CFO would defend; hi is what the champion would defend. The page uses these for sensitivity attribution and as soft editor bounds.

### G. Critique your own items

Each item must carry **≥3 specific attacks** as comments above it in `project.config.js`. The 10 lenses (pick the 3–5 most apt per item):

1. **Counterfactual capture** — does the cheapest credible alternative do most of this anyway?
2. **Overlap** — is another initiative already booking part of this benefit?
3. **Cash vs. soft** — does the dollar leave a budget line, or is it freed time that gets reabsorbed?
4. **Adoption** — built ≠ used. What drives uptake?
5. **Phase risk** — how conditional on prior phases landing?
6. **Time-of-arrival** — what if it lands a year later than modelled?
7. **Behavioural absorption** — are gains spent on more activity rather than banked?
8. **Selection bias** — modelling the average case or the marginal case?
9. **Dependency** — what if the vendor / champion / sponsor leaves?
10. **Externalities** — costs imposed on others (training, disruption, opportunity).

Critiques must be **specific**. Bad: *"this might not deliver as expected"*. Good: *"Code 87 dispute success depends more on legal precedent in airline contracts than on data quality — better evidence is necessary but not sufficient without legal-team capacity to act on it."*

### H. Write risks

Three to five risks total. Each names *one thing that could go wrong*, in plain language a non-financial buyer would parse. **Title-only** — no mitigation, signal, trigger, or owner copy. Mitigation lives in the proposal, not the BC.

```js
risks: [
  { title: "Writing business cases keeps taking longer than 4 hours",
    locus: "commitment",
    threatens: "bc_hrs_per_proposal" },
  { title: "Buyers refuse to pay on outcomes — they want a day rate",
    locus: "world",
    threatens: "pricing_uplift_pct" },
]
```

`locus` is either `"commitment"` (the implementer is accountable) or `"world"` (the world or the buyer could introduce it). `threatens` points at the assumption id the risk would falsify; the page filters risks to those relevant to scope-1.

### I. Self-critique pass — `CRITIQUE.md`

Before declaring done, produce `CRITIQUE.md` next to `project.config.js` answering in writing:

1. *What's the weakest belief?* (Lowest source quality × biggest impact on net cumulative.)
2. *What's the most likely double-count?*
3. *What single dependency, if it slipped, would change the conclusion?*

These are the questions a CFO will ask. Better to find them yourself first.

### J. Pre-flight checklist

Hard gates before declaring done:

- `meta.shortName` set — interpolated as the intervention's name throughout.
- `meta.description` written, frames the decision (counterfactual / audience / horizon / payer-vs-capturer / what-yes-unlocks).
- 1–3 benefits tagged `scope: 1`, each referencing a `controllable: true` assumption in its `gross`.
- At least one `baseline[]` entry with `kind: "revenue"`. Add `kind: "cost"` if the case is primarily cost reduction.
- 3–5 risks defined, plain language, with `locus` and `threatens`.
- Every assumption has: `source`, `description`, `sensitivityRange`, and a `controllable` flag (true or explicit false).
- Every item has: a value-chain `desc`, `uses[]`, and ≥3 specific critiques as comments above it.
- `CRITIQUE.md` exists.
- ≤8 benefits.
- Runtime validator banner clean.

If a gate fails, fix it before telling the user it's done.

## Schema reference

### `meta`

```js
meta: {
  name: "Full project title",                        // long form
  shortName: "BC discipline",                        // interpolated everywhere
  description: "Paragraph that frames the decision.",
}
```

### `granularity` and `horizon`

```js
granularity: "day" | "week" | "month" | "quarter" | "year",
horizon:     12,    // integer count of periods in the chosen granularity
```

Together they fix the cadence of the model. The cashflow series has `horizon` slots; the headline ("over the next 18 months, you net $X") reads off this unit. **All flow assumptions are expressed per-period in this granularity** — see Step 3 B. The skill picks both once, before any assumption is written.

### `baseline[]`

Implied current-state expressions, rendered under Now as live multiplication chains.

```js
baseline: [
  {
    label: "Your annual revenue today",
    formula: "proposals_per_year * (baseline_win_rate / 100) * avg_engagement_fee",
    unit: "$/yr",
    kind: "revenue",     // "revenue" → And uses this as the % denominator for revenue_uplift items
                         // "cost"    → ditto for cost_saving items
  },
]
```

### `risks[]`

```js
risks: [
  {
    title: "Plain-language statement of what could go wrong.",
    locus: "commitment" | "world",
    threatens: "assumption_id",   // assumption this risk would falsify
  },
]
```

### `assumptions[]`

```js
{
  id: "snake_case_id",            // referenced in formula strings
  label: "Human-readable label",  // shown in Now / And rows and popovers — write for a non-financial reader
  group: "Group name",            // groups together in the assumptions grid
  value: 12,                      // numeric default
  unit: "$" | "$/yr" | "$/hr" | "%" | "pp" | "hrs" | "/yr" | "/mo" | "events" | "yrs",
  step: 1,                        // editor step
  icon: "IconDollar",             // see icons below

  source: "Short attribution.",   // [CONFIRM] / vendor / URL / Fermi summary
  fermi: [                        // optional structured decomposition; rendered in provenance popover
    { label: "Active months/yr", value: 12, source: "Calendar." },
    { label: "Proposals/month",  value: 1,  unit: "/mo", source: "FY24 pipeline log." },
  ],
  description: "Plain-language definition. What is this, and why this value? One to two sentences.",

  controllable: true,             // true → commitment (And row); false/missing → world fact (Now row)

  sensitivityRange: { lo: 0.5, hi: 2.0 },   // multipliers on `value`; drives sensitivity attribution + soft editor bounds
}
```

Icons: `IconDollar`, `IconUsers`, `IconPercent`, `IconBuilding`, `IconClock`, `IconBolt`, `IconShield`, `IconTrend`, `IconLeaf`, `IconCube`.

### `items[]`

```js
{
  id: "unique_id",
  name: "Plain-language item name.",     // a non-financial reader parses this literally
  kind: "cost" | "benefit",
  scope: 1 | 2 | 3,                       // benefit only; absent on costs

  benefitKind: "revenue_uplift" | "cost_saving" | "qualitative",
                                          // qualitative items have gross: "0"

  lump: false,                            // true → one-off in startPeriod; false → recurring through horizon (or endPeriod)
  startPeriod: 1,                         // 1-indexed period in the case's granularity. Use to defer items
                                          //   (e.g. granularity: "month", startPeriod: 6 → benefit kicks in at month 6)
  endPeriod: undefined,                   // optional 1-indexed last active period; default = horizon

  // Formula string, sandboxed. Allowed: assumption ids, math helpers
  // (pow, min, max, abs, log, sqrt, exp, floor, ceil, round, PI, E),
  // numeric literals, operators + - * / ( ) , .
  // The result is the item's value PER PERIOD in the case's granularity —
  // not per year. If granularity is monthly, gross returns monthly value.
  gross: "proposals_per_period * (baseline_win_rate/100) * avg_engagement_fee * (pricing_uplift_pct/100)",

  desc: "1–2 sentence value-chain mechanism (project action → world change → $ → who captures).",
  uses: ["proposals_per_period", "baseline_win_rate", "avg_engagement_fee", "pricing_uplift_pct"],
}
```

**Engine math.** `gross(A)` is evaluated as a per-period value. If `lump: true`, the value sits in `startPeriod` only. If `lump: false`, it repeats from `startPeriod` through `endPeriod` (or the horizon if `endPeriod` is unset). The headline is the **net cumulative** ($) over the horizon (sum of all benefits minus all costs, nominal — no discounting) and the **payback period** (the period where cumulative net first crosses zero, or `null` if it never does). **No discounting, no NPV/IRR, no risk waterfall, no cash/soft split.** If you need risk-adjusted figures, encode the adjustment into the assumption values directly (e.g. multiply `pricing_uplift_pct` by an adoption probability).

**Deferring an item.** Benefits rarely materialise the moment a project ships. Don't fudge a smaller value — defer the start with `startPeriod`:

- **Tooling reps must learn** (sales coaching, ops automation that depends on user behaviour): start 3–6 months in.
- **Habits or culture changes** (BC discipline on proposals, new review rituals): start 6–12 months in.
- **Compounding patterns** (codified playbooks, brand effects, referrals): start 12–24 months in.
- **Instant value** (a cost saving from day one, a one-off rebate): `startPeriod: 1`.

Same applies to costs that ramp on (licences rolled out one team at a time → defer to when each cohort starts; or split into multiple cost items with different start periods).

## Step 4 — Run

```sh
cd <slug>
npx live-server@1.2.2 > /tmp/live-server-<slug>.log 2>&1 &
```

Browser opens at `http://localhost:8080`. Save any file → reload → new numbers. JSX is transpiled in-browser by Babel-standalone; no build pipeline.

Share is pre-wired to `https://models.teleios.au` via `share.config.js`. Click Share → password → upload → backend returns a `/view/{id}` URL.

## Examples

- `project.config.js` — placeholder template (generic names). Overwrite when authoring.
- `examples/minimal.config.js` — smallest viable config (one cost, one benefit, one baseline, one risk).

When in doubt, copy `examples/minimal.config.js` over `project.config.js` and grow from there.

## Hard constraints

- **≤8 benefits** total — And breakdown legibility.
- **Every user-facing string passes the *Voice & writing rules* stress test.** Names, labels, titles, descriptions. No exceptions.
- **Numbers default conservative** — better defended than optimistic. The page already lets the buyer dial up via the editor.
- **Costs and benefits visually symmetric** — engine handles this; don't add custom UI.
- **Don't reinvent UI** — hover tooltips, Excel/PDF export, sensitivity attribution, scope toggle, share are all already built.

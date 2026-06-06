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
The buyer confirms the world they live in. Surfaces the **top 3 world-fact assumptions** (those with `controllable: false`, ranked by scope-1 sensitivity) and a live equation derived from `baseline[]` that shows what those assumptions imply about today (e.g. *"Your monthly revenue today"* = deals × win-rate × fee = $25k/mo, in the case's granularity). Each row has a *Sounds right* button. Once all three are confirmed, a *Let's proceed* button reveals the rest of the page.

### And
What you commit to change. Surfaces the **top 3 commitment assumptions** (those with `controllable: true`, ranked by scope-1 sensitivity). Each row has an *Okay* button. Beneath, two subtotals derived from the scope-1 quantitative benefits:
- A **% change to recurring revenue** (sum of `revenue_uplift` items as a fraction of the `kind: "revenue"` baseline) with the per-period dollar lift alongside.
- A **per-period cost saving** (sum of `cost_saving` items).

Both subtotals are read off the case's granularity (a monthly case shows monthly lift; a yearly case shows annual). A *Show the math* toggle reveals the per-benefit multiplication chains.

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
// Monthly case
{
  label: "Your monthly revenue today",
  formula: "deals_per_period * (baseline_win_rate / 100) * avg_deal_size",
  unit: "$/mo",
  kind: "revenue",
}

// Yearly case
{
  label: "Your annual revenue today",
  formula: "proposals_per_period * (baseline_win_rate / 100) * avg_engagement_fee",
  unit: "$/yr",
  kind: "revenue",
}
```

`kind: "revenue"` makes this the denominator for the *% change to recurring revenue* subtotal in And. The subtotal renders in the case's granularity (`/mo`, `/qtr`, `/yr`, etc.) — pick a label that says *which* period. If the case is primarily cost reduction, add a second entry with `kind: "cost"` so the cost-saving subtotal can also render as a percentage.

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
5. **Sequencing dependency** — is this benefit only available *after* earlier work in the project lands? (If yes, defer it with `startPeriod` and write the dependency into `desc`.)
6. **Time-of-arrival** — what if it lands a year later than modelled?
7. **Behavioural absorption** — are gains spent on more activity rather than banked?
8. **Selection bias** — modelling the average case or the marginal case?
9. **Dependency** — what if the vendor / champion / sponsor leaves?
10. **Externalities** — costs imposed on others (training, disruption, opportunity).

Critiques must be **specific**. Bad: *"this might not deliver as expected"*. Good: *"Code 87 dispute success depends more on legal precedent in airline contracts than on data quality — better evidence is necessary but not sufficient without legal-team capacity to act on it."*

### H. Identify risks — the assumption-based guideword sweep

Don't brainstorm risks. Brainstorming surfaces the risks everyone already sees and misses the ones that move the decision. **Interrogate the model you just built.** Every risk is a *named failure mode of a load-bearing assumption* — the model tells you where to look and how much each candidate is worth. The buyer-facing output stays exactly as lean as before: **title-only, grouped by locus, no mitigation.** The sweep produces a richer object underneath (the Risk Event Card anatomy) that is carried author-side and stripped at share time — the recipient never receives it.

#### H.0 — Draw the materiality line (read the tornado)

Materiality is **`computeSensitivity(...).range`** — the dollar net-swing across each assumption's `sensitivityRange`. **Do NOT hand-derive it:** `sensitivityRange.lo`/`.hi` are *multipliers* on the value (`{lo:0.5, hi:2.0}` is a 4× span), and materiality folds in both the range AND how hard the model leans on the input — read it off the rendered tornado, never reconstruct it.

1. Rank assumptions by sensitivity (the tornado does this).
2. **Load-bearing line:** keep every scope-1-relevant assumption whose net-swing is ≥10% of the case's primary benefit, capped at the top ~6. If a single assumption dominates, lower the line until ≥3 inputs are in scope and flag *fragile-by-concentration* in `CRITIQUE.md`.
3. Write the kept set in rank order — this is your **load-bearing universe** and your control flow. State it: *"These N assumptions carry the case. I'll name a failure mode for each; the rest I consciously don't interrogate."*

#### H.1 — The guideword sweep (HAZOP by source)

For each load-bearing assumption, sweep the fixed guideword set, grouped by **source**:

| Source | Guidewords (the question each asks of the assumption) |
|---|---|
| **intervention** (the idea is wrong) | `weaker-or-absent` (the lever barely/doesn't move the outcome) · `already-happening` (it'd arrive anyway) · `double-counted` (booked elsewhere) · `won't-bank` (real but never reaches the P&L) · `can't-measure` (we can't observe/confirm it; gameable) · `second-order-cancels` (a side-effect eats it) |
| **execution** (we can't deliver/adopt) | `costs-more` · `arrives-late` · `won't-adopt` · `can't-staff` · `gets-deprioritised` (loses funding/sponsorship before value lands) · `degrades` (decays across the horizon) |
| **environment** (the world shifts) | `demand-shifts` · `competitor-responds` · `rules-change` · `input-cost-moves` · `counterparty-fails` |

Visit every cell for every load-bearing assumption; most cells are "not material / not plausible here" and saying so is the point. **An empty source bucket is a prompt, not a pass** — re-sweep it once. If still empty, that IS the answer: record a one-line reason in `CRITIQUE.md` (e.g. *"pure internal-efficiency case — no external party can move these numbers"*). **Never fabricate a risk to fill a bucket** — a pure-internal initiative should legitimately be all-preventable.

**Then run the FRAME sweep ONCE over the whole case** (risks that aren't a movement in any existing assumption):
- `omitted` — a value-driver with no assumption at all (name the missing variable).
- `coupled` — two+ load-bearing assumptions that fail together via one common driver (one recession, one platform, one counterparty). Set `threatensAlso: [the second id]`; the downside isn't the sum of independent swings.
- `irreversible` — a cost that can't be unwound once committed.
- `outside-view` — the base rate for this archetype ("projects of this kind usually disappoint by X%").

#### H.1a — Archetype priming

Pick the initiative archetype and append its primers to the relevant source buckets (they bias the sweep, they don't replace the base set). `(int/exec/env)` = the source it lands in:

- **acquire** — synergy-realization(int) · culture-clash(exec) · key-person-flight(exec) · integration-overrun(exec) · diligence-gap(int) · overpayment(int) · customer-attrition(env)
- **build** — scope-creep(exec) · tech-risk(int) · adoption(exec) · maintenance-tail(exec) · obsolescence(env) · build-vs-buy-inversion(int)
- **buy-vs-build** — vendor-lock-in(env) · total-cost-of-ownership(int) · capability-erosion(exec) · vendor-viability(env) · customisation-gap(exec)
- **market-entry** — demand-validation(int) · regulatory-barrier(env) · channel-access(exec) · incumbent-response(env) · localisation-cost(exec) · ramp-time(exec) · unit-economics-don't-translate(int)
- **transformation** — change-fatigue(exec) · benefits-leakage(int) · transition-disruption(exec) · sponsor-loss(exec) · sequencing-dependency(exec) · baseline-drift(int)
- **pricing-or-GTM** — elasticity-miss(int) · competitor-matching(env) · channel-conflict(exec) · mix-shift(int) · discount-leakage(exec) · segment-backlash(env)
- **marketing-campaign** — participation(exec) · incrementality(int) · backlash-tail(env) · platform-dependency(env) · creative-fatigue(exec) · attribution-error(int)
- **cost-takeout/efficiency** — savings-rebanked(int) · baseline-inflated(int) · capacity-not-redeployable(int) · one-off-not-recurring(int) · morale-attrition(exec) · volume-grows-anyway(env)
- **compliance** — scope-expands(env) · enforcement-uncertainty(env) · over-engineering(exec) · no-do-downside-mismeasured(int)

The last two — **cost-takeout/efficiency** and **compliance** — are CBAgent's most common shapes; the growth archetypes actively mislead on them. No archetype match → sweep the base set only. Spans two → append both and de-dup in the judge step.

#### H.2 — Generate candidates with diversity (the non-obvious pass)

The first failure mode you think of is the obvious one, and obvious risks are low-value. Run these scaffolds **in your own reasoning — no tool, no API, no embedding; the clustering is your own judgment in-session.** For each load-bearing assumption:

**(a) Sample distinct mechanisms.** *"List 3 distinct ways `<assumption>` reaches its sceptical-CFO `lo` case. Make at least one a mechanism a domain expert would call surprising — not the first thing said in the room. For each, name the specific real-world event (a Tuesday-morning thing, not a metric drifting) that would tell us it's happening."* This pushes past the obvious; it is NOT a likelihood estimate — `likelihood` stays `null`.

**(b) Rotate ordinary-stakeholder lenses.** Look at the same assumption through each of these **ordinary operating roles** (not celebrity/genius personas — those collapse to the archetype), feeding each the assumption's `description`/`rationale`/`source` and the case `meta`:
- the frontline ops lead who makes it work day-to-day,
- the sceptical CFO who's seen this promised before,
- the customer on the other end of the change,
- legal/compliance counsel,
- the *named* competitor who'd be hurt most (use a proper noun),
- the *specific* regulator/standard that governs this domain (name it).

Each gets one sentence. Discard immaterial ones; keep distinct events.

**(c) Anti-fixation.** *"List them all. Force each genuinely distinct — no two share a mechanism. Where two collapse, keep the sharper phrasing. Where I wrote a metric instead of an event, rewrite it as the event."*

**(d) Judge to a diverse material subset.** *"Cluster by underlying mechanism. From each cluster keep the single sharpest, most material one. Keep the set small, distinct, and spanning the case's real exposure — maximise spread across MECHANISMS (not a source quota). Preserve at least one tail/non-obvious risk even if a more-obvious candidate scores higher on bare materiality — the page should carry the buyer's blind spot, not just confirm their fears."*

#### H.3 — Screen: materiality × decision-relevance

A candidate survives only if BOTH hold:
1. **Material** — threatens a load-bearing assumption that appears in the tornado (numeric, has a `sensitivityRange`). If a persona surfaced a risk targeting a below-line or non-numeric input, re-point `threatens` at the load-bearing numeric assumption the failure actually moves, or drop it.
2. **Decision-relevant** — *would a buyer who believed this could happen still say yes?* This admits low-probability/high-impact tail risks that don't flip the expected case but would flip a risk-averse buyer.

#### H.4 — Classify (source + Kaplan–Mikes category)

- `source` — `intervention`|`execution`|`environment`, from the guideword's column. Drives the fingerprint.
- `category` — Kaplan & Mikes class: `preventable` (internal, no upside → rules/controls), `strategy` (taken for return → monitor), `external` (uncontrollable → scenario). The category-routing UI is deferred, but author it now so it's ready.
- **`locus` is NOT authored** — the engine derives it from the threatened assumption's `controllable` flag. Leaving it off is fine.

#### H.5 — Author the Risk Event Card

Each survivor → one `risks[]` entry. The buyer page reads **only `title`**; everything else is author-side and stripped at share time.

```js
risks: [
  {
    title: "Buyers refuse to pay on outcomes — they want a day rate",  // buyer-visible, plain, an EVENT not a metric
    threatens: "pricing_uplift_pct",        // the assumption it falsifies = the strategic objective
    source: "environment",                  // intervention|execution|environment (set explicitly for intervention)
    category: "external",                   // preventable|strategy|external
    guideword: "demand-shifts",             // audit trail
    // threatensAlso: ["input_cost_idx"],   // only for `coupled`/common-cause risks
    outcomes: "Deals close on a day rate; the price-lift line goes to zero.",   // author-side only, stripped at share
    signposts: ["Two of next five RFQs specify day rates", "procurement asks for a rate card"],  // author-side only
    likelihood: null,                       // 1–5 BUYER-rated; NEVER fabricate. null = unrated.
    // likelihoodPrior: 2,                   // optional author guess — never counts as assessed/critical
  },
]
```

Impact is NOT stored — it's read live off the tornado (sensitivity of `threatens`). Prefer **one risk per assumption per source**; a second on the same assumption adds to the count but not the exposure bar (value-at-risk is counted once per assumption). For an assumption you consciously clear, add `{ threatens: "<id>", noMaterialRisk: true }` (no title — it never renders or counts; it just satisfies coverage).

#### H.checkpoint-1 — Author confirms materiality & plausibility *(human-in-the-loop)*

Batch the candidate list to the author: *"From your model, these failure modes each put net at risk (ranked by your own sensitivity numbers): (1)… (2)… Two questions: are any not plausible in your world — drop them? And is there one I missed that keeps you up at night?"* Apply edits; run any addition back through H.2–H.4. **At most one round of additions.**

#### H.checkpoint-2 — Buyer rates likelihood *(human-in-the-loop)*

`likelihood` is the buyer's judgment, never yours. Leave every `likelihood: null` unless the author gives a specific 1–5. A risk with `likelihood: null` is correctly "impact-known, likelihood-unrated" — the honest state. A risk becomes **critical** (the dashboard count) only when it threatens a load-bearing scope-1 assumption AND the buyer rated likelihood ≥3.

#### H.6 — Stop & coverage self-check

Done when: every load-bearing assumption has a named risk or a `noMaterialRisk` mark; every source bucket is swept (empty buckets justified after at most one re-sweep, not defaulted); the frame sweep ran; 3–5 risks survive with at least one tail risk preserved; and the highest-sensitivity scope-1 assumption has a named risk. Run the coverage prompt: *"My highest-sensitivity assumption is `<top tornado id>` — does it have a risk? If the most load-bearing input has none, that's confirmation bias, not safety. Re-sweep it."* The studio shows the same gap as an author-only prompt — never to the buyer.

**Why this shape (12-factor / diversity):** F8 fixed ordered sweep with bounded re-entry (one re-sweep per bucket, one round of additions). F3 you feed your own reasoning only the ranked assumptions + guidewords + archetype + each assumption's description/rationale. F4 every risk is a validated structured object. F7 two human checkpoints — which risks are material/plausible, and the buyer's likelihood; you fabricate neither. F5/F12 risks live in `project.config.js`; `computeRiskModel` is the stateless reducer. Diversity: verbalised distinct-mechanism sampling, ordinary-persona rotation, anti-fixation, and in-session self-judging dedup are the no-API substitute for surfacing the tail risks that change the decision.

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
- 3–5 risks defined via the Phase-H guideword sweep — each with `title` + `threatens`, `source` + `category` set. **Coverage gate:** every load-bearing assumption (net swing ≥10% of primary benefit) has a named risk or a `noMaterialRisk` mark; the single highest-sensitivity scope-1 assumption is covered; the studio coverage prompt is clear; every source bucket was swept (empty ones justified in `CRITIQUE.md`, not fabricated).
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
    // Label, formula, and unit are all per-period in the case's granularity.
    // Monthly case → "Your monthly revenue today", "$/mo".
    // Yearly case  → "Your annual revenue today", "$/yr".
    label: "Your monthly revenue today",
    formula: "proposals_per_period * (baseline_win_rate / 100) * avg_engagement_fee",
    unit: "$/mo",
    kind: "revenue",     // "revenue" → And uses this as the % denominator for revenue_uplift items
                         // "cost"    → ditto for cost_saving items
  },
]
```

### `risks[]`

```js
risks: [
  {
    // ── shown to the BUYER (title-only) ──
    title: "Plain-language statement of what could go wrong.",   // REQUIRED, an EVENT not a metric
    threatens: "assumption_id",      // REQUIRED — assumption it falsifies; impact = its sensitivity
    // ── categorization (optional; engine derives if absent) ──
    source: "intervention" | "execution" | "environment",  // drives the fingerprint; set explicitly for intervention
    category: "preventable" | "strategy" | "external",     // Kaplan–Mikes governance class
    guideword: "demand-shifts",      // audit trail of which sweep cell produced it
    threatensAlso: ["other_id"],     // only for `coupled`/common-cause risks
    // ── Risk Event Card body — AUTHOR-SIDE ONLY, stripped at share time ──
    outcomes: "What happens to the value if it fires.",
    signposts: ["leading indicator A", "leading indicator B"],
    likelihood: null,                // 1–5, BUYER-rated; null = unrated. NEVER author-fabricated.
    // likelihoodPrior: 2,           // optional author guess — never counts as assessed/critical
    // owner: "name",                // accountable; proposal-side
    // noMaterialRisk: true,         // with threatens only (no title) → marks an assumption consciously cleared
  },
]
// `locus` is NOT authored — the engine derives it from threatens.controllable.
// `id` is auto-derived; set it only if you want it stable across edits (buyer likelihood ratings).
```

### `assumptions[]`

```js
{
  id: "snake_case_id",            // referenced in formula strings
  label: "Human-readable label",  // shown in Now / And rows and popovers — write for a non-financial reader
  group: "Group name",            // groups together in the assumptions grid
  value: 12,                      // numeric default — expressed PER PERIOD in
                                  //   the case's granularity (monthly case →
                                  //   monthly value, weekly → weekly, etc.)

  // Free-form unit string. Pick to match the case's granularity:
  //   year   → "$/yr",  "/yr",  "hrs/yr"
  //   quarter→ "$/qtr", "/qtr", "hrs/qtr"
  //   month  → "$/mo",  "/mo",  "hrs/mo"
  //   week   → "$/wk",  "/wk",  "hrs/wk"
  //   day    → "$/d",   "/d",   "hrs/d"
  // Stocks (one-off values) stay unit-free: "$", "%", "pp", "hrs", "events".
  // The UI renders any "$/<suffix>" as `$<value><suffix>` automatically.
  unit: "$/mo",

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

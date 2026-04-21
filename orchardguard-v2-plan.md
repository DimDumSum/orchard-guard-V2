# OrchardGuard V2 — Planning Document

> **Status.** Draft in progress. Sections 1 and 2 complete. Sections 3–10 pending.
> **Purpose.** Source of truth for V2 scope, architecture, and phasing. Written so Claude Code can execute from it across multiple development sessions without re-reading the originating planning chat.
> **Provenance.** Grounded in the V1 codebase audit (55 prediction models, 22 tables, Next.js + SQLite on Railway) and the product author's lived use of V1 during the 2026 Ontario apple growing season.

---

## 1. Product Vision and Scope

### 1.1 Thesis
**OrchardGuard V2 closes the loop that V1 left open.** V1 assembled the parts of a decision-support system — prediction models, a product catalogue, a spray log, an inventory, a weather pipeline — but the parts don't talk to each other at recommendation time. V2's single job is to connect them into a working cycle: the grower sees what's happening, what it implies, which of their products and rules fit, and a record that updates the next recommendation.

### 1.2 What V2 Is
A decision-support system for certified Ontario apple growers running conventional or IPM programs. V2 surfaces risk, deficiency, and pressure states; filters the grower's inventory against Ontario label rules and resistance-management best practices; and maintains a spray and amendment history that feeds back into the next recommendation. It advises; the grower decides.

### 1.3 What V2 Is Not
- Not a prescription service. Every recommendation shows reasoning and cites sources.
- Not a farm-management platform. Labour, post-harvest, sales, and certification reporting are out of scope.
- Not an organic-apple advisor. Organic-specific workflows (OMRI filtering, organic-only rotation) are out of V2.
- Not a general fruit-crop advisor. Apple only.
- Not a public SaaS. V2 launches to a small known set of certified growers under a shared-responsibility posture.

### 1.4 Target User
A certified Ontario apple grower — holds a current Ontario Grower Pesticide Safety Certificate or Farmer Pesticide License. This regulatory floor means:
- The user can read and interpret product labels.
- The user is legally responsible for every application they make.
- The user treats the tool as an aid, not a substitute for their own judgment.

The primary user for V2 is the project author. Beyond that, V2 is offered to a small set of personally known, certified growers under an invited-beta posture. Public signup is out of scope for V2.

### 1.5 The Closed Loop
V2's core flow, executed continuously across the season:

1. **Prediction.** Weather + per-block orchard state + nutrition state (when present) → faithfully implemented published models (Mills, CougarBlight/MaryBlyt, DD accumulations, Gubler–Thomas, OMAFRA leaf-sufficiency ranges, etc.) → per-block risk state (disease, pest, abiotic), deficiency state (nutrient), and pressure state (weed, where scouted).
2. **Trigger.** Predictions are watched for state transitions. Trigger sources include weather-driven risk events ("scab infection window open, closes in N hours"), DD milestones ("codling moth 1B peak approaching"), phenology stages ("pink — boron window"), deficiency signals from test results ("leaf calcium below sufficiency"), and scouted symptoms ("yellowing consistent with magnesium deficiency").
3. **Advisor.** A trigger + block context (variety, rootstock, phenology, harvest date, recent sprays and amendments) → a candidate set of products filtered through rules.
4. **Rules.** Composable: Ontario registration, label growth-stage restriction, seasonal max, FRAC/IRAC/HRAC rotation, PHI vs. projected harvest, REI vs. permitted work types, tank-mix compatibility, inventory availability. Soil amendments have a reduced subset (no PHI/REI; timing, rate, method, adjacency with spray programs).
5. **Records.** Every spray, every amendment, every scouting observation logged updates rotation state, seasonal counts, PHI timers, inventory, and — for nutrition — the running picture of what has been applied against which deficiency.

Everything V2 builds serves one of these five stages. Anything that doesn't is a candidate to be cut, preserved as-is from V1, or deferred.

### 1.6 Scope — In

**Closed-loop core (rebuilt in V2):**
- Block-aware prediction models (V1 is orchard-wide; V2 requires per-block state)
- Trigger / event layer (does not exist in V1)
- Rule engine (does not exist in V1; scattered checks at best)
- Advisor UI surfaces (dashboard, per-model detail, "what should I do next")
- Closed-loop spray log (write + read by models and advisor)
- Inventory-aware recommendations (V1 inventory is orphaned)
- Ontario-label-accurate product catalogue with ingestion workflow
- Per-block expected harvest date (required for PHI enforcement)
- Multi-farm architecture from day one (data isolation, per-farm orchard state)

**Nutrition as a closed-loop participant:**
- Soil tests, leaf tests, and fertilizer/amendment logs (preserved from V1, upgraded to feed the advisor)
- Nutrition triggers: stage-driven (primary — works without any test data), test-driven (when recent soil/leaf data is present), symptom-driven (via scouting observations)
- Foliar nutrient products: treated like any other spray — full label data, PHI, REI, tank-mix compatibility, rule engine participation
- Soil amendments (lime, gypsum, granular blends): treated as a parallel product class with a reduced rule set
- Source of truth: OMAFRA Publication 360 / Crop Protection Hub nutrition sections as the primary target, not exclusive — better sources can be substituted if identified
- Organic amendments (compost, fish emulsion, kelp) are **out of scope** for V2 catalogue
- V2 functions fully without any nutrition data; nutrition adds recommendations when data is present

**Weeds and orchard floor as a closed-loop participant:**
- Herbicide catalogue with HRAC groups, full rule engine participation
- User-initiated weather-window selector (not a daily-running model) for application timing
- Weed scouting log (species, density, growth stage, per block)
- Trap and photo prompts for pre-emergent and post-emergent timing

**Preserved from V1 (kept, ported as-is or lightly refactored unless the closed loop demands change):**
- Irrigation water balance and logging
- Scouting data capture: pheromone trap counts, visual scouting observations, scouting photos
- Weather pipeline, extended to integrate (a) weather-station hardware on-orchard and (b) commercial/public API sources — V1 has Open-Meteo + Env Canada; V2 broadens this

**Production systems:** Conventional and IPM.

### 1.7 Scope — Out
- Organic production programs
- Non-apple crops
- Post-harvest tracking (storage conditions, packing, fruit inventory)
- Harvest labour, yield tracking, sales, crew management, worker rosters
- CanadaGAP and organic-certification audit exports
- Tier-4 static informational "models" (mosaic virus, voles, deer, SWD-as-placeholder) — at most demoted to reference content; not treated as prediction models
- Expanded financial tracking of spray or amendment costs beyond what V1 already captures
- Public signup, self-serve onboarding, marketing site
- Lab-PDF ingestion of tissue tests (manual entry only)
- Automated ML-based stage classification from photos (photo prompts are for user confirmation, not automated inference)
- Mobile application (V2 exposes an API surface a future mobile app could consume; V2 does not build the app)

### 1.8 Success Criteria
Goals for end of 2027 season (first full season on V2). Targets marked **TBD** need calibration after the 2026 V1 season produces a baseline.

| # | Criterion | How measured |
|---|-----------|--------------|
| 1 | Every scab / fire-blight / codling-moth infection window was acted on within its actionable window | Retrospective review of model output vs. spray-log timestamps; zero missed Tier-1 windows |
| 2 | Zero FRAC/IRAC/HRAC rotation violations across the season | Rule engine records every applied product and its group; season-end report shows no back-to-back single-group violations beyond label allowance |
| 3 | Spray count reduced vs. 2026 V1 baseline, with no disease or pest outbreak | Spray-count delta and season-end incidence scouting; target reduction = **TBD after 2026 baseline** |
| 4 | Time spent label-checking per recommendation reduced vs. V1 | Self-reported time tracking on a representative sample; target reduction = **TBD** |

Notes:
- "Acted on" is ambiguous between *the model predicted it* and *the grower responded to it*. V2 instruments both separately. The prediction metric tests the algorithms; the action metric tests the whole system including trigger delivery and attention.
- These metrics cannot all be calibrated until the 2026 V1 season produces a baseline. Generating that baseline is an explicit purpose of 2026 V1 use.

### 1.9 Launch Posture and Liability
V2 is architected for multi-tenant operation but launches in a **limited-audience, shared-responsibility posture**:

- Invited users only. No public signup. Each invited user is a certified Ontario grower the project author knows personally or via direct introduction.
- Every recommendation surface shows (a) a source citation (OMAFRA Crop Protection Hub, product label, published paper, OMAFRA Pub 360 for nutrition), (b) a disclaimer that the tool is decision-support not prescription, and (c) a reminder that the legal applicator bears final responsibility.
- No formal agronomist sign-off gate is imposed on V2. Because users are certified and the audience is small and known, responsibility is shared explicitly in the ToS and reinforced at every recommendation. If V2 later broadens audience, agronomist review becomes a release gate at that point (tracked as a deferred open question).
- Label-accurate product data — including nutrient products, amendments, and herbicides — is treated as a correctness concern on par with model algorithms: wrong PHI, wrong seasonal max, wrong rate, wrong sufficiency threshold, wrong HRAC group is a bug, not a content issue. Ingestion workflow is designed accordingly (Section 5).

### 1.10 Decisions Captured in This Section

| Decision | Rationale |
|----------|-----------|
| V2 is architected multi-tenant from day one | Matches stated intent; avoids V1's "structurally multi-tenant, practically single-tenant" trap (hardcoded `orchardId=1` across V1 API routes). |
| V2 launches limited-audience, not public | Multi-tenant architecture without the review burden of public SaaS; lets V2 reach invited growers while rule coverage and correctness are still maturing. |
| V2 is block-aware throughout | V1's block schema is unused by any model; varieties, phenology, PHI, and spray history all vary by block; closing the loop requires block-level state. This is the single largest architectural commitment in V2. |
| Conventional + IPM only; not organic | Organic is a meaningfully different product set and rule set (OMRI, organic-only rotation). Including it would roughly double the label-data and rule surface. Defer. |
| Per-block expected harvest date is in scope | Required for PHI rule enforcement. V1 has no harvest date at all; this is a known gap. |
| Irrigation, scouting capture, and weather-station integration preserved from V1 | Adjacent to but not part of the closed loop; functional in V1; cutting them would regress the author's own use. On-orchard weather hardware is an explicit V2 addition. |
| Nutrition is inside the closed loop, not adjacent | V1's bolted-on nutrition pattern is exactly the "parts that don't talk" problem the thesis names. Nutrition uses the same five-stage cycle as disease and pest: triggers (stage, test, symptom) → advisor → rules → records. V2 functions fully without nutrition data; nutrition enriches recommendations when data is present. |
| Foliar nutrients treated as sprays; soil amendments as a parallel product class | Foliars share PHI/REI/tank-mix concerns with pesticides and fit the existing rule engine. Soil amendments have a reduced rule set. |
| Organic amendments out of V2 catalogue | Out-of-scope production system; keeps catalogue bounded. |
| Weeds and orchard floor are a first-class closed-loop domain | Herbicides share PHI/REI/rotation concerns with pesticides and represent a real spray-decision surface. |
| Weed/herbicide advisor is user-initiated and weather-aware, not a daily model | Most weed decisions are scouting-driven with a weather window; continuously running a weed model generates noise. User invokes; advisor evaluates current and forecast conditions. |
| Tier-4 static "models" cut or demoted to reference pages | They are grid-fillers, not models. Keeping them as models dilutes the meaning of "model" in the product. |
| Thesis = "close the loop V1 left open" | Frames every V2 decision: features inside the loop are in scope; features outside are preserved-or-cut, not focused on. |

### 1.11 Open Questions Deferred from This Section
1. **Weather-station hardware purchase and deployment.** Resolved: purchase deferred to 2027. V2 launches on V1's Open-Meteo + Env Canada pipeline. Station integration code ships in V2 but awaits hardware. Davis Vantage Pro2 is the recommended model when purchased (Section 9.10).
2. **Irrigation in V2: rebuild or preserve.** In scope; whether it ports as-is or is rebuilt is a Section 8 phasing question.
3. **Success-metric baselines.** Targets for spray reduction and time-saved require 2026 V1 baselines. Revisit Q4 2026.
4. **Public-audience expansion trigger.** What conditions (agronomist partnership? rule-coverage threshold? validated season?) move V2 from limited-audience to broader distribution. Revisit after first V2 season.
5. **Tier-4 content format.** Static markdown pages committed to repo (edit = git commit, no structured fields) recommended; confirm in Section 8 UI pass.
6. **Nutrition source-of-truth ingestion.** OMAFRA Pub 360 is the primary target; confirming authoritative current edition, micro vs. macro coverage, and whether secondary sources (Cornell, Washington State, peer-reviewed leaf-analysis literature) should blend is a Section 5/6 research task.
7. **Nutrition rule authoring.** The rule surface for pesticides (PHI, REI, FRAC, seasonal max) is well-defined. The equivalent surface for nutrition — sufficiency-band thresholds, antagonism rules, stage-gating of specific nutrients — needs enumeration before the rule engine is designed. Revisit in Section 4.

---

## 2. Full Feature Inventory

This section enumerates V2 features at feature-level granularity, organized by the five closed-loop stages plus cross-cutting concerns. Each feature carries a disposition tag indicating its relationship to V1:

- **NEW** — does not exist in V1
- **REBUILD** — exists in V1 but must be rebuilt (orchard-wide → block-aware, or write-only → read/write, etc.)
- **KEEP-AS-IS** — ported from V1 with at most light refactoring
- **UPGRADE** — V1 has a working version; V2 extends it (e.g., add a published algorithm, add sources)
- **SCOUTING-SURFACE** — V1 treats it as a model; V2 demotes to a weather-aware scouting advisory without a formal risk score (see 2.1.1)
- **CUT** — exists in V1, removed from V2
- **DEMOTE** — V1 treats it as a model or feature; V2 treats it as static reference content

Phasing (which release each feature lands in) is deferred to Section 8.

### 2.1 Prediction Layer

#### 2.1.1 Disease Models

Disposition tags in this subsection are the result of a **scope-trimming review** applied to V1's Tier-2 models. Not every V1 model carries forward as a V2 model. Three outcomes per Tier-2 model: upgrade to a published algorithm with fixtures; demote to a scouting-surface (weather-aware advisory panel without a formal risk score); or cut. The goal is a smaller number of better-supported models rather than 55 tiles of varying evidential strength. The disposition tag **SCOUTING-SURFACE** is introduced for this purpose.

| Model | V1 Tier | Disposition | Notes |
|---|---|---|---|
| Apple scab | 1 | REBUILD (block-aware) | Mills + NH ascospore curve in V1 is production-grade. Rebuild to accept block context (bloom stage per block, variety + rootstock susceptibility, spray history per block). Preserve algorithm. |
| Fire blight | 1 | REBUILD (block-aware) | CougarBlight v5.1 + MaryBlyt v7.1 in V1 is thoughtfully built. Same rebuild as scab; rootstock susceptibility is especially load-bearing (M.9 vs. G.41). |
| Frost risk | 1 | REBUILD (block-aware) | Block elevation and microclimate vary; block-level kill-temp thresholds needed. |
| Powdery mildew | 2 | UPGRADE | V1 uses temp+humidity threshold; upgrade to Gubler-Thomas UC Davis model with citations. Block-aware. Fixture-backed. |
| Sooty blotch / flyspeck | 2 | UPGRADE | V1 uses cumulative high-humidity hours; published Brown & Sutton leaf-wetness-hours model exists. Upgrade. Fixture-backed. |
| Bitter pit | 2 | UPGRADE | **Archetype for nutrition-inside-the-loop.** V1 already blends variety, crop load, calcium spray count. V2 extends with leaf-test calcium, cultural factors (nitrogen excess, vigor), stage-gated recommendations. |
| Cedar apple rust | 2 | SCOUTING-SURFACE | No strong published algorithm identified beyond rain-plus-temp-during-bloom triggers. V2 surfaces a weather-aware advisory ("conditions present for cedar rust release") with scouting guidance and OMAFRA program reference, rather than fabricating a risk score. Juniper-proximity remains a factor in the advisory. |
| Black rot | 2 | SCOUTING-SURFACE | V1's "wet period ≥9h at 15–30°C" is an estimate, not a published algorithm. V2 surfaces a weather-aware advisory tied to orchard sanitation scouting and program recommendations. |
| Bitter rot | 2 | SCOUTING-SURFACE | V1 has DD base 15°C + warm-wet counting. Thinly sourced. V2 surfaces a weather-aware advisory with variety susceptibility (Honeycrisp, Gala) prominent and scouting focus. |
| Alternaria (core rot + leaf blotch) | 2 | SCOUTING-SURFACE | V1 has thin threshold logic. Advisory with scouting guidance; no risk score. |
| Nectria canker | 2 | SCOUTING-SURFACE | Wound-infection risk depends on pruning timing and canker presence observed in the orchard, not a weather-only model. Scouting-surface with pruning-season advisory. |
| Phytophthora | 2 | REBUILD (block-aware) | Keep as model — it has defensible logic (consecutive wet days × soil temp) and real V1 irrigation-data coupling. Fixture-backed; lighter fixture burden given simple algorithm. |
| White rot, Brooks spot, Bull's-eye rot | 2 | SCOUTING-SURFACE | Summer/late-season diseases managed largely through program fungicide coverage and sanitation. Advisory content + program reference rather than per-disease risk scores. |
| Sunburn, water core, frost ring, sunscald | 2 | REBUILD or CUT | Abiotic. Sunburn and water core have real weather drivers; keep as REBUILD (lighter fixture requirement per Section 6). Frost ring and sunscald are largely post-event explanatory rather than predictive; CUT from V2 prediction layer, move to reference content. |
| Apple mosaic, apple proliferation, replant, post-harvest (static) | 4 | DEMOTE | Reference pages, not models. |

**Net count after trimming for Tier-2 diseases:** 4 surviving as models (powdery mildew, sooty blotch/flyspeck, bitter pit, phytophthora) + 2 abiotic as REBUILD (sunburn, water core); remaining Tier-2 diseases become SCOUTING-SURFACE or CUT. Tier-1 diseases unchanged (3 models). Plus bitter pit. Total disease models: ~7, down from V1's ~20+.

#### 2.1.2 Pest Models

Same scope-trimming review applied to pests. DD-based timing models (Tier 3) are mostly well-supported by published DD thresholds and share a common algorithm (the DD accumulator). Trimming focuses on demoting pests that either have no actionable DD trigger or are better managed via scouting alone.

| Model | V1 Tier | Disposition | Notes |
|---|---|---|---|
| Codling moth | 1 | REBUILD (block-aware; multi-biofix support) | Tier-1 model. Fixture-backed + scenario replay. |
| Oriental fruit moth | 3 | UPGRADE | Per-block biofix; DD base 7.2°C. Shares DD accumulator fixture. |
| Plum curculio | 3 | REBUILD | DD + petal fall timing. Shares DD accumulator. |
| Apple maggot | 3 | REBUILD | DD base 5°C; emergence timing. Shares DD accumulator. |
| Rosy apple aphid | 3 | REBUILD | DD-based; green-tip timing. Shares DD accumulator. |
| Green apple aphid, woolly apple aphid | 3 | SCOUTING-SURFACE | Managed primarily through biological control and scouting thresholds, not DD timing. Advisory with scouting guidance. |
| European red mite | 3 | UPGRADE | Add mite-day threshold action; currently DD-only. Fixture-backed. |
| Two-spotted spider mite | 3 | SCOUTING-SURFACE | Scouting thresholds drive decisions; DD timing is less actionable. Advisory. |
| Obliquebanded leafroller | 3 | REBUILD | Important in Ontario; keep as DD model. Shares DD accumulator. |
| Tentiform leafminer | 3 | REBUILD | DD-based. Shares accumulator. |
| Lesser appleworm | 3 | SCOUTING-SURFACE | Secondary pest; manage via scouting. |
| Eyespotted bud moth, winter moth | 3 | SCOUTING-SURFACE | Early-season scouting targets; no strong standalone DD case. |
| Clearwing moth, dogwood borer | 3 | SCOUTING-SURFACE | Trunk-attacking pests; trap-based scouting dominates. |
| Tarnished plant bug | 3 | UPGRADE | Important at pink/bloom; keep as model with scouting integration. |
| Apple brown bug, mullein bug | 3 | SCOUTING-SURFACE | Scouting-driven. |
| San Jose scale | 3 | REBUILD | DD-driven crawler timing is actionable. Shares accumulator. |
| European fruit scale | 3 | SCOUTING-SURFACE | |
| Apple flea weevil, Japanese beetle, apple leaf midge | 3 | SCOUTING-SURFACE | |
| European apple sawfly | 3 | UPGRADE | Petal-fall timing is critical; keep as model. |
| Brown marmorated stink bug | 3 | REBUILD | Emerging Ontario concern; DD + trap-network integration. Incorporate regional trap-network data where available. |
| Spotted wing drosophila | 4→3 | UPGRADE | Upgrade to real DD-based model, not static. |
| Apple rust mite, pear psylla | 4 | DEMOTE | Reference content. |
| Voles, deer, dagger nematode | 4 | DEMOTE | Reference content. |

**Net count after trimming for pests:** 1 Tier-1 (codling moth) + ~10 Tier-3 pests as REBUILD/UPGRADE + scouting-surfaces for the rest. Total pest models: ~11, down from V1's ~26.

**Total V2 model count after scope trimming:** ~18–20 models (disease + pest + abiotic) + bitter pit + nutrition evaluators + weed evaluators. Compared to V1's nominal 55, V2 ships with fewer, better-supported models and an explicit scouting-surface tier for conditions that don't merit a formal model.

**Biofix handling.** V1 has one biofix field (codling moth). V2 supports per-pest biofix per block. NEW.

**Pheromone trap data.** Per-block trap records (species, date, count) feed biofix detection and action-threshold evaluation. System functions without trap data; when data is present, it sharpens predictions. NEW. V2 also provides **trap deployment recommendations** (species, timing, placement, count) via an informational surface — not a model. NEW.

**Scouting-surface behavior.** A scouting-surface is a dashboard panel that displays: current weather conditions relevant to the target; scouting guidance (when to look, where, what to look for); OMAFRA program reference; coincidence alerts; variety/rootstock susceptibility when relevant. It does not produce a risk score or drive advisor recommendations directly. Instead, user-logged scouting observations of the target trigger the advisor via the standard scouting-driven trigger path (Section 4). This honestly represents what V1's thin Tier-2 models were trying to do: surface context, prompt the grower to look.

#### 2.1.3 Nutrition Models

Nutrition evaluators return a **distinct output shape** from disease/pest: nutrient status per nutrient (sufficient / deficient / excessive / antagonized), current stage window applicability, recommended actions, source citation. Section 4 resolves runner implications.

All NEW or substantially rebuilt:

- **Leaf sufficiency evaluator.** Compares leaf-test values against OMAFRA Pub 360 sufficiency bands per nutrient per stage. Flags below-range, above-range, imbalance (K:Mg ratio, Ca:B ratio).
- **Soil sufficiency evaluator.** Compares soil-test values against OMAFRA ranges. Flags pH deviation, CEC / base-saturation imbalances, macro deficiencies.
- **Stage-driven nutrition program.** Encodes the standard Ontario apple nutrition calendar (silver-tip urea / zinc, pink boron, petal-fall + cover calcium series, summer foliar programs, fall boron, fall urea for scab sanitation). Runs without any test data.
- **Symptom-driven deficiency evaluator.** Takes scouting observations (chlorosis pattern, leaf cupping, stunted shoots, bitter pit symptoms in fruit) and surfaces candidate deficiencies.
- **Nutrient antagonism evaluator.** K:Mg, Ca:B, Ca:K, P:Zn antagonism rules applied across test results and planned applications.
- **Bitter pit predictor.** Upgraded from V1's Tier-2 model; flagship example of nutrition inside the loop.

#### 2.1.4 Weed and Orchard Floor Evaluators

Weed evaluators are **user-initiated, weather-aware selectors**, not daily-running models. When the grower opens the weed advisor and picks a target (pre-emergent burn-down, post-emergent broadleaf, sucker control, etc.), the evaluator runs against current and forecast weather and returns an application-window recommendation.

| Evaluator | Disposition | Notes |
|---|---|---|
| Pre-emergent timing evaluator | NEW | Inputs: soil temperature (when available), recent rainfall, forecast precipitation. Without a weather station (deferred to 2027), soil temperature falls back to an air-temperature approximation with an explicit "estimated soil temp" disclaimer on recommendations. Output: application window with weather confidence. |
| Post-emergent window evaluator | NEW | Inputs: weed scouting observations (species, density, growth stage), air temperature, wind speed, canopy wetness, forecast rainfall. |
| Orchard floor planner | NEW | Integrates mowing schedule, mulch condition, cover-crop presence; surfaces young-tree safety considerations. |
| Weed species reference | NEW | Ontario apple-orchard weed species (OMAFRA + Ontario Weeds atlas) with resistance profiles and control options. |

#### 2.1.5 Model Infrastructure

- **Per-model error isolation.** V1 runs all models synchronously; one throw blocks the dashboard. V2 wraps each evaluation; a failing model reports its failure and the rest proceed. NEW.
- **Model run cache.** V1 has a model_cache table. KEEP-AS-IS, extended for block granularity.
- **Model citation surface.** Every model exposes its source citation, algorithm summary, and parameter set. UPGRADE (V1 has citations in code comments, not surfaced to UI).
- **Model test fixtures.** Published example scenarios for Tier-1 models, replayable as unit tests. NEW.
- **Per-orchard model parameter overrides.** Author-role users can adjust thresholds per farm (e.g., known frost-pocket block). Override is logged; the advisor surfaces an override disclosure ("threshold adjusted by farm owner"). NEW.

### 2.2 Trigger Layer

All NEW. V1 has no trigger concept.

- **Trigger definition engine.** Declarative trigger specs: source model, state transition, urgency (urgent / warning / preparation / informational), time-to-close.
- **State transition detector.** Compares current model output to previous model output per block; fires events on transitions. Runs on weather refresh and on orchard-state changes.
- **Trigger deduplication and cooldown.** Prevents re-firing the same trigger on every refresh.
- **Trigger severity model.** Urgent (action within 6 h), warning (24 h), preparation (2–7 days), informational.
- **Trigger log.** Every fired trigger is persisted with its source state, so the advisor can reason over active triggers and so post-season retrospectives are possible.
- **Trigger acknowledgment.** User can acknowledge a trigger ("I've seen this, I'll handle it / I'm dismissing it"), which suppresses re-firing without hiding the underlying risk state.
- **Coincidence triggers.** When multiple model states align on the same block in the same window (e.g., scab + fire blight infection windows overlap), a coincidence trigger fires — treated as its own trigger type, not a separate UI surface.

### 2.3 Advisor Layer

All NEW or REBUILD. V1 has disease/pest pages with ad-hoc product mentions but no advisor that reasons over triggers + inventory + rules.

- **"What should I do next" surface.** Per-block, per-farm. Lists active triggers, their recommended responses, and their actionable windows.
- **Per-trigger recommendation generation.** Given a trigger and block context, generate candidate product list by: (a) filtering catalogue to registered-for-target products, (b) intersecting with grower's inventory, (c) applying rule engine, (d) ranking surviving candidates.
- **Recommendation ranking.** Primary factors: rule compliance (hard gate), FRAC/IRAC/HRAC rotation fit, efficacy vs. target, inventory age / expiry, cost. Transparent — show the ranking factors, not just the ranked list.
- **Recommendation reasoning display.** Every recommendation shows what rules it passed, which it was close to failing, and which products were filtered out with reason.
- **Tank mix builder.** Given a set of planned targets, propose compatible tank mixes. Surface incompatibilities. Save as template. UPGRADE from V1 (which has templates but no compatibility reasoning).
- **Jar-test prompt and log.** When a proposed tank mix has unknown compatibility in the catalogue, advisor surfaces a jar-test recommendation with protocol (water volume, sequence, agitation, observation period). User logs result. Per-farm compatibility knowledge accumulates and is surfaced on future recommendations. NEW.
- **Spray plan view.** Forward-looking 7–14 day plan combining predicted triggers with recommended sprays and phenology stages. NEW.
- **Nutrition advisor.** Parallel to spray advisor. Surfaces stage-driven nutritional recommendations, test-driven corrections, symptom-driven candidates. Applies nutrition rule subset.
- **Soil amendment advisor.** Surfaces amendment recommendations (lime for pH, gypsum for Ca without pH impact, micronutrient amendments). Runs on a longer timescale (seasons, not days).
- **Weed / orchard floor advisor.** User-initiated. Surfaces application-window recommendations with weather context; applies HRAC rotation rule and young-tree safety rule. NEW.
- **REI Go/No-Go panel.** Per-block, real-time. For each block, shows: REI countdown from the most recent applicable spray; work types currently prohibited vs. permitted per label (hand harvest, hand thinning, pruning, irrigation maintenance, scouting); time until each work type unlocks. Replaces V1's scheduled-work concept. NEW.

### 2.4 Rule Engine

Categories named here; full enumeration in Section 4.

- **Ontario registration rule.** Product is registered for the target on apple in Ontario. Hard gate.
- **Growth-stage compatibility rule.** Label permits application at the current bloom stage.
- **Seasonal application limits rule.** Grower has not hit label max applications this season for this product or active ingredient.
- **FRAC / IRAC / HRAC rotation rule.** Applying this product does not violate resistance-management guidance (no back-to-back same-group, seasonal mode-of-action diversity targets).
- **PHI vs. projected harvest rule.** Pre-harvest interval fits within the window to the block's expected harvest date.
- **REI rule.** Produces a structured output consumed by the REI Go/No-Go panel: hours remaining; work types prohibited until each timestamp.
- **Tank mix compatibility rule.** Proposed mix has no known incompatibilities; pH, hard-water, adjuvant constraints respected. When unknown, surfaces jar-test prompt rather than blocking.
- **Inventory availability rule.** Grower has enough product, product is not expired, lot is usable.
- **Young-tree safety rule.** Block plantings younger than a product's label-specified tree age trigger a hard gate or warning per label.
- **Nutrition rule subset.** Sufficiency-band threshold compliance, nutrient antagonism, stage-gated nutrient application, amendment rate/timing. Authored in Section 4.
- **Rule composition and precedence.** Rules combine deterministically; hard gates fail closed; soft rules affect ranking. Section 4.

All NEW. V1 has no rule engine.

With weeds added as a fourth product/rule domain (alongside pesticides, nutrients, amendments), the rule engine is correspondingly heavier. Section 4 reads heavier than a spray-only rule engine would.

### 2.5 Record Layer

- **Spray log.** REBUILD. V1 is write-only-from-models' perspective. V2 spray log is read by prediction (scab protectedDates, fire-blight strep counter generalized to all seasonal-max products), advisor (rotation state, seasonal counts), rules (all rule categories that depend on history). Captures pesticide and herbicide applications (same table, product category distinguishes).
- **Amendment log.** REBUILD from V1 fertilizer_log. Covers foliar nutrients (in the spray log or alongside — decide in Section 3) and soil amendments.
- **Inventory.** REBUILD. V1 inventory is orphaned. V2 inventory is live: spray logging deducts stock, advisor filters by available stock, expiry warnings surface, re-order thresholds alert. Covers pesticides, herbicides, foliar nutrients, and amendments.
- **Scouting observations.** KEEP-AS-IS with extension. V1 captures observations and photos; V2 routes them into trigger inputs and deficiency evaluators.
- **Scouting photos.** KEEP-AS-IS.
- **Weed scouting log.** NEW. Per-block species, density, growth stage, notes.
- **Pheromone trap counts.** NEW. Per-block trap records.
- **Phenology photo record.** NEW. User-configurable prompt cadence — user picks on enable; prompt can be disabled entirely. User uploads branch photo and confirms or corrects the DD-based stage. Photo archive retained for retrospective accuracy review.
- **Jar-test log.** NEW. Per-farm record of tested product pairs, conditions, outcomes.
- **Test records.** KEEP-AS-IS (soil_tests, leaf_tests). Manual entry only; lab PDF parsing is out of V2.
- **Harvest date per block.** NEW. Expected harvest date per block-planting, editable as season progresses. Drives PHI enforcement.
- **Product catalogue.** REBUILD. Ontario-label-accurate with structured ingestion workflow (Section 5).

### 2.6 Alerts and Delivery

V1 has an alerts system. V2 treats alerts as the primary trigger delivery channel, not an adjacent feature.

- **Alert preferences per user per farm.** Which trigger severities, which channels (email, SMS, push), quiet hours, per-model opt-out. UPGRADE from V1 alert_prefs (promoted to a proper migrated table).
- **Alert delivery channels.** Email (Resend, retained from V1), SMS (retained), push notification for a future mobile surface (API contract defined in V2; delivery built when mobile ships). UPGRADE.
- **Alert templating.** Structured templates per trigger type with block context, source citation, one-tap link to the advisor surface. UPGRADE.
- **Alert history and acknowledgment.** Per-alert acknowledgment surfaced back to the trigger layer.
- **Cron-driven evaluation.** KEEP-AS-IS, block-aware.
- **Alert digest option.** Opt into a daily digest rather than immediate delivery for preparation-level triggers. NEW.
- **Herbicide weather-window alerts.** Opt-in. When a weed advisor has been configured with active targets, alerts fire when the application window opens. Default off. NEW.

### 2.7 Weather

- **Dual-source ingestion (Open-Meteo + Environment Canada).** KEEP-AS-IS.
- **On-orchard weather station integration.** NEW code; hardware purchase deferred to 2027 (Section 9.10). Integration infrastructure (station registration, polling, source-priority conflict resolution) ships in V2 phase 3. At launch the only registered station types are public-API sources; `on_orchard_hardware` awaits hardware purchase.
- **Per-block microclimate offsets.** NEW. Simple adjustment model (elevation lapse, known frost-pocket modifiers) applied to the orchard weather series to produce per-block series. Low-fidelity, explicit about limits.
- **Weather data reconciliation.** Multiple sources per orchard; explicit priority and conflict logging. UPGRADE of V1's source-priority dedup.
- **Leaf wetness measurement.** NEW — first-class, when a station provides it. V1 estimates leaf wetness from humidity + precip. Direct measurement is substantially more accurate for scab and fire blight.
- **Soil temperature measurement.** NEW — first-class when a station provides it. Station purchase deferred to 2027 (Section 9.10); until then, soil temperature is approximated from air temperature for pre-emergent herbicide timing and phytophthora model inputs, with explicit "estimated" disclosure on affected surfaces.
- **Historical weather archive.** KEEP-AS-IS; weather_hourly + daily view.

### 2.8 Orchard State

- **Blocks and plantings.** KEEP-AS-IS from V1 v6 schema (orchard_blocks + block_plantings). CUT v5 legacy planted_blocks.
- **Per-block bloom stage.** NEW (V1 is orchard-wide).
- **Per-block phenology auto-progression with photo validation.** UPGRADE. DD-based auto-advance; configurable photo prompt with optional disable lets the user ground-truth and correct stage estimates.
- **Per-pest biofix per block.** NEW.
- **Per-block harvest date.** NEW.
- **Per-block inoculum / disease history.** UPGRADE. V1 has orchard-wide fire_blight_history; V2 has per-block history for fire blight, scab, and summer diseases.
- **Variety susceptibility registry.** NEW. Per-variety susceptibility scores for the major diseases and pests.
- **Rootstock susceptibility registry.** NEW. Per-rootstock susceptibility scores for fire blight (critical — M.9 vs. G.41 differ dramatically), woolly apple aphid, crown rot, replant disease, winter hardiness.
- **Per-block composite susceptibility.** NEW. Blend of variety + rootstock scores per disease/pest. Feeds model risk scoring and recommendation ranking.

### 2.9 Inventory

- **Product stock with lot and expiry.** REBUILD. Live deduction on spray logging, expiry warnings, re-order alerts. Covers pesticides, herbicides, foliar nutrients, soil amendments.
- **CSV import.** NEW. Bulk population after the catalogue is populated.
- **Multi-lot FIFO.** UPGRADE. V1 has a `deductInventory` function that's never called; wire it up.

### 2.10 Irrigation (Preserved from V1)

- **Water balance engine.** KEEP-AS-IS. Per-block configuration exists already; confirm block binding.
- **Irrigation log.** KEEP-AS-IS.
- **Manual rainfall entry.** KEEP-AS-IS.
- **Phytophthora / root-rot cross-linking.** NEW. Phytophthora model reads from water balance and irrigation log for soil-wetness history.

### 2.11 Admin and Multi-Farm

- **Farm / tenant isolation.** NEW. All queries scoped by farm; no cross-farm leakage.
- **User account and authentication.** NEW. V1 has none.
- **Farm-level roles.** NEW. Owner, applicator, scout — minimum. Who can log sprays, who can edit configuration, who can acknowledge alerts.
- **Farm invitation.** NEW. Invitation flow for adding the primary author's known growers. No self-serve signup.
- **Per-farm configuration.** NEW. Location, blocks, preferred weather sources, alert routing.
- **Admin tools for catalogue curation.** NEW. UI for product ingestion, rule authoring, source-citation management. Gated to author-role users.
- **Mobile-ready API surface.** NEW. Every advisor surface, every trigger delivery, every log-write has a REST or GraphQL endpoint documented. V2 does not build the mobile app.

No demo farm. Invited users receive direct onboarding from the project author.

### 2.12 Audit and Compliance

- **Spray / amendment / herbicide audit export.** NEW. Per-block, per-season CSV or PDF export of every application with date, product, rate, target, block, applicator.
- **Rule violation log.** NEW. When a grower logs an application that the rule engine would have flagged (override), record it with timestamp and reason.
- **Model output archive.** NEW. Periodic snapshots of model state, so "what did the app recommend on June 12" is answerable at season end.

### 2.13 Reference / Educational Content

Static markdown pages committed to the repo. No structured fields; edit = git commit. Suitable for low-change-rate content.

- **Scouting guides.** KEEP-AS-IS (V1 has good content).
- **Reference image galleries.** KEEP-AS-IS.
- **Product efficacy tables.** UPGRADE. V1 has external data files; V2 moves them into the catalogue with citations.
- **Trap deployment guidance.** NEW. Per-species: when to deploy, where to place, what kind of trap, count per area.
- **Jar-test protocol reference.** NEW.
- **Demoted Tier-4 content.** Apple mosaic, apple proliferation, voles, deer, pear psylla, apple rust mite, dagger nematode, SWD-as-placeholder — linked from dashboard as "Other conditions to watch for."

### 2.14 Items CUT from V1

- Worker roster (`workers` table and API) — replaced by user/role system; REI Go/No-Go replaces worker-routed SMS.
- Soil / leaf / fertilizer pages in their V1 bolted-on form (data model preserved, UI and integration rebuilt under nutrition advisor).
- v5 `planted_blocks` legacy schema.
- Cost analysis API beyond simple per-application cost capture.
- Tier-4 static models as models (see DEMOTE).
- Scheduled work calendar concept (spray timing supersedes work planning; REI Go/No-Go panel replaces it).

### 2.15 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| Every V1 model receives an explicit disposition | Honors "grounded in what exists." Prevents silently losing work or silently shipping stale algorithms. |
| Models run in per-model isolation | V1's synchronous `runAllModels` is a correctness and availability risk. One bad model should not blank the dashboard. |
| Trigger layer is a first-class subsystem, not computed inline per page | V1's disease pages each reason about risk independently. A trigger abstraction makes alerts, acknowledgment, deduplication, and retrospective review tractable. |
| Nutrition evaluators return a distinct output shape from disease/pest | Nutrition's state space (deficient / sufficient / excess / antagonism) does not map cleanly to risk-level scoring. Section 4 resolves runner implications. |
| Weed evaluators are user-initiated and weather-aware, not continuously running | Avoids noise from daily evaluation; fits real orchard practice (grower decides to apply herbicide, then asks "when"). |
| Alerts are the primary trigger delivery channel | V1's existing alert plumbing is a real asset; V2 makes it load-bearing. |
| Leaf-wetness and soil-temperature from station hardware are first-class when available | Direct measurement materially improves several model accuracies; V1's estimations are acceptable fallback. |
| Per-block phenology auto-progression is wired; photo prompts validate it | V1 has the function, never calls it. Stale bloom stage silently breaks every bloom-gated model. Photo prompt is optional, disable-able, and prevents the advance from drifting unnoticed. |
| Multi-lot FIFO inventory deduction is wired | V1 has the function, never calls it. Same pattern. |
| Pheromone trap counts are first-class data | Ontario IPM practice depends on trap counts for biofix confirmation and action-threshold decisions. V1 does not capture them. System works without trap data; data sharpens predictions when present. |
| Trap deployment guidance is provided, even though trap use is not required | Supports growers adopting IPM practices. |
| Variety and rootstock susceptibility are combined into a per-block composite score | Rootstock fire-blight susceptibility alone (M.9 vs. G.41) makes ignoring it materially worse than a paper calendar. |
| Rule violation log is in scope | Users will override the rule engine. Capturing overrides honestly is better than pretending they don't happen, and it drives rule improvement. |
| No lab-PDF ingestion for tissue tests | Scope restraint. Manual entry is acceptable; PDF parsing is a feature of its own. |
| Per-orchard model parameter overrides are allowed, logged, and displayed | Honors real microclimate knowledge; logging and display prevents silent drift from the published model. |
| Mobile API surface is in scope; mobile app is not | Lets the future mobile effort start without V2-era rework, without expanding V2's build surface. |
| Static markdown pages for demoted Tier-4 content | Low change rate, low complexity. Wiki scaffolding is overbuild. |
| Scheduled work calendar cut; REI Go/No-Go panel replaces it | Users need to know what they can and cannot do *right now* in each block; future scheduling is not load-bearing. |
| Jar-test prompt and log are in scope | Real orchard practice; tank-mix catalogue will never be complete; surfacing the gap honestly and logging per-farm results grows useful knowledge. |

### 2.16 Open Questions Deferred from This Section

1. **Organic amendments in the catalogue.** Resolved: out of scope for V2.
2. **Variety susceptibility source.** OMAFRA has general susceptibility notes; Cornell has more granular data. Authoritative source for per-variety-per-disease susceptibility scoring is a Section 5 research task. Same question for rootstock.
3. **Demo farm for onboarding.** Resolved: cut. Invited users receive direct onboarding.
4. **Printable REI signage for block gates.** Resolved: out of scope for V2.
5. **Photo prompt default cadence.** Resolved: user picks on enable; disable-able.
6. **Weed species source-of-truth.** OMAFRA weed management publications plus Ontario Weeds atlas; research task for Section 5/6.
7. **HRAC group data source.** Published by the Herbicide Resistance Action Committee; Canadian labels carry them. Research task.
8. **Young-tree age threshold authority.** Labels specify per-product, per-age-in-years. Ingestion must capture this. Section 5.
9. **Jar-test protocol authority.** OMAFRA has guidance; several university extension services have more detailed protocols. Pick one. Low urgency.
10. **Coincidence trigger rules.** Which combinations of model states constitute a coincidence trigger, and what the recommended response is, is a rule-authoring task. Section 4.
11. **Nutrition model output shape specifics.** Distinct from disease/pest confirmed; field-level design (severity levels, units, references) is Section 4.

---

## 3. Data Model

This section specifies V2's data model at column-level detail. The target database is Postgres. Column types use Postgres conventions (`bigserial`, `text`, `timestamptz`, `jsonb`, `numeric(p,s)`). Every tenant-scoped table carries `farm_id` with a NOT NULL constraint and a row-level security policy (Section 3.15). Soft-delete is not used; deletes are hard deletes with a cascade policy defined per relationship. Timestamps are stored in UTC; the orchard's local timezone is a property of the orchard row and is applied at read time.

This section does not emit CREATE TABLE statements. It defines the conceptual schema in tabular form. Claude Code will translate it into the chosen migration toolchain (Section 9).

### 3.1 Framing Principles

- **Farm-scoped everywhere.** Every table that holds operational data carries `farm_id`. Reference data (product catalogue, variety registry, weed species registry) is global and not farm-scoped. Per-farm overrides to reference data live in separate farm-scoped override tables.
- **Block-scoped where the loop demands it.** Applications, scouting observations, triggers, model runs, and phenology are all block-scoped. Weather ingestion is orchard-scoped with per-block microclimate offsets applied at read time.
- **Unified applications, extended products.** One `applications` table across pesticides, herbicides, foliar nutrients, and soil amendments. One base `products` table with four extension tables for domain-specific label data.
- **Distinct output shapes for distinct evaluator types.** `model_runs` stores disease/pest/abiotic evaluations; `nutrition_runs` stores nutrition evaluations; `weed_evaluations` stores on-demand weed advisor evaluations. Three tables, not one, because their shapes differ meaningfully.
- **Append-only where audit matters.** Model runs, trigger events, rule evaluations, alert deliveries, and rule-violation overrides are append-only. Edits to user-authored data (applications, scouting observations) record a `updated_at` timestamp without versioning; if versioning becomes important later, it is added via a generic audit table.
- **No V1 legacy tables.** v5 `planted_blocks` is dropped. `workers` is dropped. `alert_prefs` (runtime-created in V1) is formalized as a migrated table. Weather_daily view is rebuilt for block-scoped reads.

### 3.2 Identity and Tenancy

#### 3.2.1 `farms`

Root tenant entity. One row per orchard-owning business.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| name | text NOT NULL | |
| timezone | text NOT NULL DEFAULT 'America/Toronto' | IANA timezone; applied at read time for local-time displays. |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| archived_at | timestamptz NULL | When set, the farm is read-only and excluded from cron jobs. Not a soft-delete — archived farms stay for audit. |

A farm may contain one or many orchards. V1 has no farm-level abstraction; V2 introduces it because multi-tenant begins here.

#### 3.2.2 `users`

One row per authenticated identity.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| email | citext NOT NULL UNIQUE | |
| display_name | text NOT NULL | |
| phone | text NULL | For SMS alerts. |
| email_verified_at | timestamptz NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| last_active_at | timestamptz NULL | |
| certification_on_file | boolean NOT NULL DEFAULT false | Author marks this true when the user has attested to their Ontario grower pesticide certification. Not a legal verification — a shared-responsibility acknowledgment. |

Authentication tokens, password hashes, MFA secrets live in auth-provider-specific tables (Section 9). Users is the application-facing mirror.

#### 3.2.3 `farm_memberships`

Joins users to farms with a role.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| user_id | bigint NOT NULL FK → users(id) ON DELETE CASCADE | |
| role | text NOT NULL | enum: `owner`, `applicator`, `scout`, `readonly` |
| invited_by | bigint NULL FK → users(id) | |
| invited_at | timestamptz NOT NULL DEFAULT now() | |
| accepted_at | timestamptz NULL | Set when the invitee authenticates and accepts. |
| UNIQUE (farm_id, user_id) | | |

Role definitions:
- **owner** — everything: manage members, edit config, log applications, override rules, manage inventory, acknowledge alerts.
- **applicator** — log applications, deduct inventory, acknowledge alerts. Cannot manage members or edit orchard config.
- **scout** — log scouting observations, upload photos, record trap counts. Cannot log applications.
- **readonly** — view everything farm-scoped. Cannot write.

A separate global role (`author`) exists outside the farms table for catalogue curation and reference-data management. See 3.10.

#### 3.2.4 `farm_invitations`

Pending invites awaiting acceptance.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| email | citext NOT NULL | |
| role | text NOT NULL | |
| token | text NOT NULL UNIQUE | One-time acceptance token. |
| invited_by | bigint NOT NULL FK → users(id) | |
| invited_at | timestamptz NOT NULL DEFAULT now() | |
| expires_at | timestamptz NOT NULL | |
| accepted_at | timestamptz NULL | Once accepted, a `farm_memberships` row is created. |

### 3.3 Orchard State

#### 3.3.1 `orchards`

Refactored from V1. A farm may have multiple orchards (separated properties); V1's hardcoded `orchardId=1` assumption is removed.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| name | text NOT NULL | |
| latitude | numeric(9,6) NOT NULL | |
| longitude | numeric(9,6) NOT NULL | |
| elevation_m | numeric(7,2) NULL | |
| microclimate | text NOT NULL DEFAULT 'moderate' | enum: `cool_wet`, `moderate`, `warm_dry`. Applied at orchard level; per-block overrides in 3.3.4. |
| juniper_nearby | boolean NOT NULL DEFAULT false | Cedar rust proximity flag. |
| preferred_weather_source | text NOT NULL DEFAULT 'open_meteo' | enum: `station`, `open_meteo`, `env_canada`. Priority at read time falls through if preferred source has gaps. |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |

V1 columns dropped at migration: `primary_varieties` (moved to block_plantings), `rootstock` (moved to block_plantings), `fire_blight_history` (moved to per-block history in 3.3.5), `bloom_stage` (moved to per-block state in 3.3.4), `codling_moth_biofix_date` (moved to per-block biofix in 3.3.6), `petal_fall_date` (moved to per-block state), `blocks` JSON (replaced by 3.3.2).

#### 3.3.2 `orchard_blocks`

Preserved from V1 v6. Schema confirmed.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | Denormalized for RLS; kept in sync with orchards.farm_id via trigger or application-layer invariant. |
| orchard_id | bigint NOT NULL FK → orchards(id) ON DELETE CASCADE | |
| block_name | text NOT NULL | e.g., "North 4 Acres", "Honeycrisp East". |
| total_area_ha | numeric(6,3) NOT NULL | |
| year_established | integer NULL | |
| soil_type | text NULL | enum matching irrigation soil_type enum for cross-linking. |
| irrigation_system | text NULL | enum: `drip`, `microjet`, `traveling_gun`, `solid_set`, `none`. |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |
| UNIQUE (orchard_id, block_name) | | |

#### 3.3.3 `block_plantings`

Preserved from V1 v6. Represents a homogeneous planting (variety + rootstock + year) within a block. A block can have multiple plantings if subdivided.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | Denormalized from block → orchard → farm. |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| variety_id | bigint NOT NULL FK → varieties(id) | See 3.9.1. V1 stores variety as free-text; V2 normalizes. |
| rootstock_id | bigint NOT NULL FK → rootstocks(id) | See 3.9.2. |
| tree_count | integer NOT NULL | |
| spacing_in_row_m | numeric(4,2) NULL | |
| spacing_between_rows_m | numeric(4,2) NULL | |
| rows_description | text NULL | |
| planted_year | integer NOT NULL | Drives young-tree safety rule (Section 4). |
| expected_harvest_date | date NULL | Per-planting because varieties harvest at different times. **Load-bearing for PHI rule.** Can be refined as season progresses; null means "PHI rule warns when any spray's PHI would extend past any plausible harvest window for this variety" (fallback from variety default). |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |

**Open design question flagged inline.** An application is currently block-scoped (see 3.5.1), not planting-scoped. Most real applications cover a whole block across multiple plantings. But PHI is per-planting because harvest date varies. The rule engine reads planting-level PHI and applies the most restrictive across all plantings in the block when an application is logged block-wide. This works but means a Gala planting's shorter PHI is held hostage to a Honeycrisp planting in the same block. The alternative — planting-level applications — matches physical reality better but is substantially more log entry work. **Decision: block-scoped applications with per-planting PHI resolution, warn when block contains plantings with PHIs that conflict with the application.** Revisit if 2026 data shows growers frequently split applications across plantings.

#### 3.3.4 `block_states`

Per-block dynamic state that varies across the season. One row per block, updated in place.

| Column | Type | Notes |
|---|---|---|
| block_id | bigint PK FK → orchard_blocks(id) ON DELETE CASCADE | |
| farm_id | bigint NOT NULL | |
| bloom_stage | text NOT NULL DEFAULT 'dormant' | enum: 8 stages from dormant through fruit_set. V1 enum preserved. |
| bloom_stage_source | text NOT NULL DEFAULT 'manual' | enum: `manual`, `auto_dd`, `photo_confirmed`. For audit of how the stage was set. |
| bloom_stage_updated_at | timestamptz NOT NULL DEFAULT now() | |
| microclimate_override | text NULL | Overrides orchard.microclimate when set. |
| photo_prompt_enabled | boolean NOT NULL DEFAULT false | |
| photo_prompt_cadence_days | integer NULL | Required when photo_prompt_enabled=true. User picks on enable. |
| photo_prompt_last_at | timestamptz NULL | |

#### 3.3.5 `block_disease_history`

Per-block, per-disease qualitative history. Replaces V1's orchard-wide `fire_blight_history`.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| disease_slug | text NOT NULL | `fire_blight`, `apple_scab`, `bitter_rot`, etc. |
| history_level | text NOT NULL | enum per disease. For fire_blight: `none`, `nearby`, `in_block`. For scab: `clean`, `moderate`, `heavy_prior_year`. Extensible per-disease. |
| notes | text NULL | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |
| UNIQUE (block_id, disease_slug) | | |

#### 3.3.6 `block_pest_biofix`

Per-block, per-pest biofix dates. Replaces V1's single `codling_moth_biofix_date`.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| pest_slug | text NOT NULL | `codling_moth`, `oriental_fruit_moth`, `apple_maggot`, etc. |
| biofix_date | date NOT NULL | |
| source | text NOT NULL DEFAULT 'manual' | enum: `manual`, `trap_threshold`. Future automation reads trap counts and sets biofix; V2 supports both but only manual is wired at launch. |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| UNIQUE (block_id, pest_slug) | | |

### 3.4 Product Catalogue

Base + extension pattern. Every product has one base row and exactly one extension row corresponding to its category.

#### 3.4.1 `products` (base)

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| category | text NOT NULL | enum: `pesticide`, `herbicide`, `foliar_nutrient`, `soil_amendment`. Determines which extension table holds the rest of the data. |
| trade_name | text NOT NULL | Canonical trade name as it appears on the current Ontario label. |
| manufacturer | text NULL | |
| pcp_number | text NULL | Pest Control Product registration number (Health Canada). Null for amendments and some nutrients which are not PMRA-registered. |
| ontario_registered | boolean NOT NULL DEFAULT true | Whether the product is currently registered for use on apple in Ontario. When set false, the product is excluded from recommendations but retained for history. |
| label_url | text NULL | Link to the canonical label PDF. |
| label_version | text NULL | Label revision or date, when the label is dated. |
| label_ingested_at | timestamptz NULL | When V2's catalogue last captured label data. |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |

No farm_id — reference data. Not subject to RLS; readable by all authenticated users. Writes gated to `author` role (3.10).

#### 3.4.2 `products_pesticide`

| Column | Type | Notes |
|---|---|---|
| product_id | bigint PK FK → products(id) ON DELETE CASCADE | |
| active_ingredients | jsonb NOT NULL | Array of {name, percentage, unit}. Supports multi-active products (Merivon, Inspire Super). |
| frac_groups | text[] NOT NULL DEFAULT '{}' | For fungicides. |
| irac_groups | text[] NOT NULL DEFAULT '{}' | For insecticides. |
| pesticide_type | text NOT NULL | enum: `fungicide`, `insecticide`, `miticide`, `bactericide`, `biological`. Combo products carry multiple in an array if needed. |
| target_pests | text[] NOT NULL DEFAULT '{}' | Slugs from pest/disease registries this product is registered for. |
| rate_min | numeric(8,3) NULL | |
| rate_max | numeric(8,3) NULL | |
| rate_unit | text NOT NULL | enum: `L_per_ha`, `kg_per_ha`, `g_per_ha`, etc. |
| phi_days | integer NULL | |
| rei_hours | integer NOT NULL | |
| rei_work_types | jsonb NULL | Label-specified work-type restrictions keyed by rei duration. E.g., `{"hand_harvest": 72, "hand_thinning": 48, "scouting": 12}`. Null falls back to rei_hours for all work types. |
| seasonal_max_applications | integer NULL | |
| seasonal_max_rate | numeric(8,3) NULL | Alternative or additional limit. |
| seasonal_max_rate_unit | text NULL | |
| min_days_between_applications | integer NULL | |
| min_tree_age_years | integer NULL | Young-tree safety rule input. Null means no restriction. |
| growth_stage_min | text NULL | Earliest bloom stage allowed. |
| growth_stage_max | text NULL | Latest bloom stage allowed. |
| kickback_hours | integer NULL | For fungicides with curative activity. |
| rainfast_hours | integer NULL | |
| resistance_risk | text NULL | enum: `low`, `medium`, `high`. |
| tank_mix_known_compatible | bigint[] NOT NULL DEFAULT '{}' | Array of product_ids. |
| tank_mix_known_incompatible | bigint[] NOT NULL DEFAULT '{}' | |
| tank_mix_notes | text NULL | Free-text conditions (pH sensitive, hard water concerns, adjuvant requirements). |
| organic_approved | boolean NOT NULL DEFAULT false | Informational; V2 does not filter for organic programs but the flag is captured. |

#### 3.4.3 `products_herbicide`

| Column | Type | Notes |
|---|---|---|
| product_id | bigint PK FK → products(id) ON DELETE CASCADE | |
| active_ingredients | jsonb NOT NULL | Same shape as pesticide. |
| hrac_groups | text[] NOT NULL DEFAULT '{}' | |
| mode_of_action | text NULL | |
| herbicide_type | text NOT NULL | enum: `pre_emergent`, `post_emergent`, `both`, `burndown`, `systemic`. |
| target_weeds | text[] NOT NULL DEFAULT '{}' | Slugs from weed species registry. |
| rate_min | numeric(8,3) NULL | |
| rate_max | numeric(8,3) NULL | |
| rate_unit | text NOT NULL | |
| phi_days | integer NULL | Some apple herbicides have PHIs; most are structured around dormant or early-season application. |
| rei_hours | integer NOT NULL | |
| rei_work_types | jsonb NULL | |
| seasonal_max_applications | integer NULL | |
| min_tree_age_years | integer NULL | Often larger than pesticides (e.g., 3 or 5 years). |
| soil_temp_min_c | numeric(4,1) NULL | Pre-emergent application guidance. |
| rainfall_trigger_mm | numeric(5,1) NULL | Minimum rainfall needed to activate pre-emergent. |
| min_hours_before_rain | integer NULL | For rainfast-sensitive post-emergents. |
| application_method | text NOT NULL | enum: `directed_spray`, `broadcast`, `spot_treatment`, `wiper`. |
| young_tree_warning | text NULL | Free-text label warning about contact with young bark. |
| resistance_risk | text NULL | |
| notes | text NULL | |

#### 3.4.4 `products_foliar_nutrient`

| Column | Type | Notes |
|---|---|---|
| product_id | bigint PK FK → products(id) ON DELETE CASCADE | |
| nutrient_analysis | jsonb NOT NULL | Array of {nutrient, percentage, form}. E.g., calcium chloride at 36% Ca; boron as sodium borate at 20.5% B. |
| primary_nutrient | text NOT NULL | enum: `N`, `P`, `K`, `Ca`, `Mg`, `S`, `B`, `Zn`, `Mn`, `Fe`, `Cu`, `multi`. |
| rate_min | numeric(8,3) NULL | |
| rate_max | numeric(8,3) NULL | |
| rate_unit | text NOT NULL | |
| phi_days | integer NULL | Generally none but some foliars have advisory intervals. |
| rei_hours | integer NOT NULL | |
| rei_work_types | jsonb NULL | |
| seasonal_max_applications | integer NULL | |
| seasonal_max_nutrient_kg_per_ha | numeric(8,3) NULL | Upper limit on cumulative nutrient delivery per season. |
| growth_stage_min | text NULL | |
| growth_stage_max | text NULL | |
| target_deficiencies | text[] NOT NULL DEFAULT '{}' | Which deficiencies this product addresses. |
| compatible_adjuvants | text[] NOT NULL DEFAULT '{}' | |
| tank_mix_known_compatible | bigint[] NOT NULL DEFAULT '{}' | |
| tank_mix_known_incompatible | bigint[] NOT NULL DEFAULT '{}' | |
| tank_mix_notes | text NULL | |
| ph_sensitivity | text NULL | enum: `acid`, `neutral`, `alkaline`, null. |

#### 3.4.5 `products_soil_amendment`

| Column | Type | Notes |
|---|---|---|
| product_id | bigint PK FK → products(id) ON DELETE CASCADE | |
| amendment_type | text NOT NULL | enum: `lime`, `gypsum`, `sulfur`, `granular_fertilizer`, `elemental_amendment`, `other`. |
| nutrient_analysis | jsonb NULL | For granular fertilizers. Null for lime/gypsum etc. |
| calcium_carbonate_equivalent | numeric(5,2) NULL | For liming materials. |
| neutralizing_value_pct | numeric(5,2) NULL | |
| rate_min | numeric(8,3) NULL | |
| rate_max | numeric(8,3) NULL | |
| rate_unit | text NOT NULL | Typically `kg_per_ha` or `tonnes_per_ha`. |
| application_timing | text[] NOT NULL DEFAULT '{}' | enum values: `dormant`, `pre_bloom`, `post_harvest`, `fall`, `spring`. |
| application_method | text NOT NULL | enum: `broadcast`, `banded`, `fertigation`, `incorporation`. |
| seasonal_max_rate | numeric(8,3) NULL | |
| interaction_notes | text NULL | E.g., "raises pH; avoid stacking with urea within 2 weeks." |

#### 3.4.6 `product_efficacy`

Per-product, per-target efficacy score. Preserved from V1's external data file; moved into catalogue with citation.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| product_id | bigint NOT NULL FK → products(id) ON DELETE CASCADE | |
| target_slug | text NOT NULL | Pest, disease, or weed slug. |
| efficacy_rating | integer NOT NULL | 1–5 (V1 uses star rating). |
| source | text NOT NULL | Citation string; typically OMAFRA Crop Protection Hub reference or university extension publication. |
| source_url | text NULL | |
| notes | text NULL | |
| UNIQUE (product_id, target_slug) | | |

### 3.5 Applications and Inventory

#### 3.5.1 `applications`

Unified log across pesticides, herbicides, foliar nutrients, and soil amendments. Every application in V2 writes here. V1's `spray_log` and `fertilizer_log` data migrate into this table.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | See 3.3.3 flag re: block-scoped vs. planting-scoped. |
| product_id | bigint NOT NULL FK → products(id) | |
| category | text NOT NULL | Denormalized from product for query performance; enforced to match products.category via trigger or application-layer check. |
| application_date | date NOT NULL | |
| application_time | time NULL | Optional. REI countdown starts here if provided; else at end-of-day UTC of application_date. |
| rate | numeric(8,3) NOT NULL | |
| rate_unit | text NOT NULL | |
| area_ha | numeric(6,3) NULL | Defaults to block.total_area_ha if null. |
| target_slug | text NOT NULL | Primary target (pest, disease, weed, deficiency). |
| secondary_targets | text[] NOT NULL DEFAULT '{}' | When one spray addresses multiple targets. |
| applicator_user_id | bigint NULL FK → users(id) | |
| applicator_name | text NULL | Free-text fallback when applicator is not a user (custom applicator, casual help). |
| tank_mix_group_id | bigint NULL FK → tank_mix_groups(id) | Non-null when this application is part of a multi-product tank mix. See 3.5.3. |
| cost | numeric(8,2) NULL | Preserved from V1. |
| notes | text NULL | |
| rule_override_id | bigint NULL FK → rule_overrides(id) | Non-null if this application was logged against a rule-engine objection. See 3.11. |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |
| created_by | bigint NOT NULL FK → users(id) | Who wrote the record. |

Indexes: `(farm_id, application_date DESC)`, `(block_id, application_date DESC)`, `(product_id, application_date DESC)`.

#### 3.5.2 `inventory_lots`

Replaces V1's `inventory` table with lot-level tracking for proper FIFO.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| product_id | bigint NOT NULL FK → products(id) | |
| lot_number | text NULL | |
| quantity_on_hand | numeric(10,3) NOT NULL | Decremented on application logging. |
| unit_measure | text NOT NULL | Matches container label unit, not necessarily application rate unit. |
| purchase_date | date NULL | FIFO sort key. |
| expiry_date | date NULL | Rule engine flags expired lots. |
| purchase_price | numeric(8,2) NULL | |
| supplier | text NULL | |
| storage_location | text NULL | |
| opened_at | date NULL | Some products degrade faster once opened. |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

A product can have multiple lots. FIFO deduction is by `purchase_date`, then `created_at`.

#### 3.5.3 `tank_mix_groups`

Groups multiple applications that were sprayed together in one tank. Represents "we mixed product A, B, and C in one tank, applied to block X on June 10."

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | |
| application_date | date NOT NULL | Redundant with child applications; kept for query ergonomics. |
| tank_size_l | numeric(6,1) NULL | |
| water_rate_l_per_ha | numeric(6,1) NULL | |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

Child applications reference `tank_mix_groups.id` via `applications.tank_mix_group_id`. Deleting a tank mix cascades to its applications; deleting a single application leaves the group intact unless it was the last member.

#### 3.5.4 `tank_mix_templates`

Preserved from V1. Reusable mix recipes the grower saves.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| name | text NOT NULL | |
| components | jsonb NOT NULL | Array of {product_id, rate, rate_unit}. |
| tank_size_l | numeric(6,1) NULL | |
| water_rate_l_per_ha | numeric(6,1) NULL | |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.5.5 `jar_test_log`

Per-farm record of tank-mix jar tests the grower has performed. Feeds back into tank-mix compatibility reasoning.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| test_date | date NOT NULL | |
| product_a_id | bigint NOT NULL FK → products(id) | |
| product_b_id | bigint NOT NULL FK → products(id) | |
| additional_products | bigint[] NOT NULL DEFAULT '{}' | For three-way or four-way tests. |
| water_source | text NULL | |
| water_ph | numeric(3,1) NULL | |
| result | text NOT NULL | enum: `compatible`, `incompatible`, `marginal`. |
| observations | text NULL | |
| created_by | bigint NOT NULL FK → users(id) | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

### 3.6 Weather

Weather is orchard-scoped at ingestion. Per-block microclimate adjustments are applied at read time, not stored.

#### 3.6.1 `weather_stations`

Registered weather data sources per orchard.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| orchard_id | bigint NOT NULL FK → orchards(id) ON DELETE CASCADE | |
| station_type | text NOT NULL | enum: `on_orchard_hardware`, `open_meteo`, `env_canada`, `custom_api`. |
| external_id | text NULL | Station ID for Env Canada, lat/lng key for Open-Meteo. |
| vendor | text NULL | Davis, Ambient, Meter, etc. |
| latitude | numeric(9,6) NULL | |
| longitude | numeric(9,6) NULL | |
| elevation_m | numeric(7,2) NULL | |
| priority | integer NOT NULL DEFAULT 0 | Higher-priority stations preferred at read-time conflict resolution. |
| active | boolean NOT NULL DEFAULT true | |
| config | jsonb NOT NULL DEFAULT '{}' | API keys, hardware endpoints, auth tokens. Secrets handled per Section 9. |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.6.2 `weather_hourly`

Upgrade of V1 table with explicit station FK and added measurements.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | Denormalized from station → orchard → farm. |
| station_id | bigint NOT NULL FK → weather_stations(id) ON DELETE CASCADE | |
| timestamp | timestamptz NOT NULL | UTC. |
| temp_c | numeric(5,2) NULL | |
| humidity_pct | numeric(5,2) NULL | |
| precip_mm | numeric(6,2) NULL | |
| wind_kph | numeric(5,2) NULL | |
| wind_gust_kph | numeric(5,2) NULL | |
| wind_direction_deg | numeric(5,2) NULL | |
| leaf_wetness_hours | numeric(4,2) NULL | Direct measurement when hardware supports it. |
| leaf_wetness_estimated | boolean NOT NULL DEFAULT false | True when the value is computed from humidity+precip rather than measured. |
| dew_point_c | numeric(5,2) NULL | |
| soil_temp_c | numeric(5,2) NULL | First-class in V2 (new). |
| soil_moisture_pct | numeric(5,2) NULL | When hardware supports it. |
| solar_radiation_w_m2 | numeric(6,2) NULL | |
| source_raw | jsonb NULL | Original payload for debugging. |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| UNIQUE (station_id, timestamp) | | |

Indexes: `(farm_id, timestamp DESC)`, `(station_id, timestamp DESC)`.

#### 3.6.3 `weather_daily`

Materialized view (or table refreshed by cron, depending on performance) aggregating `weather_hourly` by orchard local date. Replaces V1's view.

Columns: orchard_id, date_local, max_temp, min_temp, mean_temp, total_precip, avg_humidity, max_humidity, leaf_wetness_hours, degree_hours_15_5, degree_hours_18_3, degree_hours_10, degree_days_base5, degree_days_base10, soil_temp_avg.

V2 difference from V1: orchard-scoped rather than station-scoped; conflict resolution across stations happens at aggregation time using `weather_stations.priority`.

### 3.7 Models and Triggers

#### 3.7.1 `model_runs`

Append-only log of disease/pest/abiotic model evaluations per block. Used for retrospective review and the model output archive (Section 2.12).

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| model_slug | text NOT NULL | `apple_scab`, `fire_blight`, etc. |
| run_at | timestamptz NOT NULL DEFAULT now() | |
| risk_level | text NOT NULL | enum: `none`, `light`, `moderate`, `severe`, `not_applicable`. |
| risk_score | numeric(5,2) NULL | 0–100 where applicable. |
| result | jsonb NOT NULL | Full model output — preserves the shape V1 models already produce. |
| weather_range_start | timestamptz NULL | |
| weather_range_end | timestamptz NULL | |
| weather_source_id | bigint NULL FK → weather_stations(id) | |
| model_parameters_hash | text NULL | Hash of the parameters used (for override audit). |
| error | text NULL | When set, the model threw; result may be null. Ensures per-model isolation is visible in the archive. |

Partitioned by month via Postgres native partitioning for retention management.

Indexes: `(block_id, model_slug, run_at DESC)`, `(farm_id, run_at DESC)`.

#### 3.7.2 `nutrition_runs`

Append-only log of nutrition evaluations per block. Distinct from `model_runs` because the output shape differs.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| evaluator_slug | text NOT NULL | `leaf_sufficiency`, `soil_sufficiency`, `stage_program`, `symptom_deficiency`, `antagonism`, `bitter_pit`. |
| run_at | timestamptz NOT NULL DEFAULT now() | |
| status | text NOT NULL | enum: `sufficient`, `deficient`, `excessive`, `antagonized`, `stage_action_due`, `no_data`. |
| nutrients | jsonb NOT NULL | Per-nutrient detail: {nutrient, value, unit, sufficiency_band, delta_from_band}. |
| result | jsonb NOT NULL | Full evaluator output including recommendations. |
| input_test_id | bigint NULL FK → leaf_tests(id) or soil_tests(id) | Polymorphic; use a `input_test_type` discriminator. |
| input_test_type | text NULL | enum: `leaf`, `soil`, `none`. |
| error | text NULL | |

#### 3.7.3 `weed_evaluations`

On-demand log. User invokes the weed advisor; one row per invocation.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| evaluated_at | timestamptz NOT NULL DEFAULT now() | |
| target_slug | text NOT NULL | `pre_emergent_burndown`, `post_emergent_broadleaf`, `sucker_control`, etc. |
| weather_window_recommendation | jsonb NOT NULL | Structured output: window_start, window_end, confidence, limiting_factors. |
| recommended_product_ids | bigint[] NOT NULL DEFAULT '{}' | |
| result | jsonb NOT NULL | Full evaluator output including reasoning. |
| evaluated_by | bigint NOT NULL FK → users(id) | |

#### 3.7.4 `triggers`

Fired trigger events. Append-only.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | |
| trigger_type | text NOT NULL | `scab_infection_window_open`, `fire_blight_high_risk_24h`, `codling_moth_peak_approaching`, `leaf_ca_below_sufficiency`, `coincidence_scab_fire_blight`, etc. |
| severity | text NOT NULL | enum: `urgent`, `warning`, `preparation`, `informational`. |
| source_model_run_id | bigint NULL FK → model_runs(id) | |
| source_nutrition_run_id | bigint NULL FK → nutrition_runs(id) | |
| source_scouting_observation_id | bigint NULL FK → scouting_observations(id) | |
| state_snapshot | jsonb NOT NULL | Snapshot of the source state at firing time. |
| fired_at | timestamptz NOT NULL DEFAULT now() | |
| window_opens_at | timestamptz NULL | |
| window_closes_at | timestamptz NULL | When the action window closes. |
| acknowledged_at | timestamptz NULL | |
| acknowledged_by | bigint NULL FK → users(id) | |
| acknowledgment_note | text NULL | |
| superseded_at | timestamptz NULL | Set when a later trigger of the same type for the same block supersedes this one (deduplication). |

Indexes: `(farm_id, fired_at DESC)`, `(block_id, trigger_type, fired_at DESC)`, partial index on `acknowledged_at IS NULL` for active-trigger queries.

#### 3.7.5 `trigger_definitions`

Configurable declarative triggers. Most V2 triggers are code-defined (hard-coded in model evaluators), but this table lets the author add, adjust, or disable triggers per farm without a code change.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NULL | Null means global (applies to all farms); non-null is a per-farm override. |
| trigger_type | text NOT NULL | |
| enabled | boolean NOT NULL DEFAULT true | |
| severity | text NOT NULL | |
| condition | jsonb NOT NULL | Declarative spec — source model, state transition, thresholds. |
| cooldown_hours | integer NOT NULL DEFAULT 24 | Deduplication window. |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.7.6 `coincidence_rules`

Per-disease/pest combinations that produce coincidence triggers. Reference data with per-farm override capability.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NULL | Null means global. |
| slug | text NOT NULL | `scab_plus_fire_blight`, `codling_moth_plus_obliquebanded_leafroller`, etc. |
| component_trigger_types | text[] NOT NULL | All must be active for the coincidence to fire. |
| within_hours | integer NOT NULL DEFAULT 24 | Alignment window. |
| severity | text NOT NULL | Usually higher than any component. |
| recommendation_template | text NULL | |

### 3.8 Alerts and Rule Evaluations

#### 3.8.1 `alert_preferences`

Formalized from V1's runtime-created `alert_prefs` table. Per-user-per-farm.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| user_id | bigint NOT NULL FK → users(id) ON DELETE CASCADE | |
| channels | text[] NOT NULL DEFAULT '{email}' | enum values: `email`, `sms`, `push`. |
| urgent_enabled | boolean NOT NULL DEFAULT true | |
| warning_enabled | boolean NOT NULL DEFAULT true | |
| preparation_enabled | boolean NOT NULL DEFAULT true | |
| informational_enabled | boolean NOT NULL DEFAULT false | |
| digest_enabled | boolean NOT NULL DEFAULT false | When true, non-urgent alerts bundle into a daily digest. |
| digest_time | time NOT NULL DEFAULT '07:00' | Local to orchard timezone. |
| quiet_start | time NULL | |
| quiet_end | time NULL | |
| per_model_overrides | jsonb NOT NULL DEFAULT '{}' | Map of model_slug → enabled. Overrides severity-level toggles. |
| herbicide_window_alerts_enabled | boolean NOT NULL DEFAULT false | Explicit opt-in; default off. |
| UNIQUE (farm_id, user_id) | | |

#### 3.8.2 `alerts`

One row per alert delivery attempt.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| trigger_id | bigint NOT NULL FK → triggers(id) ON DELETE CASCADE | |
| recipient_user_id | bigint NOT NULL FK → users(id) ON DELETE CASCADE | |
| channel | text NOT NULL | enum: `email`, `sms`, `push`. |
| status | text NOT NULL | enum: `pending`, `sent`, `failed`, `suppressed_quiet_hours`, `suppressed_digest`. |
| sent_at | timestamptz NULL | |
| delivery_id | text NULL | Resend/SMS provider's ID for retry correlation. |
| error | text NULL | |
| opened_at | timestamptz NULL | When the alert email/SMS link is clicked. |
| acknowledged_at | timestamptz NULL | Mirror of triggers.acknowledged_at; denormalized for per-alert tracking. |

Indexes: `(farm_id, sent_at DESC)`, `(trigger_id)`, `(recipient_user_id, status)`.

#### 3.8.3 `rule_evaluations`

When the advisor generates a recommendation, every rule applied to every candidate product writes a row here. Supports "show your work" on recommendation surfaces.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| advisor_session_id | bigint NOT NULL FK → advisor_sessions(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL | |
| product_id | bigint NOT NULL FK → products(id) | |
| rule_slug | text NOT NULL | `ontario_registration`, `growth_stage`, `seasonal_max`, `frac_rotation`, `phi_vs_harvest`, `rei_vs_work`, `tank_mix`, `inventory`, `young_tree_safety`, `nutrition_*`. |
| outcome | text NOT NULL | enum: `pass`, `warn`, `block`, `not_applicable`. |
| details | jsonb NOT NULL | Rule-specific output (e.g., `{applied_count: 2, label_max: 4}` for seasonal_max). |
| evaluated_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.8.4 `advisor_sessions`

Groups a set of rule evaluations produced during one advisor invocation. Retained for audit and "what did the app recommend on June 12."

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL | |
| trigger_id | bigint NULL FK → triggers(id) | |
| invoked_by | bigint NULL FK → users(id) | Null when the session is system-initiated (e.g., cron-generated digest). |
| invoked_at | timestamptz NOT NULL DEFAULT now() | |
| context | jsonb NOT NULL | Snapshot of block_states, recent applications, active triggers at invocation time. |
| ranked_products | bigint[] NOT NULL DEFAULT '{}' | Final ordered list of recommended products. |

#### 3.8.5 `rule_overrides`

When a grower logs an application that a rule would have blocked, the override is recorded. `applications.rule_override_id` references this.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL | |
| product_id | bigint NOT NULL | |
| rule_slug | text NOT NULL | |
| rule_outcome | text NOT NULL | The outcome that was overridden — typically `block` or `warn`. |
| rule_details | jsonb NOT NULL | Copy of the rule's details at override time. |
| override_reason | text NOT NULL | Grower-provided rationale. |
| overridden_by | bigint NOT NULL FK → users(id) | |
| overridden_at | timestamptz NOT NULL DEFAULT now() | |

### 3.9 Reference Registries

Global reference data, no farm_id, writes gated to `author` role.

#### 3.9.1 `varieties`

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| slug | text NOT NULL UNIQUE | `honeycrisp`, `mcintosh`, `gala`, `ambrosia`. |
| display_name | text NOT NULL | |
| parentage | text NULL | |
| harvest_window_default_start | text NULL | Relative to petal fall in weeks or approximate date. |
| harvest_window_default_end | text NULL | |
| notes | text NULL | |

#### 3.9.2 `variety_susceptibility`

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| variety_id | bigint NOT NULL FK → varieties(id) ON DELETE CASCADE | |
| target_slug | text NOT NULL | Disease or pest slug. |
| susceptibility | text NOT NULL | enum: `very_low`, `low`, `moderate`, `high`, `very_high`. |
| source | text NOT NULL | Citation. |
| notes | text NULL | |
| UNIQUE (variety_id, target_slug) | | |

#### 3.9.3 `rootstocks`

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| slug | text NOT NULL UNIQUE | `m9`, `m26`, `g41`, `g11`, `b9`, etc. |
| display_name | text NOT NULL | |
| vigor | text NULL | enum: `dwarf`, `semi_dwarf`, `semi_standard`, `standard`. |
| notes | text NULL | |

#### 3.9.4 `rootstock_susceptibility`

Same shape as variety_susceptibility. Fire blight susceptibility is the load-bearing entry.

#### 3.9.5 `weed_species`

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| slug | text NOT NULL UNIQUE | |
| common_name | text NOT NULL | |
| scientific_name | text NULL | |
| life_cycle | text NOT NULL | enum: `annual`, `biennial`, `perennial`. |
| growth_habit | text NOT NULL | enum: `broadleaf`, `grass`, `sedge`. |
| resistance_profile | jsonb NOT NULL DEFAULT '{}' | Known herbicide-resistance patterns. |
| notes | text NULL | |

#### 3.9.6 `per_farm_parameter_overrides`

Per-orchard model parameter overrides (Section 2.1.5).

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| orchard_id | bigint NOT NULL FK → orchards(id) ON DELETE CASCADE | |
| block_id | bigint NULL FK → orchard_blocks(id) ON DELETE CASCADE | Null means orchard-wide. |
| model_slug | text NOT NULL | |
| parameter_name | text NOT NULL | |
| parameter_value | jsonb NOT NULL | |
| reason | text NOT NULL | |
| author_user_id | bigint NOT NULL FK → users(id) | |
| effective_from | date NOT NULL | |
| effective_until | date NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

Model runs read overrides in effect at `run_at` and apply them. `model_runs.model_parameters_hash` changes when overrides apply, surfaced at the recommendation UI.

### 3.10 Author Role and Admin Tables

The `author` role is global, not farm-scoped. Used by the primary project author (and any delegates) to manage reference data.

#### 3.10.1 `global_roles`

| Column | Type | Notes |
|---|---|---|
| user_id | bigint PK FK → users(id) ON DELETE CASCADE | |
| role | text NOT NULL | enum: `author`, `support`. Separate from farm_memberships.role. |
| granted_at | timestamptz NOT NULL DEFAULT now() | |
| granted_by | bigint NULL FK → users(id) | |

#### 3.10.2 `catalogue_ingestion_runs`

Provenance tracking for product catalogue data. Every batch ingestion writes a row.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| source | text NOT NULL | enum: `manual_entry`, `pdf_extraction`, `csv_import`, `omafra_scrape`. |
| source_reference | text NULL | URL, filename, or OMAFRA resource ID. |
| ingested_at | timestamptz NOT NULL DEFAULT now() | |
| ingested_by | bigint NOT NULL FK → users(id) | |
| products_added | integer NOT NULL DEFAULT 0 | |
| products_updated | integer NOT NULL DEFAULT 0 | |
| notes | text NULL | |

Individual product rows reference `label_ingested_at` and indirectly the ingestion run via audit tables when needed. Section 5 covers the ingestion workflow in detail.

### 3.11 Scouting and Test Records

#### 3.11.1 `scouting_observations`

Preserved from V1 with block scoping.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | |
| observation_date | date NOT NULL | |
| observation_time | time NULL | |
| target_slug | text NOT NULL | Pest, disease, or deficiency. |
| severity | text NULL | enum: `trace`, `light`, `moderate`, `severe`, null for presence-only. |
| incidence_pct | numeric(5,2) NULL | Percentage of trees, fruit, or leaves affected. |
| count | integer NULL | Used for countable observations (number of insects per trap, number of galls per tree). |
| notes | text NULL | |
| observed_by | bigint NOT NULL FK → users(id) | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.11.2 `scouting_photos`

Preserved from V1. Block-scoped (upgrade from V1 orchard-scoped).

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | |
| scouting_observation_id | bigint NULL FK → scouting_observations(id) | Optional link. |
| target_slug | text NULL | |
| photo_date | date NOT NULL | |
| file_path | text NOT NULL | |
| mime_type | text NOT NULL | |
| notes | text NULL | |
| uploaded_by | bigint NOT NULL FK → users(id) | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.11.3 `phenology_photos`

NEW. Photos tagged with a stage for stage-progression validation.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| photo_date | date NOT NULL | |
| stage_claimed | text NOT NULL | The stage the system suggested. |
| stage_confirmed | text NOT NULL | The stage the grower confirmed (may equal or correct the claimed stage). |
| file_path | text NOT NULL | |
| mime_type | text NOT NULL | |
| uploaded_by | bigint NOT NULL FK → users(id) | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.11.4 `pheromone_traps`

NEW. Per-block trap deployments.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) ON DELETE CASCADE | |
| pest_slug | text NOT NULL | |
| trap_type | text NOT NULL | enum: `delta`, `bucket`, `wing`, `pherocon`, `other`. |
| deployed_at | date NOT NULL | |
| removed_at | date NULL | |
| location_notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.11.5 `pheromone_trap_counts`

Observation rows against trap deployments.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL | |
| trap_id | bigint NOT NULL FK → pheromone_traps(id) ON DELETE CASCADE | |
| count_date | date NOT NULL | |
| count | integer NOT NULL | |
| observed_by | bigint NOT NULL FK → users(id) | |
| notes | text NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.11.6 `weed_scouting`

NEW. Per-block weed pressure observations.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | |
| observation_date | date NOT NULL | |
| species_observed | jsonb NOT NULL | Array of {weed_species_id, density, growth_stage, notes}. |
| general_pressure | text NOT NULL | enum: `low`, `moderate`, `high`. |
| notes | text NULL | |
| observed_by | bigint NOT NULL FK → users(id) | |
| created_at | timestamptz NOT NULL DEFAULT now() | |

#### 3.11.7 `soil_tests` and `leaf_tests`

Preserved from V1 with block scoping.

**`soil_tests`:** id, farm_id, block_id, sample_date, pH, organic_matter_pct, N/P/K/Ca/Mg/B/Zn/Mn (ppm), CEC, base_saturation, lab_name, notes, created_at. V1 had orchard-scoped soil tests; V2 ties each test to a block.

**`leaf_tests`:** id, farm_id, block_id, sample_date, sample_type (mid-season/post-harvest), N/P/K/Ca/Mg (pct), B/Zn/Mn/Fe/Cu (ppm), lab_name, notes, created_at.

Manual entry only. Lab PDF ingestion is out of scope for V2.

### 3.12 Irrigation (Preserved from V1)

Preserved from V1 with block scoping promoted to primary key where necessary. Tables: `irrigation_config` (per-block), `water_balance` (per-block-per-date), `irrigation_log`, `irrigation_rainfall_manual`.

No schema changes beyond:
- Add `farm_id` NOT NULL to every table.
- Replace `orchard_id` with `block_id` on `irrigation_config`, `water_balance`, `irrigation_log`, `irrigation_rainfall_manual`. V1's per-orchard irrigation config becomes per-block config; the typical case of one config for the whole orchard is handled by copying the config row across blocks at migration time.
- `water_balance` adds `soil_temp_c` when available (for cross-link with weed pre-emergent timing).

### 3.13 Scab Infection Log

Preserved from V1.

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) ON DELETE CASCADE | |
| block_id | bigint NOT NULL FK → orchard_blocks(id) | |
| infection_date | date NOT NULL | |
| severity | text NOT NULL | enum: `light`, `moderate`, `severe`. |
| mean_temp | numeric(5,2) NULL | |
| wetness_hours | numeric(4,2) NULL | |
| protected | boolean NOT NULL DEFAULT false | |
| fungicide_used_product_id | bigint NULL FK → products(id) | |
| notes | text NULL | |
| model_run_id | bigint NULL FK → model_runs(id) | Link to the scab model run that identified this event. |
| created_at | timestamptz NOT NULL DEFAULT now() | |

### 3.14 Retention and Partitioning

- `model_runs`, `nutrition_runs`, `weed_evaluations`, `triggers`, `alerts`, `rule_evaluations`, `weather_hourly` are partitioned by month. Retention default: 5 seasons. Older partitions archived or dropped per farm preference.
- `applications`, `scouting_observations`, `scouting_photos`, `phenology_photos`, `soil_tests`, `leaf_tests`, `jar_test_log`, `pheromone_trap_counts`, `weed_scouting`, `rule_overrides` are retained indefinitely. These are the record-of-truth for the farm.
- `weather_hourly` partitioning allows V1's "typically 7 days retained" pattern to shift to "7 days hot, rest archived" at scale.

### 3.15 Row-Level Security

Every farm-scoped table has an RLS policy enforcing that `current_setting('app.current_farm_id')::bigint = farm_id`. The application layer sets this per-request based on the authenticated user's active farm membership. Reference tables (products, varieties, rootstocks, weed_species, coincidence_rules with null farm_id) are readable by all authenticated users and writable only by `author` role.

RLS is the second line of defense. The first is application-layer query builders that always inject `farm_id` filters. Belt and suspenders.

### 3.16 V1 → V2 Migration Notes

V2 is a new database. V1 data migrates via an ETL script run once per farm onboarding. Key migration moves:

- V1 single orchard → V2 farm with one orchard. Seed the owner's user record, a single farm, a single orchard, a single farm_membership (owner role).
- V1 `spray_log` + `fertilizer_log` → V2 `applications`. Map V1's free-text `product` field to `products.id` by name matching (Section 5 surfaces unmapped entries for manual resolution). V1 records without a `product_id` FK are lowest-confidence matches.
- V1 `inventory` → V2 `inventory_lots`. One lot per V1 inventory row.
- V1 `orchards.bloom_stage`, `codling_moth_biofix_date`, `petal_fall_date`, `fire_blight_history` → per-block equivalents. At migration time these are copied to every block in the orchard; the grower then per-block-corrects any differences.
- V1 `planted_blocks` (v5 legacy): dropped. V1 `orchard_blocks` + `block_plantings` (v6): preserved, re-keyed with farm_id.
- V1 `workers`: dropped. If any were used for alert SMS, their phone numbers migrate to the corresponding `users.phone` field where they exist; orphaned worker rows are discarded with a migration report.
- V1 `alert_prefs` (runtime-created): migrated to `alert_preferences`.
- V1 `soil_tests`, `leaf_tests`, `fertilizer_log`: migrated with block_id populated from orchard→block lookup (grower confirms block assignment at migration time).
- V1 `scab_infection_log`: migrated with block_id populated.

Migration ETL is a Section 5 concern; data-model-side requirement is that every V2 table has sufficient structure to receive the V1 data without loss.

### 3.17 Standard Index Policy

Beyond the explicit indexes noted per table:

- Every FK column has an index.
- Every `(farm_id, created_at DESC)` or `(farm_id, *_at DESC)` pattern commonly queried has an index.
- Partitioned tables have local indexes on the partition key plus filter columns.
- Full-text indexes are not introduced in V2; search UI is filter-based.

### 3.18 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| Postgres with RLS | Multi-tenant from day one; RLS as belt-and-suspenders on `farm_id` enforcement. |
| Single schema with farm_id | Operational simplicity; schema-per-tenant's benefits don't apply. |
| Unified `applications` table with category enum | History queries (FRAC, seasonal max, PHI) need to union across domains; one table removes four-way unions. |
| Base `products` + four extension tables | Domain data differs enough that a wide nullable table becomes ugly. Extensions keep each domain clean. |
| Three distinct evaluator-output tables (`model_runs`, `nutrition_runs`, `weed_evaluations`) | Output shapes differ. Forcing them into one table means JSONB for everything, which forfeits the schema value. |
| Block-scoped applications with per-planting PHI resolution | Matches how growers actually spray (one pass over a block); per-planting PHI is computed from block contents. Revisit if 2026 data shows split applications are common. |
| Append-only for model_runs, triggers, alerts, rule_evaluations, rule_overrides | Audit matters for these; update-in-place would lose the history that makes retrospective review work. |
| `inventory_lots` replaces V1 `inventory` | Lot-level tracking is needed for real FIFO and lot-specific expiry. V1's single-row-per-product was always going to need this. |
| Farm-scoped everywhere except reference registries | Clear boundary. Reference data is managed by `author` role globally; farm data is RLS-protected. |
| Monthly partitioning on append-only + weather tables | Retention management becomes tractable as V2 scales across farms and seasons. |
| `block_plantings.expected_harvest_date` is the PHI anchor | Required for PHI rule; null falls back to variety default; most restrictive across all plantings in a block governs block-wide applications. |
| V1 orchard-wide state fields move to per-block state tables | Closed-loop advisor requires block-level state; storing it at orchard level blocks block-aware modeling. |
| `rule_evaluations` records every rule pass per product per advisor session | "Show your work" on recommendation surfaces and post-season audit depend on this. Storage cost is acceptable given monthly partitioning. |
| Reference data writes gated to `author` role, not farm owners | Catalogue correctness is a shared correctness concern. Farm-owner edits would fork per-farm catalogues and undermine the shared-responsibility posture. Per-farm customization is via `per_farm_parameter_overrides`, not catalogue edits. |

### 3.19 Open Questions Deferred from This Section

1. **Input-test polymorphism on `nutrition_runs`.** `input_test_id` + `input_test_type` is a classic polymorphic-association pattern that SQL tools don't love. Alternative: two nullable columns, `input_leaf_test_id` and `input_soil_test_id`. Pick before Section 4.
2. **`rule_evaluations` volume.** One advisor session produces dozens of rows (every rule × every candidate product). For 5 farms × daily advisor runs × 10 blocks × 30 candidate products × 10 rules ≈ 15K rows/day. Postgres will handle this, but if advisor runs become more frequent (per-trigger), revisit partitioning and retention. Flag for Section 8 load testing.
3. **Photo storage.** Columns reference `file_path`. Actual storage location (local disk, S3/R2, Railway volume) is a Section 9 decision.
4. **JSONB vs. typed columns for `result` fields.** Most evaluator outputs are JSONB. The pragmatic cost: queryability suffers. If specific fields become hot query targets, promote them to typed columns with generated-column extraction. Defer until usage patterns emerge.
5. **Time zone handling on `phenology_photos.photo_date` and other `date` columns.** Storing date rather than timestamptz is correct for "logical date in orchard time" but introduces ambiguity at day boundaries. Policy: `date` columns are orchard-local dates; the application layer converts from client-local time using `farms.timezone`. Flagging in case Section 8 testing surfaces edge cases.
6. **`weather_daily`: materialized view vs. table.** V1 uses a view. At V2 scale with partitioning, a refreshed table may be faster. Benchmark before Section 8.
7. **Tank mix groups when one component is a soil amendment.** Tank mixing is almost always sprays-with-sprays; logging a soil amendment into a tank mix group doesn't make physical sense. Do we enforce that tank_mix_groups only contain pesticide/herbicide/foliar_nutrient applications? Constraint added at the application layer, not the schema. Flag for Section 4 rule work.

---

## 4. Rule Engine Architecture

The rule engine is the piece of V2 that turns raw data — triggers, block state, catalogue entries, application history, inventory — into filtered, ranked, explainable recommendations. It is the largest net-new subsystem in V2; V1 has no rule engine and the lack of one is the single biggest reason V1's parts don't talk.

This section specifies how the engine is structured, how rules compose, how context flows, how outputs are consumed, and how rules are authored. It stops short of naming every rule exhaustively — individual rule specs are an implementation concern — but names every rule *category* and provides representative sub-rule examples.

### 4.1 Design Goals

The engine is built for five properties, in decreasing priority:

1. **Explainability.** Every rule outcome carries enough structured data to reconstruct *why* a product was filtered out or why a warning fired. No opaque "this spray isn't allowed" messages. V1 has no reasoning trail; V2 has one by default.
2. **Composability.** Adding a rule is a matter of implementing an interface and registering it, not editing advisor internals. The set of active rules is data, not code paths.
3. **Determinism.** Given the same inputs — catalogue state, block state, application history, active triggers — the engine produces the same output every time. No stochasticity, no wall-clock dependencies beyond explicit "as of this timestamp" inputs.
4. **Failure isolation.** One rule that throws or produces unexpected data does not halt the evaluation. The advisor records the failure, skips that rule, and produces a best-effort recommendation with the failure surfaced. Same pattern as Section 2.1.5 per-model isolation.
5. **Testability.** Every rule has unit tests against representative inputs. Rule changes are verifiable before deployment. This matters more than usual because rule correctness is what makes the shared-responsibility liability posture honest.

### 4.2 Engine Structure

The engine has four layers:

1. **Context assembly.** Gathers every input a rule might need and packages it into a single immutable context object. The advisor calls this once per session.
2. **Rule application.** Each rule is a function `(context, candidate_product) → RuleEvaluation`. Rules run against every candidate product independently. Rules do not see each other; they see the context.
3. **Outcome aggregation.** Per candidate product, the engine collects all rule evaluations and produces a `CandidateVerdict` summarizing whether the product is recommendable, with warnings, or blocked.
4. **Ranking.** Recommendable candidates are ordered by a ranking function that reads all rule evaluations plus context.

This separation exists so that rule authoring stays local — a new rule is a new function, not a change to shared logic. The engine's middle layers (application, aggregation, ranking) are rule-agnostic.

### 4.3 Context Object

The context object is assembled once per advisor invocation. It is read-only during rule evaluation. Approximate shape:

```
RuleContext {
  farm_id
  block_id
  evaluated_at        // timestamp, injected not wall-clock
  trigger             // the trigger being acted on, if any
  orchard             // orchard row
  block               // orchard_blocks row
  plantings           // block_plantings rows for this block
  block_state         // current bloom stage, microclimate override
  applications_history  // applications table slice relevant to rule evaluation
  inventory           // inventory_lots slice for this farm
  recent_model_runs   // last run per model for this block
  recent_nutrition_runs // last run per nutrition evaluator for this block
  active_triggers     // unacknowledged triggers on this block
  variety_susceptibility // precomputed per-planting susceptibility scores
  rootstock_susceptibility
  weather_recent      // weather_hourly slice for the past ~168h
  weather_forecast    // forecast slice for next ~168h
  parameter_overrides // per_farm_parameter_overrides in effect
}
```

The context is bounded — assembly queries are capped in time range (typically 90 days back for applications, 7 days for weather, 30 days for model runs). Rules that need older data declare it explicitly and assembly widens the queries accordingly; the default is small for performance.

### 4.4 Rule Interface

Every rule implements:

```
Rule {
  slug: text              // stable identifier, matches rule_evaluations.rule_slug
  category: text          // from 4.6
  applies_to: [category]  // product categories this rule evaluates
  required_context: [text] // which context fields must be populated
  evaluate(context, product) → RuleEvaluation
}

RuleEvaluation {
  rule_slug
  outcome: "pass" | "warn" | "block" | "not_applicable" | "error"
  severity: "informational" | "advisory" | "hard"  // see 4.5
  reason: text            // human-readable, grower-facing
  details: jsonb          // structured, for UI display and audit
  citations: [{label, url}]  // sources for this rule's rationale
}
```

Rules are pure functions of context and candidate. They do not perform I/O; all data is in context. Rules do not modify state; the advisor writes `rule_evaluations` rows based on returned evaluations.

### 4.5 Outcome Severity and the Hard/Soft Split

The engine recognizes three outcome severities, which together with the four outcome values produce the final verdict:

- **Hard.** A `block` outcome at hard severity removes the candidate from recommendations. The grower cannot get to a recommendation of this product without an explicit override (Section 4.8). Hard rules are regulatory or physical: registration, PHI, REI.
- **Advisory.** A `warn` or `block` outcome at advisory severity does not remove the candidate, but marks it with a warning that surfaces in the UI. The grower chooses. Advisory rules are best-practice guidance: FRAC rotation, resistance-risk management, tank-mix compatibility where published label data doesn't forbid it.
- **Informational.** A `warn` outcome at informational severity produces a badge or note on the candidate. No impact on ranking. Rules: efficacy rating, inventory age nearing expiry, cost flag.

The hard/advisory distinction matters. V2's posture is "the grower decides" — advisory rules honor that. But some things are not advisory: spraying a product not registered for use on apple in Ontario is not a judgment call. The engine enforces the hard/soft split explicitly; rule authors specify severity at registration time.

When evaluating a candidate, the engine produces:

```
CandidateVerdict {
  product_id
  overall: "recommendable" | "recommendable_with_warnings" | "blocked_hard" | "blocked_by_override_only"
  hard_blocks: [RuleEvaluation]
  advisory_blocks: [RuleEvaluation]
  warnings: [RuleEvaluation]
  informational: [RuleEvaluation]
  passing: [RuleEvaluation]
}
```

`blocked_by_override_only` indicates no hard blocks, but at least one advisory block that the grower could override. `blocked_hard` indicates at least one hard block; the grower cannot proceed without an *author-authorized* override (Section 4.8), which is rare and deliberately frictional.

### 4.6 Rule Categories

Twelve categories. Each named here with representative sub-rules. Full sub-rule enumeration is an implementation concern; what matters is the shape.

#### 4.6.1 Registration

Severity: **hard**. Applies to all product categories.

- **ontario_registration.** Product's `ontario_registered = true` and `pcp_number` is on file. Block if false.
- **target_on_label.** The trigger's target slug is in the product's `target_pests` or `target_weeds` list. Warn (advisory) if the target is closely related but not explicitly listed; block (hard) if entirely off-label.

#### 4.6.2 Growth Stage Compatibility

Severity: **hard** when the label explicitly forbids; **advisory** when the label is ambiguous.

- **stage_window.** Current `block_state.bloom_stage` falls within `[growth_stage_min, growth_stage_max]` from the product extension. Block (hard) if outside.
- **bloom_stage_active_ingredient.** Specific active ingredients have stage-specific prohibitions beyond the product-level range (e.g., some sterol inhibitors not during bloom). Warn or block per label.

#### 4.6.3 Seasonal Application Limits

Severity: **hard** for label maxima; **advisory** for best-practice limits below the label max.

- **seasonal_max_applications.** Count applications of this product on this block in current season. Block if applying would exceed `seasonal_max_applications`.
- **seasonal_max_rate_cumulative.** Sum of actual rate applied in current season. Block if applying would exceed `seasonal_max_rate`.
- **seasonal_max_by_active_ingredient.** Some label restrictions are per-active-ingredient, not per-product. Count applications of any product sharing this active ingredient. Block if exceeded.
- **min_days_between_applications.** Days since last application of this product on this block. Block if less than `min_days_between_applications`.

#### 4.6.4 Resistance Management (FRAC / IRAC / HRAC)

Severity: **advisory**. Rotation is strongly recommended, not regulated.

- **back_to_back_same_group.** Last applicable application's FRAC/IRAC/HRAC group against the same or overlapping target. Warn if applying same-group again; block (advisory) if the grower's pattern is three or more consecutive same-group applications.
- **seasonal_group_diversity.** For each major target category, count distinct groups applied this season. Warn if diversity is below resistance-management guidance for the target (e.g., scab program should use at least 3 distinct FRAC groups).
- **high_resistance_risk_flag.** Products with `resistance_risk = high` (single-site fungicides like strobilurins, SDHIs) trigger a stronger warning when used back-to-back.

This is a distinct rule category, not part of registration, because the rules are best-practice — the label permits back-to-back use; good stewardship discourages it.

#### 4.6.5 Pre-Harvest Interval

Severity: **hard**. Applies to pesticides, herbicides, and foliar nutrients with PHIs. Does not apply to soil amendments.

- **phi_vs_block_harvest.** For every planting in the block with a populated `expected_harvest_date`, check (application_date + `phi_days`) ≤ expected_harvest_date. Block (hard) if violated for any planting. The reason includes which variety's harvest date governs.
- **phi_vs_plausible_harvest.** When `expected_harvest_date` is null, fall back to the variety's `harvest_window_default_start`. Warn (advisory) rather than block — the grower hasn't committed to a date.

#### 4.6.6 Re-Entry Interval

Severity: **advisory**. The grower can re-enter with PPE regardless; the rule flags prohibited work.

- **rei_active_hours.** Hours since the most recent applicable application. Output feeds the REI Go/No-Go panel (Section 2.3): current REI countdown, work types prohibited, time until each unlocks. This rule does not block recommendations — it's about reading the current REI state, not about a pending application.
- **rei_work_type_specific.** When `rei_work_types` in the product extension specifies per-work-type intervals, produce per-work-type status.

This rule is slightly different from the others: it rarely runs during candidate evaluation (REI is about post-application state, not pre-application decision). It runs on-demand to generate the Go/No-Go panel and at application-logging time to confirm the application is not happening into an active REI from a different product. In the latter case it can block hard.

#### 4.6.7 Tank Mix Compatibility

Severity: **advisory**.

- **known_incompatible.** Proposed candidate is in `tank_mix_known_incompatible` of any product already in the tank mix group. Block (advisory) with jar-test prompt.
- **known_compatible.** Proposed candidate is in `tank_mix_known_compatible`. Pass.
- **unknown_compatibility.** Proposed candidate has no record with the other mix members. Warn and surface jar-test prompt with protocol. Advisor logs result to `jar_test_log` and future evaluations read it.
- **ph_incompatibility.** Product's `ph_sensitivity` conflicts with another mix member's pH or with the mix's water source pH. Warn.
- **adjuvant_compatibility.** Required adjuvants are declared compatible. Warn if not.

#### 4.6.8 Inventory Availability

Severity: **advisory** (a recommendation to buy, not a regulatory block).

- **in_stock.** Sum of `inventory_lots.quantity_on_hand` for this product ≥ expected need (rate × area). Warn if short; block (advisory) if zero stock.
- **not_expired.** At least one lot has `expiry_date > evaluated_at` or null. Warn if all lots are expired; block (advisory).
- **lot_nearing_expiry.** At least one lot expires within 30 days. Informational badge.

#### 4.6.9 Young-Tree Safety

Severity: **hard** when the label explicitly prohibits; **advisory** when the label warns.

- **tree_age_minimum.** For every planting in the block, check `evaluated_at.year - planted_year ≥ product.min_tree_age_years`. Block (hard) if any planting is too young and the label prohibits; warn (advisory) if the label only cautions.
- **bark_contact_sensitivity.** For herbicides and some pesticides, young-bark contact is a concern. Warn based on `products_herbicide.young_tree_warning`.

#### 4.6.10 Nutrition Rules

Severity: **split between hard and advisory** — regulatory or physically irreversible constraints are hard; guidance and optimization are advisory.

**Hard:**

- **late_season_nitrogen_cutoff.** Applying nitrogen after a regional cutoff date (Ontario bearing apples: typically mid-August). Late-season N suppresses cold hardiness; the consequence (winter injury) is not recoverable within the season. Block (hard). Cutoff date is a `per_farm_parameter_overrides`-eligible parameter for known climate variation.
- **seasonal_nutrient_cap_labeled.** When the foliar nutrient's label specifies a hard seasonal maximum per hectare. Block (hard) if applying would exceed.
- **amendment_pH_overshoot.** For liming amendments: estimated resulting pH after application exceeds target pH band by more than the tolerance threshold. Overshooting is costly to correct (requires sulfur or gypsum and years). Block (hard).

**Advisory:**

- **sufficiency_band_compliance.** For foliar nutrients and amendments, the intended nutrient's current state (from most recent `nutrition_runs`) is `deficient` or `no_data`. Warn if `sufficient` (no need); warn if `excessive` (risk of worsening toxicity or antagonism).
- **nutrient_antagonism.** Applying nutrient X while state shows excess of antagonist Y (K-Mg, Ca-B, Ca-K, P-Zn). Warn.
- **stage_gated_nutrient.** Nutrient timing relative to bloom stage (boron at pink, calcium petal fall through cell division, fall urea for scab sanitation). Warn if applied outside the window; outside-window can still be valid, hence advisory.
- **seasonal_nutrient_cap_guidance.** Cumulative nutrient delivered per season exceeds guidance target (below labeled max, above best-practice target). Warn.
- **amendment_pH_target.** For liming amendments, estimated resulting pH within target band but at the edge. Warn if near the edge without overshooting.
- **amendment_timing.** Current date falls within `application_timing` windows (dormant, pre-bloom, post-harvest, fall, spring). Warn if outside.

#### 4.6.11 Weed Advisor Rules

Severity: **hard** for label-regulated; **advisory** for timing guidance.

- **pre_emergent_soil_temp.** Current soil temperature ≥ `soil_temp_min_c` from herbicide extension. Warn if below (pre-emergent won't activate).
- **pre_emergent_rainfall_window.** Forecast rainfall ≥ `rainfall_trigger_mm` within 7–14 days. Warn if insufficient.
- **post_emergent_weed_stage.** Most recent `weed_scouting` shows weeds at effective growth stage for the candidate herbicide. Warn if weeds are past effective stage.
- **rainfast_window.** Forecast shows no rain for ≥ `min_hours_before_rain` hours post-application. Warn if rain is imminent.
- **wind_speed_application_limit.** Forecast wind speed at intended application window ≤ label-specified drift limit. Warn or block per label.
- **application_method_permitted.** Directed spray vs broadcast per label. Block (hard) if the grower's intended method is disallowed.

#### 4.6.12 Coincidence Rules

Severity: **informational** at the rule-evaluation level; the coincidence trigger itself is what elevates urgency.

- **coincidence_match.** Active triggers on this block match a pattern in `coincidence_rules`. Produces a recommendation suggestion at the advisor level ("both scab and fire blight windows are open; prioritize products with activity against both"). The rule doesn't filter products; it annotates them with "addresses N of the current coincident targets."

#### 4.6.13 (Reserved) Future Categories

Space for rules that emerge from 2026 V1 usage: rotation-group-per-target-pattern (some scab programs structure rotation around target life stages, not calendar time), weather-dependent-efficacy (some fungicides lose effectiveness above certain temperatures), spray-coverage-economy (when the advisor is proposing a minor tweak and a later trigger will require another spray anyway, it can suggest deferring). These are speculative; Section 8 revisits after 2026 data.

### 4.7 Ranking

Recommendable candidates are ranked by a function that reads rule evaluations and context. The ranking function is deterministic and explainable.

Ranking factors, in order of influence:

1. **Hard compliance.** Candidates with any `blocked_hard` are excluded from ranking entirely. This is not a factor in the ordering; it's a filter preceding ranking.
2. **Multi-target coverage.** In multi-trigger sessions, candidates that address multiple active targets outrank candidates that address only one. Coverage is weighted by trigger urgency.
3. **Warning count and severity.** Fewer advisory warnings rank higher. An advisory block outranks a warn; a warn outranks an informational.
4. **Efficacy rating against the trigger's targets.** From `product_efficacy.efficacy_rating`. Higher efficacy ranks higher. In multi-trigger sessions, efficacy is averaged (or summed, weighted by target urgency) across covered targets.
5. **FRAC/IRAC/HRAC rotation fitness.** A product that advances the season's group diversity score outranks one that doesn't. A product that avoids a back-to-back group match outranks one that would cause a back-to-back.
6. **Inventory availability.** In-stock products outrank not-in-stock. Among in-stock, older lots (FIFO) outrank newer.
7. **PHI safety margin.** Larger gap between (application_date + PHI) and the earliest harvest_date outranks tighter gaps.
8. **Cost.** Lower cost-per-application outranks higher, but this is the lowest-weight factor — never enough to flip the order based on higher-weight factors alone.

Ranking produces an ordered list with a per-candidate score breakdown. The UI shows the breakdown on hover or expansion. No black-box scoring.

**Ties.** Broken by product trade name alphabetically for determinism.

**Substitution suggestions (post-V2 enhancement).** A planned enhancement beyond V2 will add a "substitution suggestions" panel alongside the ranked list: "top recommendation is Merivon ($180/ha, 5★ for scab); alternative Captan ($22/ha, 4★ for scab) offers meaningful cost savings." This is deliberately a separate panel, not a collapsed single score, so the grower sees both numbers and decides. Not in V2; tracked in Section 10.

### 4.8 Overrides

Growers can override advisory blocks. They cannot override hard blocks without author-authorized elevation (rare, used for e.g. emergency minor-use exemptions).

Override flow:

1. Grower attempts to log an application that an advisory block would prevent.
2. UI surfaces a dialog: "This application was blocked by [rule_slug]: [reason]. Record override reason and proceed?"
3. Grower enters a reason. System writes a `rule_overrides` row with all rule details preserved, then proceeds to write the `applications` row with `rule_override_id` set.
4. Post-season reporting includes override counts per rule slug and per user, surfacing patterns.

Hard overrides require a different flow: the grower contacts the author (out-of-band), and the author authorizes by adding a `per_farm_parameter_overrides` entry that relaxes the specific rule for a bounded time window. This is deliberately frictional.

### 4.9 Rule Authoring Workflow

Rules are code, living in a `rules/` directory organized by category. Each rule has:

- A source file implementing the `Rule` interface
- A unit test file with representative inputs and expected outputs
- A citation record (OMAFRA Crop Protection Hub reference, label PDF, or peer-reviewed source) stored alongside the rule and surfaced in `RuleEvaluation.citations`

Adding a rule:

1. Author writes the rule function and test cases.
2. Author adds the rule to the registry (a declarative list loaded at engine startup).
3. Pull request includes citation, test cases, and a plain-language description of what the rule does and who it protects.
4. Merge deploys the rule. The engine picks it up on next restart.

Disabling a rule (for investigation or because it's too aggressive):

1. Registry entry is flipped to `enabled: false`. Engine restart required.
2. Or, for per-farm disable, a `trigger_definitions`-style override is added — this is not yet implemented for rules, flagged as an open question.

### 4.10 Context Assembly Performance

Context assembly is the most expensive part of evaluation. Budget: ≤ 500 ms for a full context on a production-sized block. The advisor is not a real-time UI path — most invocations are cron-driven or human-initiated — so 500 ms is acceptable. When the budget is at risk:

- Context assembly runs queries in parallel where possible.
- Commonly-accessed slices (recent applications, active triggers, inventory summary per farm) are cached at advisor-session scope, not rule-evaluation scope.
- Rules declare `required_context` and the assembler skips unused slices. A simple registration check doesn't need 7 days of weather.

### 4.11 Integration with the Advisor

The advisor invokes the engine per trigger *or per trigger set*. When multiple triggers are active on the same block within a close time window (24h default, configurable), the advisor runs a **combined multi-trigger session** that considers all active triggers together rather than producing independent recommendations for each.

The flow for a single-trigger session:

1. Trigger fires (or user opens advisor on an active trigger).
2. Advisor calls context assembly with `(farm_id, block_id, trigger_id)`.
3. Advisor identifies the candidate set — products in the catalogue registered for the trigger's target, intersected with the grower's inventory (filter can be disabled to show "what I would need to buy").
4. Advisor runs the rule engine on each candidate.
5. Advisor persists `advisor_sessions` and `rule_evaluations` rows.
6. Advisor returns the ranked list to the UI with full reasoning.

The flow for a multi-trigger session:

1. Trigger A fires on a block. Advisor checks for other active (unacknowledged) triggers on the same block within the merge window.
2. If other triggers exist, advisor assembles context once and identifies the combined candidate set — products registered for *any* of the active targets, union.
3. Rule engine evaluates each candidate against each target it's registered for. A candidate can pass for target X and fail for target Y; the combined verdict records both.
4. Ranking considers **coverage** — candidates that address multiple active targets rank higher, weighted by target urgency (urgent triggers weight more than preparation triggers).
5. The advisor also surfaces **tank-mix bundles**: candidate pairs or triples that together cover all active targets, are mutually tank-mix-compatible, and pass rules individually. Jar-test prompts surface for pairs with unknown compatibility.
6. One `advisor_sessions` row is written referencing all source triggers; `rule_evaluations` rows carry the target against which each evaluation was made.

A single advisor session may produce 20–200 rule_evaluations rows in single-trigger mode or 50–500 in multi-trigger mode. Monthly partitioning of that table (Section 3.14) handles the volume.

When a grower acknowledges one trigger but not others, the outstanding triggers remain in the active set for subsequent sessions. Acknowledged triggers are excluded from merge consideration on the next invocation.

### 4.12 Testing Strategy

Three test levels:

- **Rule unit tests.** Each rule has scenario-based tests: positive cases (should pass), negative cases (should block/warn), edge cases (boundary conditions on thresholds, null inputs where relevant).
- **Engine integration tests.** Fixed catalogue + fixed block state + fixed history → expected verdict. Tests verify that rule composition produces the expected combined outcome.
- **Scenario tests.** End-to-end advisor invocations replaying historical (real or synthetic) seasons. 2026 V1 usage data, once collected, becomes a replay corpus: "given the 2026 state on May 15, would V2 have produced a better recommendation than the grower's actual spray?" These tests are the strongest evidence that V2 is net-positive vs. V1.

Every rule change requires updated unit tests. Engine-level changes require re-running integration and scenario tests.

### 4.13 Observability

The engine emits structured logs and metrics:

- Every advisor session's duration, candidate count, rule count, final ranking.
- Every rule's per-invocation outcome and duration.
- Every rule error with full context.
- Rule-override frequency per farm, per rule, per user.

The observability target is: given a grower's complaint "the app told me to spray X on June 12 and it was wrong," the author can reconstruct the full rule evaluation from persisted `advisor_sessions` + `rule_evaluations` rows. No reliance on logs for audit.

### 4.14 Rule Catalogue Versioning

Rules and their thresholds change over time — OMAFRA guidance updates, labels revise, best practices evolve. The engine stores:

- A git-committed rule registry with rule implementations versioned by git SHA.
- `model_runs.model_parameters_hash` and equivalent for rule evaluations capture the parameter state at evaluation time.
- When rules change in ways that would retroactively affect past evaluations, old evaluations are not re-run automatically. Past `rule_evaluations` rows reflect the rules in effect at that time.

"What did the app recommend on June 12" is answered from `advisor_sessions` and `rule_evaluations`, which are append-only and reflect the then-current rules.

### 4.15 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| Rules are pure functions over an immutable context object | Determinism, testability, failure isolation all follow from this. Rules that do I/O are rules that break. |
| Hard / advisory / informational severity is a first-class rule property | The "grower decides" posture requires a clear line between regulatory blocks (not overridable casually) and best-practice guidance (advisory). Collapsing severities into one scale would dilute the posture. |
| Nutrition rules are split between hard and advisory | Most nutrition guidance is advisory, but some constraints (late-season N cutoff, labeled seasonal caps, pH overshoot) protect against irreversible consequences and are hard. |
| Advisor writes `rule_evaluations` for every rule-candidate pair | Explainability and audit. Storage cost is managed by monthly partitioning. |
| Rules can be disabled globally via registry flag but not per-farm yet | Per-farm disable is useful but adds state and trust concerns. Defer until a real need emerges. |
| Ranking factors are deterministic, weighted, and transparent in the UI | No ML-learned ranker; no opaque score. The grower should be able to understand why product A ranks above product B. |
| Multi-target coverage is a top ranking factor | When multiple triggers are active, a product that addresses several targets is meaningfully better than one that addresses one. Ranking reflects that. |
| Advisory blocks are overridable with recorded reason; hard blocks are not casually overridable | Matches regulatory reality (off-label use is not a personal choice) and user posture (experienced growers deserve flexibility). |
| Hard blocks cannot be session-overridden; catalogue-data errors are fixed in the catalogue, not worked around | Honest with the shared-responsibility posture. A hard block surfaced incorrectly is a catalogue bug, not a rule-engine exception. |
| Context assembly is bounded and caches at session scope | 500 ms budget; caching reduces redundant queries within a session. Cross-session caching not needed. |
| Rules live in a versioned `rules/` directory with citations and tests | Authoring is git-tracked; citations make the "decision-support, not prescription" framing concrete at every rule. |
| No per-farm rule overrides in V2 beyond `per_farm_parameter_overrides` on models | Rules are policy. Opening them per-farm risks fragmenting the shared catalogue and undermining the shared-responsibility posture. |
| REI rule runs on-demand to generate Go/No-Go panel, and at application-log-time | It is not a pre-application candidate filter; it's a post-application state generator plus a conflict check at logging time. |
| Coincidence rules produce informational annotations, not filtering | Coincidence elevates urgency at the trigger level; at the rule level, it informs ranking rather than filtering candidates. |
| Tank mix compatibility treats unknown pairs as "prompt for jar test," not as blocks | Surfacing the gap is honest; blocking would be overly restrictive given the real-world practice of jar-testing. |
| Multi-trigger advisor sessions in V2 | Combining active triggers on the same block into one advisor session lets the engine recommend one spray for multiple targets, find tank-mix bundles that cover all, and honor the closed-loop thesis. Single-trigger sessions leave the cross-target integration to the grower — exactly the V1 pattern V2 is meant to close. |
| Substitution suggestions deferred to a post-V2 enhancement | Useful but requires finer-grained efficacy data than OMAFRA's 1–5 scale; honesty about what can be said today keeps V2 credible. |
| Engine failure isolation mirrors model-runner isolation | Same pattern; one bad rule should never halt a session. |
| Scenario-replay tests against 2026 V1 data are the primary correctness evidence | Unit tests prove rules are implemented as specified; scenario replay proves the system produces useful recommendations on real data. Both matter; scenario replay matters more to users. |

### 4.16 Open Questions Deferred from This Section

1. **Per-farm rule-enable overrides.** Not in V2. Revisit if a grower has a legitimate reason to suppress a rule (e.g., a rule that's known buggy and the fix is deferred). Meanwhile, global disable is the escape valve.
2. **Rule ordering within a category.** Rules within a category run in an undefined order. If one rule's output should feed another's (e.g., tank-mix rules that depend on FRAC rotation), the dependency lives in context, not in rule ordering. Flag if cases emerge where this is insufficient.
3. **Rule precedence when rules conflict.** Example: FRAC rotation rule warns "don't back-to-back Group 11," but the only in-stock product registered for this scab infection is Group 11. Current behavior: both rule evaluations are recorded, ranking handles trade-offs. If the grower has no alternative, the Group 11 ranks at the top with a warning. No explicit conflict resolution logic; the UI shows the tension. Revisit if this produces bad UX.
4. **Rate suggestion within recommendations.** The rule engine checks label ranges but doesn't recommend *which* rate within the range. Should it? Factors: pest pressure (from scouting and model), weather (reapplication interval under rain), disease history. Probably yes, but pushed to a post-V2 enhancement (Section 10).
5. **Late-season nitrogen cutoff date default.** Ontario bearing apples: typically mid-August is the rule-of-thumb; the precise date varies by region and variety. Implementation needs a defensible default with `per_farm_parameter_overrides` available. Research task for Section 6.
6. **How `coincidence_rules` fire triggers vs. how they inform ranking.** Defined at both layers in the schema; the boundary needs implementation care. Flag for Section 6 / scenario replay testing.
7. **Multi-trigger merge window default.** 24h named as default in 4.11; confirm with 2026 V1 retrospective.

---

## 5. Label Ingestion Workflow

The rule engine is only as good as the catalogue it reads from. Wrong PHI, wrong seasonal max, wrong tank-mix flags produce wrong recommendations, and — per Section 1.9 — wrong label data is a bug, not a content concern. This section specifies how product data enters the catalogue, how it stays current, and how quality is maintained.

Ingestion is the single most operationally demanding part of V2. The correctness bar is high because the liability posture depends on it.

### 5.1 Sources and Their Roles

Three sources inform catalogue entries, in order of authority:

- **Product label PDF (authoritative).** Every catalogue field that describes what a grower may or must do with a product traces to the label. Labels live on the PMRA Public Registry (pr-rp.hc-sc.gc.ca) or on manufacturer product pages. The label is the legal document; ingestion ends at the label.
- **OMAFRA Crop Protection Hub (accelerator).** OMAFRA's Hub aggregates efficacy ratings, recommended rates for specific targets on apple, and program-level guidance. It is a high-quality starting point, not a replacement for the label. Efficacy data (`product_efficacy` table) comes primarily from OMAFRA.
- **Manufacturer technical bulletins (supplementary).** For tank-mix compatibility details, pH sensitivity, adjuvant requirements, and other fine print that labels sometimes underspecify. Treated as advisory; conflicts with the label resolve to the label.

Priority at ingestion conflict: label > OMAFRA > manufacturer bulletin > everything else. Every field in the catalogue carries a source citation (`products.label_url`, per-extension field notes, `product_efficacy.source` and `source_url`).

### 5.2 Bootstrap Scope

The bootstrap catalogue is the set of products populated before V2 goes live for any invited grower. Scope defined as **hybrid** (Section 1.10 decision language):

- Every product currently in V1's seed catalogue (~30 products)
- Every product OMAFRA lists in its recommended programs for: apple scab, fire blight, powdery mildew, cedar rust, summer diseases, codling moth, oriental fruit moth, apple maggot, plum curculio, aphids and mites
- Every herbicide registered for apple in Ontario that OMAFRA lists in its weed management guidance
- Every foliar nutrient and soil amendment commonly used in Ontario apple production (OMAFRA Publication 360 nutrition sections)

Target bootstrap count: **~100–150 products**. This gets the catalogue to "genuinely useful for a real grower on day one" without 150+ hours of pre-launch author work. Products outside the bootstrap set are added on-demand via the grower request flow (5.9).

### 5.3 Ingestion Pipeline

Every product goes through the same pipeline. The pipeline has explicit stages and every product has a current stage visible to the author.

#### 5.3.1 Stages

1. **Discovery.** Product identified as in-scope. Created as a shell row in `products` with `ontario_registered = false` and `label_ingested_at = null`. Not yet recommendation-eligible.
2. **Scrape.** Automated fetch from OMAFRA Crop Protection Hub populates initial fields — trade name, active ingredients, target pests, efficacy ratings, recommended rates. Writes to `catalogue_ingestion_runs` with `source = 'omafra_scrape'`.
3. **Label attachment.** Label PDF URL added to `products.label_url`. PDF downloaded to local cache for manual review. `label_version` captured where the label is dated.
4. **Manual entry.** Author reviews the scraped fields against the label PDF and fills remaining fields (seasonal max, REI, REI-by-work-type, kickback, rainfast, tank mix known flags, etc.) directly from the label. This is where the real work happens.
5. **Verification.** A second pass — typically by the same author, sometimes a delegate — confirms every field against the label. `catalogue_ingestion_runs` records the verification with `source = 'manual_entry'`.
6. **Publication.** `ontario_registered` set to `true`, `label_ingested_at` set to current timestamp. Product is now recommendation-eligible.

The gate between stage 5 and 6 is the strict quality bar (Section 5.8). A product that has been scraped but not verified is not recommendation-eligible.

#### 5.3.2 Pipeline State Tracking

`products` table gains an implicit state via columns already defined in 3.4.1:

- `ontario_registered = false, label_ingested_at = null` → Discovery or Scrape
- `ontario_registered = false, label_ingested_at set` → Manual Entry or Verification
- `ontario_registered = true, label_ingested_at set` → Published

No separate state-machine column; the existing columns carry the status. An admin UI filter surfaces products in each implicit stage.

### 5.4 Scraping OMAFRA

Scraping is an accelerator, not an automation. The scraper fetches and stages data; a human verifies before publication.

**Scraping posture.** OMAFRA Crop Protection Hub is a Government of Ontario public resource. Scraping is designed to meet politeness norms:

- Rate-limited to no more than 1 request per 2 seconds; aggregate daily cap.
- User-agent header identifies the tool (`OrchardGuard/<version> (+contact_email)`).
- No circumvention of any access controls, pagination limits, or robots.txt directives.
- No redistribution of scraped data in bulk beyond V2's own use.
- Run as a scheduled batch (weekly during season, monthly off-season), not continuously.

**Outreach commitment.** Once bootstrap scraping is complete and V2 is in invited-beta use, the author contacts OMAFRA to disclose the tool, ask whether a preferred data feed exists, and offer to stop scraping if requested. Not a blocker for launch; a commitment for responsible ongoing use.

**What gets scraped.**

| Field | Source in OMAFRA Hub |
|---|---|
| trade_name, manufacturer, active_ingredients | Product detail page |
| frac_groups, irac_groups, hrac_groups | Product classification section |
| target_pests, target_weeds | Efficacy tables cross-referenced back to product |
| efficacy_rating per target | Star-rated efficacy tables |
| rate_min, rate_max, rate_unit (OMAFRA recommendation) | Rate recommendation tables |
| phi_days (OMAFRA-stated) | Program tables; always cross-verified against label |
| rei_hours (OMAFRA-stated) | Program tables; always cross-verified against label |
| label_url (where OMAFRA links to PMRA) | Where available |

**What is never scraped and is always manual from the label.**

- `seasonal_max_applications`, `seasonal_max_rate`, `min_days_between_applications`
- `rei_work_types` (per-work-type REI matrix)
- `min_tree_age_years`
- `growth_stage_min`, `growth_stage_max`
- `kickback_hours`, `rainfast_hours`
- `tank_mix_known_compatible`, `tank_mix_known_incompatible`, `tank_mix_notes`
- `resistance_risk`
- PCP number

These fields require reading the actual label text. Scraping them from summary tables — even OMAFRA's — is how wrong-data bugs enter the catalogue.

#### 5.4.1 Scraper Architecture

- Stateless Python script (or TypeScript; Section 9 decides). Runs on schedule via cron.
- Outputs JSON staging files, not direct database writes. Staged data is reviewable before promotion.
- Idempotent — re-running the scraper for the same product produces the same staging output. Changes to the staging output are treated as OMAFRA updates and flagged for author review.
- Diff detection. When OMAFRA's scraped data for a published product changes, the author is notified (email; feeds the same alert channel as trigger alerts but tagged as a catalogue event). The author reviews and either updates the catalogue or dismisses.

### 5.5 Manual Entry UI

The admin UI for catalogue curation is the workspace where bootstrap happens and where ongoing verification runs.

**Key UI patterns:**

- **Side-by-side label PDF + form.** Label PDF rendered in one pane; structured form in the other. The form is category-aware — pesticide products show the pesticide extension fields, herbicides show herbicide fields, etc.
- **Field provenance display.** Every field shows where its current value came from: "scraped from OMAFRA 2026-03-14" or "manually entered by author 2026-03-15." Hovering shows the source URL or the verification run ID.
- **Required-field checklist.** A product cannot move to the Verification stage until all required fields are populated. Optional fields are marked as optional.
- **Unresolved conflict flags.** When scrape and manual entry disagree, both values are shown with a required "resolve" action before publication. The resolution is recorded.
- **Verification signoff.** Clicking "Verify and Publish" writes the `catalogue_ingestion_runs` row, flips `ontario_registered`, and sets `label_ingested_at`. This is the strict quality gate (Section 5.8).

**Keyboard-first.** Bootstrap is data entry. The UI should be keyboard-navigable with shortcuts for common operations (mark field verified, move to next field, save and next product).

**Bulk operations.** A CSV import for products that share a clearly-defined schema (generic re-registrations, formulation variants). Always followed by per-product verification — CSV is a starting accelerator, not a bypass of the quality gate.

### 5.6 PDF Extraction — What's Automated and What Isn't

Automated PDF text extraction is available for the staging step, not the authoritative step.

- **Automated OCR and text extraction** runs on the label PDF to produce searchable text. This speeds up manual entry by letting the author search the label for specific terms ("seasonal max," "REI," "tank mix") rather than reading the whole document.
- **Structured field extraction from PDFs is not attempted.** Labels vary too much in layout for robust extraction; an extracted field that looks right but is wrong is more dangerous than a field the author had to read manually. If Section 8 load testing shows manual entry is the critical bottleneck, this decision can be revisited. V2.0 is manual-authoritative.

### 5.7 Catalogue Currency — Keeping Data Fresh

Labels revise. PHIs change. Products get re-registered or lose registration. Seasonal maxima shift. The catalogue can decay silently if no process keeps it current.

**Currency mechanisms:**

- **Scheduled scraper re-runs.** Weekly in-season, monthly off-season. Diffs against published data flag changes.
- **OMAFRA bulletin monitoring.** OMAFRA publishes update bulletins; the author subscribes and treats each bulletin as a trigger to review affected products.
- **PMRA registration status check.** PMRA publishes de-registrations. Quarterly, the scraper cross-checks every published product's PCP number against the Public Registry. De-registered products flip `ontario_registered = false` with a catalogue event.
- **Annual label re-verification pass.** Every published product is re-verified against its current label at least once per year. This is tracked via `label_ingested_at` — products with `label_ingested_at` older than 365 days surface in the admin UI as "needs re-verification."
- **Grower-reported discrepancies.** An in-app action lets a grower flag a product's catalogue entry as "doesn't match my label." The report creates a catalogue event routed to the author for review.

Products that fail currency (de-registered or failed re-verification) are not deleted. They remain in the catalogue with `ontario_registered = false`. Historical applications referencing them stay valid; future recommendations exclude them.

### 5.8 The Strict Quality Gate

Per Section 1.9 and confirmed for ingestion: a product is recommendation-eligible only after manual label verification. Scraped-but-unverified products are visible in the admin UI but do not appear in advisor candidate sets.

**Implementation:**

- Advisor candidate selection filters to `products.ontario_registered = true AND label_ingested_at IS NOT NULL`.
- Grower-facing product catalogue browsing (for inventory or curiosity) shows all products including unverified ones, with an explicit "unverified label data — not recommended by app" tag.
- The grower can manually log an application of an unverified product (inventory is inventory; the grower owns physical stock regardless of catalogue state). The application is recorded; the rule engine evaluates what rules it can and flags unverified-data rules as `not_applicable`. The application enters `rule_overrides` with a system-generated reason.

The strict gate makes the shared-responsibility posture honest: when V2 recommends a product, every field the recommendation depended on has been human-verified against the label.

### 5.9 Grower Product Requests

Invited growers will have products V2 doesn't know about. The request flow:

1. Grower hits a product gap (e.g., logging inventory of a product not in the catalogue).
2. Grower opens the "request product" action, fills a minimal form: trade name, manufacturer, PCP number if known, label URL or uploaded PDF, intended use.
3. System creates a catalogue request entry (new table `catalogue_requests` — added here rather than Section 3 because it's request-layer, not core schema).
4. Author receives notification, triages, and either: accepts (runs the product through the ingestion pipeline), defers (marks "under review"), or declines (out of scope with reason).
5. Requesting grower is notified of the outcome.

**`catalogue_requests` table (addendum to Section 3):**

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| farm_id | bigint NOT NULL FK → farms(id) | |
| requested_by | bigint NOT NULL FK → users(id) | |
| trade_name | text NOT NULL | |
| manufacturer | text NULL | |
| pcp_number | text NULL | |
| label_url | text NULL | |
| uploaded_label_path | text NULL | |
| intended_use | text NULL | |
| status | text NOT NULL | enum: `requested`, `under_review`, `accepted`, `declined` |
| resolution | text NULL | Author's response when status is accepted or declined. |
| product_id | bigint NULL FK → products(id) | Set when accepted and ingested. |
| requested_at | timestamptz NOT NULL DEFAULT now() | |
| resolved_at | timestamptz NULL | |

### 5.10 Efficacy Data Ingestion

Efficacy ratings (`product_efficacy`) are ingested from OMAFRA separately from label data, with their own lighter workflow.

- Scraper extracts per-product-per-target efficacy ratings from OMAFRA program tables.
- Ratings are stored with `product_efficacy.source = "OMAFRA Crop Protection Hub"` and `source_url` to the scraped page.
- Manual verification for efficacy is lighter than for label data: the author spot-checks, not field-by-field verifies. Efficacy drives ranking, not hard rules; a wrong efficacy rating produces a worse recommendation, not an off-label one.
- Efficacy is re-scraped with the same schedule as label data.

### 5.11 Ingestion Workflow for Nutrition Products

Foliar nutrients and soil amendments have different sourcing. Many are not PMRA-registered (nutrients generally aren't pesticides). The ingestion pipeline accommodates:

- **PMRA Public Registry** is skipped for most nutrition products.
- **OMAFRA Pub 360 nutrition sections** become the primary OMAFRA source for nutrition.
- **Manufacturer technical data sheets** substitute for the "label" as the authoritative document. These are typically PDFs on manufacturer websites.
- Rate guidance and seasonal caps are often advisory rather than regulated; the `severity: advisory` vs. `severity: hard` split in Section 4.6.10 is what makes this workable.

The ingestion pipeline stages are the same; the sources differ.

### 5.12 Ingestion Workflow for Herbicides

Herbicides follow the same pipeline as pesticides. HRAC group data comes from the Herbicide Resistance Action Committee's published lists (hracglobal.com); this source is cross-referenced with the label. Canadian apple-registered herbicides are a smaller set than pesticides (~40–80 products), making bootstrap tractable.

### 5.13 V1 Catalogue Migration

V1's ~30 seeded products are the starting point for V2's bootstrap. They migrate as follows:

1. Each V1 product is created as a V2 `products` row with category inferred from V1's `product_group`.
2. Extension rows are created based on category.
3. V1 fields map as documented in Section 3.16; missing fields (e.g., `rei_work_types`, `min_tree_age_years`, `hrac_groups`) are left null.
4. Each migrated product enters the pipeline at the Manual Entry stage — V1's product data is treated as scraped-quality, not verified. The author runs each one through the full ingestion pipeline to move it to Published.
5. Product name reconciliation (V1 models reference "Captan" but DB has "Captan 80 WDG") happens during migration by establishing a canonical `trade_name` per product and retiring free-text mismatches.

Expected migration effort: ~15–20 hours to move V1's catalogue through the V2 pipeline to published state. This is part of bootstrap, not separate from it.

### 5.14 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| Product label PDF is the authoritative source | Labels are the legal document; all other sources are accelerators or cross-checks. |
| OMAFRA Crop Protection Hub is a scraping-assisted accelerator | Good-quality aggregated data; dramatically faster than pure-manual entry; does not replace the label. |
| Bootstrap scope is hybrid: ~100–150 products covering V1 + OMAFRA-recommended programs | Balances day-one usefulness against author workload. Full "all registered products" coverage is a goal, not a launch gate. |
| Strict quality gate: only verified products are recommendation-eligible | Makes the shared-responsibility posture honest. Grower can still log applications of unverified products; advisor won't recommend them. |
| Automated structured field extraction from PDFs is not attempted in V2 | A wrong extracted field is worse than a field the author had to read. Manual entry is the floor; OCR-assisted search helps without automating. |
| Scheduled scraper re-runs with diff detection | Labels revise; the catalogue can't be static and correct. |
| Annual label re-verification is required | `label_ingested_at` older than 365 days surfaces the product for re-verification. |
| De-registered products are retained with `ontario_registered = false` | History needs to stay valid; future recommendations exclude them. |
| Grower product requests route through the author for ingestion | Shared catalogue correctness; no grower bypass of the quality gate. |
| `catalogue_requests` is a new table in the schema | Added via this section as an addendum to Section 3. |
| V1 catalogue migrates in but requires re-verification | V1 product data is treated as scraped-quality, not published-quality. Running each through the pipeline is part of bootstrap. |
| Efficacy data verification is lighter than label data verification | Efficacy drives ranking, not hard rules; wrong efficacy is worse recommendation, not off-label. |
| Catalogue writes are gated to `author` role, not farm owners | Shared correctness concern; per-farm edits would fork the catalogue. Exceptions route through per-farm parameter overrides (Section 3.9.6), not catalogue edits. |
| Proactive outreach to OMAFRA once V2 is in beta | Responsible use of a government data source; opens the door to a preferred mechanism if OMAFRA has one. |

### 5.15 Open Questions Deferred from This Section

1. **Scraper stack.** Language and library choice (Python+BeautifulSoup vs. Playwright vs. TypeScript) deferred to Section 9.
2. **Label PDF storage.** Labels are redistributed under fair-use for verification purposes; storing them in the V2 database as base64 BLOBs, storing them in object storage, or keeping only `label_url` — storage choice deferred to Section 9.
3. **OMAFRA outreach timing.** Before or after invited-beta launch? My default: after, once the tool is demonstrably useful and we have something to show. Flag if you want to reach out earlier.
4. **Bulk efficacy re-ratings.** OMAFRA occasionally republishes efficacy tables with methodology changes. How do re-rating events propagate — invalidate all affected products, require re-verification, or roll forward with a diff-surfaced alert? Implementation detail for Section 6.
5. **Multi-jurisdiction scope.** Labels vary by jurisdiction — an apple product registered in Ontario may not be registered in BC. V2 is Ontario-only, but `orchards.latitude/longitude` could someday drive jurisdictional filtering. Out of V2; flagged for Section 10.
6. **Catalogue request volume estimate.** How many requests will invited growers generate? Unknown until beta. If request volume outpaces author capacity, the Strict gate becomes a bottleneck. Mitigation: if volume is a problem, hire contract ingestion help or invite trusted growers to contribute verifications subject to author review.
7. **Bootstrap completeness signal.** When is the bootstrap "done enough" to invite the first outside grower? Proposal: bootstrap is done when every product an invited grower's V1 spray history references (from their V1 data export) is Published in V2. That makes bootstrap grower-specific; revisit in Section 8.

---

## 6. Model Correctness Strategy

V2's recommendations depend on models being faithful implementations of their published sources. A model that looks right but is subtly wrong — wrong parameter, wrong unit conversion, wrong edge case — produces bad recommendations with no immediate visible symptom. This section specifies how V2 makes models trustworthy and keeps them that way.

The correctness gate for V2 has two layers:

- **Published-fixture gate (universal).** Every model in V2 — Tier 1, surviving Tier 2, Tier 3 — implements at least one fixture test drawn from a published source (or a documented constructed fixture where no published worked example exists). Fixture tests pass before the model merges.
- **Scenario-replay gate (Tier-1 only).** For the four highest-stakes models (apple scab, fire blight, codling moth, frost risk), the author runs the V2 model against 2026 V1 season data and writes a review of whether V2's recommendations on that season would have been defensible. The replay is **not** a comparison to V1's output — V1 is not the benchmark. The replay compares V2's output to published-algorithm expectations, OMAFRA guidance, and what actually happened (condition materialized or didn't).

Neither gate is agronomist sign-off (Section 1.9). Together they are the minimum defensible bar for a tool driving real spray decisions by certified growers.

This section incorporates the scope-trimming decision from Section 2.1: V2 ships with roughly 18–20 models, not 55. Scope-trimmed Tier-2 conditions become SCOUTING-SURFACEs with no risk score; those surfaces carry no fixture requirement because they aren't models.

### 6.1 The Correctness Problem

Four failure modes the strategy protects against:

1. **Algorithmic drift from source.** The published paper specifies a formula; the implementation deviates. Example: Mills Table uses specific temperature bands; an implementation that interpolates differently between bands produces different infection calls.
2. **Parameter error.** Right formula, wrong numbers. Example: CougarBlight's temperature cap at 31°C with linear decline to 40°C — a misread of the graph that caps at 32°C subtly shifts every risk threshold downstream.
3. **Unit error.** Right formula, right numbers, wrong units. Example: Degree-hours vs. degree-days, Fahrenheit vs. Celsius, inches vs. mm of rain. Silent and pervasive.
4. **Edge-case error.** Correct under typical conditions, wrong at boundaries. Example: a bloom-stage-gated model behaves unexpectedly at the exact moment of stage transition; a DD-accumulator handles a biofix date equal to the query date inconsistently.

Unit tests catch what the author thought to test. The published-fixture gate catches drift from the canonical reference the author was implementing in the first place. The scenario-replay gate catches calibration and edge-case problems that fixtures can miss because the fixture's inputs are too clean.

### 6.2 Citation Structure

Every model file (V1 naming: `lib/models/<slug>.ts`, V2 TBD in Section 9) carries a header block with structured citation metadata:

```
/**
 * Model: Apple Scab Infection
 * Slug: apple_scab
 * Algorithm: Modified Mills Table + NH ascospore maturity curve
 * Primary source:
 *   MacHardy, W. E. (1996). Apple Scab: Biology, Epidemiology, and Management.
 *   APS Press. Chapter 7, Table 7.2 (Mills infection periods).
 * Secondary source:
 *   Gadoury, D. M., & MacHardy, W. E. (1982). A model to estimate the maturity
 *   of ascospores of Venturia inaequalis. Phytopathology, 72(7), 901–904.
 * OMAFRA reference:
 *   OMAFRA Publication 310, Apple Scab Management.
 *   https://www.ontario.ca/page/apple-scab
 * Parameters version: 2026-03-14
 * Last reviewed against source: 2026-03-14 by <author>
 */
```

The header is not decorative. At startup, the model runner reads these headers and populates an in-database `model_citations` table — NEW, added here as an addendum to Section 3. This makes citations queryable by the UI (so every recommendation surface can display the source) and by admin tools (so "which models haven't been reviewed in > 12 months" is answerable).

**`model_citations` table (addendum to Section 3):**

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| model_slug | text NOT NULL UNIQUE | |
| algorithm_name | text NOT NULL | |
| primary_source | text NOT NULL | Full citation. |
| primary_source_url | text NULL | |
| secondary_sources | jsonb NOT NULL DEFAULT '[]' | Array of {citation, url}. |
| omafra_reference | text NULL | |
| omafra_reference_url | text NULL | |
| parameters_version | text NOT NULL | Date or version tag. |
| last_reviewed_at | date NOT NULL | |
| last_reviewed_by | text NOT NULL | |
| created_at | timestamptz NOT NULL DEFAULT now() | |
| updated_at | timestamptz NOT NULL DEFAULT now() | |

The row is maintained by a startup reconciliation that reads the header block and updates the row; divergence between code header and DB row is a catalogue event.

### 6.3 Test Fixture Sources

Published fixtures come from four source types, in rough priority:

- **The original paper's worked examples.** Most defensible. The paper author published this to demonstrate the algorithm. Example: Mills 1944 includes tables showing infection calls for specific temperature × wetness combinations; these become test cases directly.
- **OMAFRA / Cornell / Washington State extension worked examples.** Extension services often publish "here's how the Mills Table works" walkthroughs with numeric examples. Less primary but very usable.
- **Subsequent validation papers.** Later papers that tested the original model often publish their test conditions and results. These double-check the original.
- **Author-constructed fixtures, explicitly marked.** For models where no published worked example exists, the author constructs a fixture from first principles, documents the construction in the test file, and marks the fixture as `constructed`. These are weaker evidence but still useful regression protection.

Every fixture in the codebase is tagged with its source type.

### 6.4 Fixture File Structure

Tests live alongside model source files. Directory convention (V2 structure finalized in Section 9):

```
lib/models/
  apple-scab.ts
  apple-scab.test.ts         // unit tests (synthetic inputs)
  apple-scab.fixtures.ts     // published-fixture tests
  apple-scab.fixtures.json   // fixture data
```

A fixture file contains one or more test cases of the shape:

```
{
  "fixture_id": "mills_1944_table_7_2_row_3",
  "source_type": "original_paper",
  "source_citation": "Mills 1944, Table 7.2, row 3",
  "source_url": null,
  "description": "10°C mean temperature, 16h wetness → light infection per Mills",
  "inputs": {
    "hourly": [...],
    "daily": [...],
    "bloom_stage": "petal_fall",
    "petal_fall_date": "2024-05-14",
    "evaluated_at": "2024-05-20T08:00:00Z"
  },
  "expected": {
    "infection_level": "light",
    "wet_period_end": "2024-05-19T23:00:00Z",
    "wet_period_duration_hours": 16
  },
  "tolerance": {
    "wet_period_duration_hours": 0.5
  }
}
```

Tolerances are explicit. Floating-point comparisons in published data rarely round-trip exactly; the fixture declares how close is close enough. Tolerances are a design decision per-fixture, not a global knob.

### 6.5 Which Models Require Published Fixtures

After scope trimming (Section 2.1.1 and 2.1.2), V2 ships with roughly 18–20 models plus nutrition evaluators. Fixture requirements proportional to tier:

- **Tier 1 (apple scab, fire blight, codling moth, frost risk).** Published fixtures **required and mandatory**. At least one per model, sourced from the primary paper or an OMAFRA-equivalent. Merge blocked without passing fixture tests. Also subject to the scenario-replay gate (Section 6.10).
- **Tier 2 surviving as models (powdery mildew, sooty blotch/flyspeck, bitter pit, phytophthora, sunburn, water core).** Published fixtures **required where they exist**. For powdery mildew (Gubler-Thomas), sooty blotch (Brown & Sutton), and bitter pit — fixtures exist and are mandatory. For phytophthora, sunburn, water core — author-constructed fixtures with documented construction are acceptable where no published worked example is found. Constructed fixtures carry the explicit `constructed` tag.
- **Tier 3 surviving as models (~10 pest DD-timing models).** A single published fixture for the DD accumulator itself, plus unit tests per pest for its specific base temperature and stage thresholds. Shared fixture infrastructure means marginal fixture cost per pest is low.
- **Tier 4 demoted / scouting-surface conditions.** No fixtures. SCOUTING-SURFACEs produce advisory context, not risk scores; there's no model output to fixture against. Static reference content similarly has no fixture target. Editorial review replaces fixture review for these.
- **Nutrition evaluators.** Fixtures required. Fixture source is OMAFRA Publication 360 sufficiency-band worked examples where they exist; constructed fixtures with OMAFRA citation otherwise. Hard nutrition rules (late-season N cutoff, labeled seasonal caps, pH overshoot — see Section 4.6.10) have mandatory fixtures because they block; advisory nutrition rules accept constructed fixtures.
- **Weed evaluators.** Weather-window selectors, not epidemiological models. Unit tests sufficient; no fixture requirement.

The CI system enforces the fixture requirement by tier via the citation header's declared tier. A Tier-1 model PR without fixture tests fails to merge; a constructed-fixture-only Tier-1 model PR also fails (Tier 1 must have published fixtures).

### 6.6 What Fixture Tests Actually Check

A fixture test asserts that, given the fixture's inputs, the model produces output matching `expected` within `tolerance`. Three categories of check:

- **Output equality on primary outputs.** The model's `riskLevel`, `riskScore`, or equivalent primary output matches.
- **Intermediate value checks.** For models where the primary output is coarse-grained (light / moderate / severe), intermediate values are also checked: wet period duration, cumulative DD, degree-hours, ascospore maturity percentage. A model can arrive at the right coarse answer via the wrong path; intermediate checks catch that.
- **Error-path checks.** Fixtures can also assert expected errors: "given a bloom stage of 'dormant' and a petal_fall_date in the future, the model returns not_applicable, not an exception." Edge-case protection.

### 6.7 The Gate in Practice

A model is merge-eligible when:

1. It has a citation header that reconciles with `model_citations`.
2. It has unit tests covering the author's identified test cases.
3. It has at least one fixture test per its tier's requirement (published for Tier 1; published-or-constructed for Tier 2/3 per 6.5).
4. All tests pass in CI.
5. A human (initially the author; delegates allowed as the project grows) has reviewed the PR.

Additionally, a **Tier 1 model is advisor-enabled only after scenario replay review passes** (Section 6.10). A Tier-1 model can be merged and live in the codebase with fixtures passing, but it does not drive advisor recommendations until the scenario-replay artifact exists and is signed off. This two-phase gate — merge-eligible vs. advisor-enabled — is implemented via a registry flag; Section 4.9's rule-registry pattern extends to models.

When any merge-eligibility requirement is missing, CI blocks merge. The advisor-enablement step is tracked via a checklist in the PR template for Tier 1 models and a registry entry once replay is complete.

### 6.8 Parameter Drift Detection

Bugs enter not only at model creation but during refactors. A helpful refactor that inlines a constant, consolidates a helper, or tightens types can silently shift a parameter. Detection:

- **Parameter hash on every model run.** `model_runs.model_parameters_hash` (Section 3.7.1) captures a hash of the parameter set in effect at run time.
- **Snapshot fixtures.** Every model has at least one "snapshot" fixture that exercises the full parameter set — a synthetic scenario that is sensitive to every tunable. If the snapshot output changes, the hash changes, and CI surfaces the diff. Intentional changes (parameter revision, algorithmic correction) are acknowledged explicitly in the PR.
- **Per-run citation of parameters version.** `model_citations.parameters_version` is bumped when parameters change. The recommendation UI displays the parameters version the recommendation was based on.

This does not prevent drift; it makes drift visible. The visibility is the safeguard.

### 6.9 The Nutrition Correctness Challenge

Nutrition evaluators face a different correctness problem than disease/pest models. The published evidence base for "leaf calcium below X ppm at mid-season → bitter pit risk" is thinner than Mills. Sufficiency ranges from OMAFRA Pub 360 are derived from accumulated field experience plus some foundational research, but point estimates with uncertainty envelopes aren't routinely published.

Mitigations:

- **Citations must be specific.** "OMAFRA Pub 360 leaf analysis table for apples, Ca sufficiency range 1.3–1.8%" is citable. "Calcium is important" is not.
- **Fixtures are OMAFRA-example-driven.** OMAFRA's nutrition sections include worked examples of interpretation; these become fixtures.
- **Advisory-only for most rules.** Per Section 4.6.10, most nutrition rules are advisory. The correctness cost of a wrong advisory rule is lower than a wrong hard rule — the grower sees a warning and exercises judgment.
- **Exception: hard nutrition rules (late-season N cutoff, labeled seasonal caps, pH overshoot) require fixtures.** Because they block rather than advise, they must be defensible.
- **The inherent limit is acknowledged.** The project doc (this section) states plainly that nutrition's evidence base is thinner than disease/pest; recommendations carry a "nutrition guidance is based on OMAFRA Pub 360 and accumulated field experience" note; growers are invited to contribute disagreements.

### 6.10 Scenario Replay for Tier-1 Models

The scenario-replay gate applies to the four Tier-1 models only: apple scab, fire blight, codling moth, frost risk. These are the models with the highest-consequence recommendations, the richest published validation materials, and the most V1 operational data to replay against.

**What scenario replay actually means in V2.** Not "does V2 match V1." V1 is not the benchmark. The replay uses 2026 V1 conditions — weather, orchard state, bloom progression, spray history, observed outcomes — as the *test input* for V2's model. The question is: given these 2026 conditions, does V2's output match what the published algorithm says should happen, match OMAFRA's program-level guidance, and make sense against what actually occurred in 2026?

**Replay workflow per Tier-1 model:**

1. Author selects 10–20 representative days from the 2026 season covering diverse conditions (pre-bloom, bloom, petal fall, primary scab season, summer, harvest approach). Each day's weather and orchard state become a replay scenario.
2. V2 model runs against each scenario; output is captured.
3. Author reviews each output against: (a) published-algorithm expectation — the fixture-level check extended to real conditions; (b) OMAFRA's guidance for those conditions; (c) 2026 ground truth — did the condition materialize, did the grower spray, was the decision correct in hindsight.
4. Author writes a replay review artifact stored in the repo: `replay/<model_slug>_2026.md`. Structure: scenarios covered, V2 output summary, divergences from expectation, calibration notes, recommendation on whether V2 ships this model advisor-enabled.
5. PR updates the registry to advisor-enable the model. Merge of that PR is the gate passing.

**What replay catches that fixtures don't.** Fixtures use clean inputs (the paper's example data). Real weather data has noise, gaps, odd source conflicts, partial hours. A model that passes fixtures can still behave oddly on real data — time-zone boundaries, missing leaf-wetness when estimation diverges from reality, unusual weather patterns. Replay surfaces these.

**Note on weather source during replay.** V2's scenario replay runs against 2026 V1 data — which means Open-Meteo + Environment Canada sources, with leaf-wetness estimated rather than directly measured. On-orchard weather hardware is deferred to 2027 (Section 9.10). This means the replay corpus reflects the same weather-source posture V2 launches under; replay is valid for what V2 will actually see in production. When a station is installed post-launch, a second replay pass on the hardware-equipped data is a natural follow-up.

**Budget:** 40–80 hours to build replay tooling (data loading, model harness, output capture, diff surfacing). Per-model replay review: 4–8 hours × 4 Tier-1 models = 16–32 hours. Total scenario-replay work: 56–112 hours pre-launch.

**Non-Tier-1 models** do not require scenario replay to ship. Post-launch, replay extends to other models as useful — the tooling built for Tier 1 is reusable. This is the "important but not launch-blocking" bucket.

### 6.11 V1 Model Carry-Forward

V1 has 55 models, most of which do not survive Section 2.1's scope trimming. The correctness workflow at V2 bootstrap, revised to match trimmed scope:

- **V2 Tier-1 models (4).** Full fixture work + scenario replay. Published fixture sourcing is real work — finding the right worked example, transcribing inputs, validating the implementation. ~6–10 hours per model for fixtures, 4–8 hours per model for replay review. ~40–72 hours total for Tier 1.
- **V2 Tier-2 surviving models (~6).** Fixture work only, mixed published and constructed. ~3–6 hours per model. ~20–40 hours total.
- **V2 Tier-3 surviving models (~10).** Shared DD-accumulator fixture + per-pest unit tests. ~1–2 hours per pest. ~10–20 hours total.
- **Nutrition evaluators (~6).** Fixture work, OMAFRA-driven. ~3–5 hours per evaluator for hard-rule evaluators; less for advisory. ~15–25 hours total.
- **SCOUTING-SURFACEs.** No fixture work; editorial review of the advisory content and scouting guidance. ~1 hour per surface × ~15 surfaces = ~15 hours.
- **CUT or DEMOTE content.** No work.

**Revised pre-launch model correctness budget:**

- Fixture work across surviving models: ~85–155 hours
- Scenario replay tooling: 40–80 hours
- Scenario replay reviews (Tier-1 only): 16–32 hours
- Scouting-surface editorial review: ~15 hours

**Subtotal: 156–282 hours of model correctness work pre-launch.** This is tighter than the blanket-promotion estimate (235–440h) and tighter than the no-replay estimate (40–80h for fixtures alone) because scope trimming paid for the added replay work.

**V1 models that survive scope trimming do not get a fixture pass.** Every V2 model that ships goes through the full gate, whether it's a V1 carry-forward or a rebuild. The rebuild is the opportunity to get citation, parameters, and fixture right.

### 6.12 The Author's Blind-Spot Problem

The author is not an agronomist. Some errors are invisible to the author even with fixtures and scenario replay in place — errors where the author and the source share an assumption that is itself wrong, or where domain intuition is needed to recognize an implausible output.

V2 does not solve this. It contains it with four mitigations:

- **Scenario replay on Tier-1 models (Section 6.10).** The highest-stakes models are reviewed against real 2026 conditions before shipping, not just against fixture inputs. This catches calibration and edge-case issues that clean fixtures miss.
- **OMAFRA as a sanity check.** When the advisor surfaces a recommendation, the grower can cross-reference against OMAFRA Crop Protection Hub's own recommendation for the target. Disagreement is a flag the grower or author investigates.
- **Informal peer review from invited growers.** Once V2 has invited users, growers who develop familiarity with the tool become informal reviewers. Their feedback ("the app told me to spray X, but that doesn't match what I'd normally do here") is a low-cost signal.
- **Planned post-V2 extension: replay to non-Tier-1 models, and agronomist review.** Section 10 flags both as future work. Extending scenario replay to Tier-2 and Tier-3 models becomes natural once the replay tooling is in place; agronomist partnership becomes appropriate when broadening to non-certified users.

The blind-spot risk is explicit, not papered over. The shared-responsibility posture in Section 1.9 depends on the grower understanding that V2 is decision-support with acknowledged gaps.

### 6.13 Continuous Correctness — Post-Launch

Correctness work doesn't end at launch.

- **Annual fixture review.** Every model's `model_citations.last_reviewed_at` older than 12 months surfaces in the admin UI. The author re-reads the source, confirms parameters, touches the review date.
- **Source updates.** When OMAFRA publishes revised guidance, affected models are flagged for review. Same channel as catalogue-currency alerts (Section 5.7).
- **Grower-reported model disagreements.** An in-app action — parallel to the "report catalogue discrepancy" flow — lets growers report "this model's recommendation doesn't match my experience here." Reports are triaged by the author. Useful even without agronomist oversight: a grower seeing five clean seasons on an uninoculated block knows the scab risk was overstated.

### 6.14 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| Two-layer correctness gate: published-fixture (universal) + scenario replay (Tier-1 only) | Fixtures catch algorithmic drift, parameter error, unit error, edge cases against clean inputs. Scenario replay catches calibration and edge cases that fixtures miss on real data. Applied asymmetrically by tier because consequence is asymmetric. |
| V1 is not the scenario-replay benchmark | Replay compares V2 output to published-algorithm expectation, OMAFRA guidance, and 2026 ground truth. V1's output is context, not target. V2 must do better than V1, not match it. |
| Scope trimming: V2 ships ~18–20 models, not 55 | V1's thin Tier-2 threshold models are estimates dressed as models. Keeping them with a higher correctness bar validates implementations of weak algorithms. Trimming produces a smaller, better-supported set. |
| SCOUTING-SURFACE disposition for scope-trimmed Tier-2 conditions | Honest representation of what V1's thin models were trying to do: surface context, prompt the grower to look. No risk score, no fixture, no pretense of precision. |
| Citation header + `model_citations` table | Makes sources queryable, displayable in the UI, and auditable for freshness. |
| Fixture requirements asymmetric by tier | Tier 1 mandatory published fixtures. Tier 2 published-or-constructed. Tier 3 shared DD-accumulator fixture plus per-pest unit tests. Weed evaluators none (not epidemiological). |
| Constructed fixtures are allowed for non-Tier-1 models but explicitly tagged | Honest about evidence strength; preserves regression protection where published validation doesn't exist. Tier 1 cannot substitute constructed fixtures. |
| Nutrition hard rules require fixtures; advisory rules accept constructed | Hard rules block; hard rules need defensible evidence. Advisory rules warn; warnings tolerate softer evidence. |
| Parameter hash on every model run + snapshot fixtures | Drift is detected at CI time and recorded per run; the recommendation UI can cite the parameters version. |
| Author is the reviewer; delegates allowed as the project grows | Realistic for V2 staffing; not agronomist sign-off, aligned with Section 1.9. |
| Merge-eligible vs. advisor-enabled distinction for Tier-1 models | A Tier-1 model can exist in the codebase with passing fixtures and not yet drive recommendations. Advisor-enablement requires scenario replay review. Implemented via registry flag. |
| The author's blind-spot problem is acknowledged and mitigated, not solved | Scenario replay on Tier 1, OMAFRA cross-reference, invited-grower peer review. Post-V2: extended replay, agronomist partnership. |
| Weed evaluators do not require fixtures | They are weather-window selectors, not epidemiological models. Unit tests cover the logic. |
| Annual review cadence for citations; source-update alerts for revision events | Correctness decays without active maintenance. Same pattern as catalogue currency (Section 5.7). |
| In-app grower discrepancy reports for models and recommendations | Low-cost peer review from certified users who live with the recommendations. |
| Pre-launch model correctness budget: 156–282 hours | Scope trimming paid for the scenario-replay promotion. Tighter than a blanket-replay approach would have been. |
| Scenario replay tooling is V2 investment; extending replay to non-Tier-1 models is post-V2 | Tool built once is reusable. Post-launch, extending replay to Tier 2 and Tier 3 is a natural follow-on. |

### 6.15 Open Questions Deferred from This Section

1. **Fixture repository licensing.** Published-paper-derived fixtures may need to be redistributed with the tool. Licensing of Mills (1944, presumably public domain), MacHardy 1996 (copyrighted), and extension publications (varies) is a legal review, not a design decision. Fixtures can cite tables without reproducing them. Flag for Section 7 liability work.
2. **Snapshot fixture sensitivity calibration.** A snapshot fixture testing every parameter produces hash noise on intentional edits. Calibration needed at implementation time.
3. **Grower-reported disagreement triage workflow.** Volume unknown; triage is author's. Mirrors Section 5's catalogue-request volume concern. Mitigation path: contract review or trusted-grower triage contributors.
4. **Model deprecation fallback.** When a model is found wrong and no correct replacement exists, advisor skips it with a visible "temporarily unavailable" note. Implementation detail, registry-flag-driven.
5. **Parameters version retrospective.** When a model's parameters change, past `model_runs` rows are stamped with the old version. Allow "re-run 2026 with 2027 parameters" for retrospective? Useful but non-trivial; post-V2.
6. **Scope-trimmed content accuracy.** SCOUTING-SURFACE content (cedar rust, black rot, bitter rot, etc.) and DEMOTE content (voles, deer, mosaic) is advisory or static but still needs to be correct. Lightweight "reference content review" checklist in PR template covers this; no fixture.
7. **Late-season nitrogen cutoff default research.** Mid-August rule-of-thumb; precise date varies by region. Research task before the hard rule ships.
8. **Fixture sourcing for Gubler-Thomas upgrade.** Powdery mildew upgrade requires UC Davis extension materials. Pre-implementation task.
9. **Scenario-replay scenario selection methodology.** 10–20 representative days per Tier-1 model — but which days? Proposal: cover each phenology transition plus high-risk weather events plus a representative sample. Needs a methodology document before first replay run.
10. **Replay review artifact format standardization.** `replay/<model_slug>_2026.md` — structure and required sections are sketched in 6.10; formalize as a template before first replay.

---

## 7. Liability and Disclaimer Approach

Previous sections established posture commitments — certified users (Section 1.4), shared-responsibility launch (Section 1.9), advisor not prescriber (Section 1.5), label data as correctness concern (Section 1.9), hard/advisory rule split (Section 4.5), override logging (Section 4.8), strict catalogue quality gate (Section 5.8), published-fixture + scenario-replay model gate (Section 6). This section turns those commitments into concrete artifacts: a Terms of Service document, per-user acknowledgments, UI disclosure requirements, and internal policies that make the posture operational rather than aspirational.

V2 uses both a formal ToS and per-user acknowledgments. The ToS is the durable legal framing; the acknowledgments are where the grower personally affirms the commitments each time they matter. Neither replaces the other.

### 7.1 Who Bears Responsibility

V2's liability posture rests on three parties, with explicit roles:

- **The applicator (grower / invited user).** Legally responsible for every application they make. Holds a current Ontario Grower Pesticide Safety Certificate or Farmer Pesticide License. Reads product labels. Exercises professional judgment. Decides what goes on their trees.
- **The tool (V2 / OrchardGuard).** Decision-support, not prescription. Surfaces model outputs, filtered candidate lists, rule evaluations, citations, and disclaimers. Operates correctly per its fixtures, scenario replay, and published sources.
- **The author / operator.** Maintains catalogue accuracy, model correctness, and ingestion currency. Triages discrepancy reports. Publishes ToS updates when posture or scope changes materially.

No agronomist is in this model. Section 1.9 makes that explicit; Section 7 operationalizes it. If V2's audience ever broadens to non-certified users, the liability model changes and agronomist review becomes a release gate — tracked in Section 10.

### 7.2 Terms of Service — Structure and Scope

The ToS is a web-hosted document at a stable URL, versioned, displayed in-app. It is not a shrinkwrap wall-of-text; it's a short, readable document that invited growers can actually read.

**Required sections:**

1. **Purpose and scope.** What V2 is (decision-support for certified Ontario apple growers, conventional/IPM) and is not (prescription, organic, non-apple, public SaaS).
2. **User eligibility.** Current Ontario Grower Pesticide Safety Certificate or Farmer Pesticide License required. Self-attested by the user; the operator does not verify certification status externally.
3. **User responsibilities.** Read labels. Verify recommendations against labels before applying. Log applications accurately. Report discrepancies. Maintain proper training and PPE. Comply with all federal, provincial, and municipal regulations.
4. **Operator responsibilities.** Maintain catalogue accuracy per the strict quality gate. Publish source citations for every recommendation. Disclose model correctness posture. Notify users of material changes. Retain data per the retention policy.
5. **Limitations and disclaimers.** V2 is advisory. Recommendations are not guarantees. Model outputs can be wrong; catalogue data can be stale; weather predictions can be incorrect; the user's professional judgment is the final decision. The operator is not liable for crop loss, off-label application, resistance development, environmental harm, or regulatory violation resulting from the user's application decisions.
6. **Data ownership and use.** Farm data (orchard configuration, spray logs, scouting observations, test results, photos) is the user's. The operator holds it to run V2 and for the retention period. Data is not sold or shared beyond the farm. Farm data may be used to train or improve ML systems only with the user's explicit opt-in consent, captured separately from ToS acknowledgment and revocable at any time. Anonymized, aggregated usage analytics may inform V2 development; identifiable farm data never leaves the user's tenant absent opt-in.
7. **Access revocation.** The operator may revoke access for ToS violations. The user may terminate their account and request data export at any time.
8. **Dispute resolution.** Good-faith direct communication is the first step. Applicable jurisdiction for disputes (likely Ontario; confirm with legal review).
9. **ToS changes.** Material changes trigger re-acknowledgment (Section 7.5). Non-material changes (typos, clarifications) are versioned but don't require re-signoff.
10. **Effective date and version.**

**What the ToS is not:**

- Not a warranty. V2 is provided as-is.
- Not a service-level agreement. No uptime guarantees for invited beta.
- Not a support contract. The author provides best-effort support; formal SLAs are out of scope for V2.

### 7.3 Per-User Acknowledgments

The ToS establishes the frame. The acknowledgments are where each user personally affirms the specific commitments that make shared responsibility honest. Acknowledgments are captured at two points:

**At invitation acceptance (first-time):**

- "I hold a current Ontario Grower Pesticide Safety Certificate or Farmer Pesticide License. I understand the operator does not verify this externally; I am attesting truthfully."
- "I understand OrchardGuard provides decision-support, not prescription. Every recommendation is advisory. I read the product label before every application and verify the recommendation against the label."
- "I accept legal responsibility for every application I make, including applications I log in OrchardGuard, regardless of what OrchardGuard recommended."
- "I understand OrchardGuard's model outputs are based on published algorithms and OMAFRA guidance. I understand those outputs can be wrong. I exercise professional judgment."
- "I have read and accept the OrchardGuard Terms of Service [linked]."

Captured as a row in `user_acknowledgments` (new table; added as Section 3 addendum below), with the ToS version acknowledged, timestamp, IP address, and the exact text of the acknowledgments at that version.

**At ToS material change (re-acknowledgment):**

When the ToS changes materially, every user must re-acknowledge before their next write operation. The block surface shows: "The Terms of Service have changed. Review and re-acknowledge to continue." Read-only access remains while unacknowledged; writes are blocked.

**`user_acknowledgments` table (addendum to Section 3):**

| Column | Type | Notes |
|---|---|---|
| id | bigserial PK | |
| user_id | bigint NOT NULL FK → users(id) ON DELETE CASCADE | |
| tos_version | text NOT NULL | Matches the hosted ToS version string. |
| acknowledged_at | timestamptz NOT NULL DEFAULT now() | |
| ip_address | inet NULL | For dispute resolution, not for any other purpose. |
| acknowledgment_text | text NOT NULL | Full text of acknowledgments at that version; preserved even when ToS changes. |

### 7.4 Per-Recommendation Disclosures

Every recommendation surface — the "what should I do next" list, per-trigger advisor output, spray-plan view — displays:

- **A source citation** for the trigger (the model or rule that produced the trigger), showing the published algorithm, OMAFRA reference, and parameters version. Hover or tap reveals full citation.
- **A source citation** for the candidate product data (label URL, `label_ingested_at` timestamp, PCP number, OMAFRA efficacy reference).
- **A standing disclaimer** in each recommendation card: "Decision-support only. Verify against product label and OMAFRA guidance before applying. You are the legal applicator." This text is not collapsible or dismissible.
- **Rule-evaluation summary** showing which rules the product passed, which it warned on, and which were overridden. Tied to `rule_evaluations` (Section 3.8.3).

The citations are not decoration. They are the mechanism by which a grower who disagrees with a recommendation can walk back to the source and make their own call. Without citations, V2 becomes a black box; with citations, it is decision-support with a traceable basis.

### 7.5 Material Change Policy

Material changes to the ToS or to V2's core posture require re-acknowledgment. The operator defines what is material; the definition is stable across versions so users aren't surprised.

**Material changes** (re-acknowledgment required):

- Changes to user eligibility requirements
- Changes to user or operator responsibilities
- Changes to data ownership, use, or retention
- Changes to disclaimers or limitations of liability
- Scope expansion (e.g., adding non-apple crops, adding organic program support, broadening audience beyond invited)
- Shift in liability posture (e.g., if agronomist review is added, if a formal SLA is introduced)

**Non-material changes** (versioned, not re-acknowledged):

- Typos, clarifications, formatting
- Link updates
- Contact information
- Changes to non-core features that don't affect responsibility allocation

Material-change events are logged and visible in the user's account settings: "ToS version history with change summaries."

### 7.6 Model and Catalogue Correctness Disclosures

V2 already commits to citation display (Section 6.2) and catalogue provenance (Section 5.1). Section 7 adds disclosure requirements that tie those commitments to the liability framing:

- **Model correctness posture page.** A public page (linked from the ToS and from every recommendation surface) that explains V2's fixture-gate and scenario-replay-gate approach, lists each model's current `last_reviewed_at` date, and states explicitly which models are advisor-enabled and which are in-codebase-but-not-advisor-enabled.
- **Catalogue freshness display.** Every product's detail page shows `label_ingested_at` and a notice if it's older than 365 days ("This product's label data has not been re-verified in over a year; verify against the current label before relying on rule output.").
- **Known-issue log.** A public log of known inaccuracies, pending fixes, and their expected resolution. Example entry: "Cedar apple rust advisory currently uses a threshold model, not a published algorithm. Scope trimming has designated this a SCOUTING-SURFACE in V2; the advisory is informational only. Scouting is the primary basis for decisions."
- **Source-update notifications.** When OMAFRA publishes a bulletin affecting a product or model, users whose catalogue or model usage is affected receive a catalogue event (same channel as trigger alerts, tagged distinctly).

### 7.7 Override and Violation Audit

Section 3.8.5 defined `rule_overrides` and Section 4.8 defined the override flow. Section 7 states the liability posture: **overrides are logged, not suppressed or discouraged.** The grower has authority to override advisory rules; V2 records the override with the grower's stated reason. Post-season summaries present override patterns to the grower as self-review material, not as regulatory reports.

Hard-rule overrides require contact with the operator and a per-farm parameter override entry with bounded effective window (Section 4.8). Hard overrides are rare, deliberately frictional, and logged with full context.

Post-season retrospective — an in-app report summarizes the season's overrides: rule categories invoked, reasons cited, patterns worth noticing. This is for the grower's use, not the operator's; it supports the grower's own reflection on their decisions.

### 7.8 Data Retention and Export

Ties to Section 3.14 retention policy. Specifically:

- **User-owned records** (applications, scouting, photos, tests, rule_overrides): retained indefinitely during active account; retained for a defined grace period after account termination (default 90 days) for reversibility; purged on explicit user request or at end of grace period.
- **System-generated records** (model_runs, triggers, alerts, rule_evaluations): retained 5 seasons in hot storage per Section 3.14; older partitions archived or dropped per farm preference.
- **Data export.** Users can request a full export of their tenant's data at any time. Export format: JSON + CSVs for tabular data, file-tree for photos. Delivered within 7 days of request. Parallels GDPR / PIPEDA portability norms without claiming compliance with any specific regime.
- **Data deletion.** Users can request deletion of their tenant's data. Deletion is complete within 30 days of request. Historical aggregate analytics derived from the user's data before deletion may remain in anonymized form; identifiable references are purged.

These policies are stated in the ToS (Section 7.2 point 6) and operationalized in admin tools.

### 7.9 Discrepancy Reporting

Section 5.7 introduced catalogue discrepancy reports and Section 6.13 introduced model discrepancy reports. Section 7 unifies them as a single user-facing flow and commits to response expectations:

- **In-app report action** available on every product detail page, every recommendation card, every model output surface. Pre-filled with context (product, block, date, what the system said).
- **Triage commitment.** Reports acknowledged within 7 days. Resolution target: catalogue-data fixes within 7 days of acknowledgment when verifiable; model-correctness investigations within 7 days of acknowledgment with initial finding (full resolution may take longer for complex issues); feature-level concerns routed to the backlog with visibility in the known-issue log.
- **Reporter feedback.** When a report results in a catalogue update, model adjustment, or documentation change, the reporting user is notified.

This is where the invited-grower peer review (Section 6.12) becomes a concrete flow rather than a promise.

### 7.10 Insurance and Incident Response

**Insurance.** For V2 invited-beta, the operator does not carry commercial liability insurance beyond personal/professional umbrella coverage appropriate to the scale. This is acceptable for a limited, known audience of certified professionals under a shared-responsibility posture. Before any audience expansion beyond invited users, commercial liability insurance is reviewed and likely obtained. Tracked in Section 10 as a gating concern for broader launch.

**Incident response.** If V2 produces a recommendation that contributes to a crop-loss or off-label event, the response protocol:

1. Operator captures full context from `advisor_sessions`, `rule_evaluations`, `model_runs`, and catalogue state as of the relevant date.
2. Operator reviews whether the recommendation was defensible per then-current published sources and catalogue state.
3. If defensible: the incident is one the shared-responsibility posture anticipated. Operator notes in the known-issue log and may refine relevant rule or citation.
4. If not defensible: operator acknowledges the defect, provides the affected user with documentation of the state that produced the recommendation, updates the catalogue or model, and reviews whether other users were exposed to the same defect. Notifies affected users if applicable.
5. Either way, the incident is logged and archived; patterns of incidents inform correctness priority.

### 7.11 ToS Drafting and Legal Review Posture

The ToS is drafted by the author informed by this section and the rest of the planning document. **For V2 launch, the ToS is self-drafted with a "legal review planned" posture** — V2 onboards invited growers under the self-drafted ToS, and formal legal review is scheduled but not treated as a launch gate.

This is a deliberate trade-off:

- **What's gained.** Faster path to V2 launch; no dependency on lawyer engagement timeline or cost.
- **What's accepted.** Weaker legal footing than a reviewed ToS. Mitigated by: invited-beta scope limits exposure; certified users as counterparties reduces novelty of the risk; personal/professional umbrella insurance covers the likely incident space.
- **When review happens.** Before any of: audience expansion beyond invited beta; scope expansion to non-apple crops or organic programs; incident that escalates beyond internal resolution; first material ToS change that introduces new commitments.

**What the self-drafted ToS must get right, even without lawyer review:**

- Clear eligibility (certified applicator self-attestation)
- Clear responsibility allocation (Section 7.1)
- Clear limitation-of-liability language
- Clear data ownership and use
- Clear termination and export rights

**What legal review will specifically address when it happens:**

- Enforceability of shared-responsibility framing under Ontario law
- Adequacy of disclaimer and limitation-of-liability language
- Consistency with PIPEDA for data provisions
- Jurisdiction for dispute resolution
- Review of self-attestation approach

Legal review is tracked in Section 10 as a high-priority post-launch task, not a launch gate. A lawyer with Ontario agricultural-tech or software-licensing experience is the target engagement.

### 7.12 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| Formal ToS + per-user acknowledgments (Option C from Section 6 discussion) | ToS is durable legal framing; acknowledgments are where the grower personally affirms commitments. Neither replaces the other. |
| Certification is self-attested, not externally verified | Matches invited-beta audience and shared-responsibility posture. External verification would require Ontario Ministry of Labour integration and is out of V2 scope. |
| Material-change re-acknowledgment required; non-material changes versioned but not re-acknowledged | Balances legal diligence with user burden. Stable definition of "material" prevents surprise re-acknowledgments. |
| Per-recommendation disclaimers are non-dismissible | Honest posture requires persistent disclosure, not one-time dismissal. |
| Every recommendation carries source citations (model, catalogue, rule evaluations) | The mechanism by which a grower can walk back to sources and exercise professional judgment. Without citations, V2 is a black box. |
| `user_acknowledgments` table captures full text of acknowledgments at each ToS version | Dispute-resolution durability. "What did you agree to in 2027" is answered with exact text, not inference. |
| Override and violation log is self-review material, not regulatory reporting | Logging is about the grower reflecting on their own decisions, not about the operator policing. |
| Data retention: user records indefinite with 90-day grace after termination; system records 5 seasons | Preserves user value while managing storage. Export and deletion on request honor data ownership. |
| Anonymized aggregate analytics allowed; ML use requires explicit opt-in consent; identifiable farm data never shared absent opt-in | Aligns with PIPEDA norms; preserves future ML optionality without making it a default. Opt-in is captured separately from ToS acknowledgment and is revocable. |
| Discrepancy reports acknowledged within 7 days; catalogue fixes within 7 days when verifiable; model-correctness initial findings within 7 days | Concrete commitment turns the "peer review" promise from Section 6 into an operational flow. Uniform 7-day windows are memorable and honest. |
| Self-drafted ToS with legal-review-planned posture for V2 launch | Legal review is valuable but treating it as a hard gate on invited-beta adds dependency risk (lawyer engagement, cost, timeline) that is disproportionate to invited-beta scale. Self-drafted ToS with specified post-launch review is the honest middle path. |
| No commercial liability insurance for V2 invited-beta; reviewed before any audience expansion | Proportional to invited-beta scale and shared-responsibility posture. Flagged as gating for broader launch. |
| Incident response protocol defined | Every operational tool of this kind should know in advance what it will do when something goes wrong. Section 7.10 makes that concrete. |
| Model correctness posture page is public and lives at a stable URL | Transparency about what's advisor-enabled, what's not, and why. Linked from ToS and every recommendation surface. |

### 7.13 Open Questions Deferred from This Section

1. **Jurisdiction for dispute resolution.** Likely Ontario; confirm with lawyer. If V2 ever onboards a grower outside Ontario (e.g., Quebec or Eastern US apple regions), revisit.
2. **Certification self-attestation vs. external verification.** If Ontario implements a digital verification mechanism for grower pesticide certificates, V2 can integrate. Not a launch gate.
3. **Commercial liability insurance carrier and coverage level for post-invited-beta.** Specific to post-V2 audience expansion. Revisit when that transition is on the horizon.
4. **Jurisdiction-specific ToS variants.** If V2 onboards users outside Ontario, the ToS may need jurisdiction-specific sections (e.g., US data privacy, Quebec French-language requirements). Out of V2; flagged.
5. **Retention of scouting photos and phenology photos.** Photos are storage-heavy. 5-season retention may need adjustment for photo archives. Implementation detail at storage-tier selection.
6. **Discrepancy report volume capacity.** 7-day acknowledgment commitment assumes manageable volume. If invited-grower count grows or reports spike, the commitment tightens author's capacity. Mitigation: community-triage via trusted growers or contract support, pre-committed.
7. **Incident notification scope.** Section 7.10.4 says "notifies affected users if applicable." Criteria for "affected" — every user who received a similar recommendation in the last N days? Needs a defensible rule before first incident.
8. **Lawyer selection and engagement.** Ontario agricultural-tech-experienced lawyers are a narrow field. Engagement process and budget flagged as pre-launch operational task.
9. **Known-issue log format.** Public markdown, versioned database entries, or GitHub issues? Low urgency; pick at implementation.
10. **Aggregate analytics scope.** "Anonymized aggregate analytics may inform V2 development" — what specifically? Usage patterns (which models get consulted, which rules override most often) are low-risk; application-level data (pesticide use trends across farms) risks de-anonymization at small N. Define concretely before publishing V2 ToS.

---

## 8. Phased Build Plan

This section translates the scope, schema, rules, ingestion, correctness, and liability commitments of Sections 1–7 into a development sequence. It is **guided by the work**, not fixed by a date. Aspirational target: V2 usable for the 2027 Ontario apple season (which effectively means the first invited grower onboarded by March 2027 and at full posture by May 2027). Hard launch dates on a tool that makes spray decisions create pressure to cut exactly the correctness work that matters most; V2 treats the aspiration as a planning signal and the actual launch as dependent on milestone completion.

The 2026 V1 season runs in parallel throughout pre-launch development. V1 continues collecting data; that data becomes input to catalogue V1-migration, to Tier-1 scenario replay, and to success-metric baselines.

### 8.1 Phasing Model

V2 ships as a single release (no user-facing V2.0 / V2.1 split, per earlier decision). Internally, development is sequenced through **five phases**, each with entry and exit criteria. Phases are not calendar quarters; they are work states. A phase completes when its exit criteria are met.

The five phases:

1. **Foundation.** Infrastructure, schema, auth, RLS, CI, and the minimum closed-loop skeleton.
2. **Closed-loop vertical slice.** One trigger type, one advisor surface, one rule category, one catalogue-ingested product class, end-to-end.
3. **Breadth build-out.** All model disposition work (fixtures, scouting-surfaces, scope-trimming), all rule categories, nutrition and weed and amendment domains, catalogue bootstrap.
4. **Correctness and hardening.** Scenario replay on Tier-1 models, parameter-drift detection wired to CI, discrepancy-reporting flows live, ToS and acknowledgments drafted and hosted.
5. **Invited-beta onboarding.** First grower beyond the author, then second, with catalogue completeness gating each invitation (per Section 5's grower-specific bootstrap).

Phases overlap where they safely can (phase 3 can start before phase 2 is 100% complete; phase 4 overlaps phase 3). Exit criteria are hard.

### 8.2 Pre-Launch Budget Rollup

Consolidating estimates from earlier sections:

| Category | Source | Estimate (hours) |
|---|---|---|
| Model correctness — fixtures across surviving models | Section 6.11 | 85–155 |
| Model correctness — scenario replay tooling | Section 6.10 | 40–80 |
| Model correctness — Tier-1 scenario replay reviews | Section 6.10 | 16–32 |
| Model correctness — scouting-surface editorial review | Section 6.11 | ~15 |
| V1 catalogue verification | Section 5.13 | 15–20 |
| Bootstrap catalogue (~100 products, hybrid scope) | Section 5.2 | 60–100 |
| ToS self-drafting + acknowledgment wiring | Section 7 | 10–20 |
| Scraper build (OMAFRA) | Section 5.4 | 30–60 |
| Schema migration tooling + RLS policies | Section 3 | 20–40 |
| Auth + multi-tenant bootstrap | Section 3.2 | 20–40 |
| Rule engine skeleton + context assembly | Section 4.2–4.4 | 40–80 |
| Rule authoring (all 12 categories to spec) | Section 4.6 | 80–160 |
| Trigger layer implementation | Section 4.11 | 40–80 |
| Advisor UI (per-block, recommendations, reasoning, REI panel) | Section 2.3 | 80–160 |
| Spray log / applications / amendments / inventory wiring | Section 2.5 | 40–80 |
| Weather pipeline extension (station integration, leaf wetness, soil temp) | Section 2.7 | 40–80 |
| Alerts pipeline upgrade | Section 2.6 | 20–40 |
| Orchard state rebuild (block-aware, phenology auto-progress, photo prompts) | Section 2.8 | 40–80 |
| Variety + rootstock susceptibility registry population | Section 2.8 | 20–40 |
| Audit/compliance exports + rule-override log | Section 2.12 | 20–40 |
| Testing (unit, integration, scenario) | Section 6.12 | 60–120 |
| Buffer (always required; estimates run long) | | 20% on total |

**Subtotal before buffer:** ~790–1500 hours.
**With 20% buffer:** ~950–1800 hours.

At 20 hours/week sustained effort (realistic part-time pace alongside running an orchard), this translates to **48–90 weeks** of development — roughly 11–21 months. At 40 hours/week full-time pace, ~24–45 weeks — roughly 6–11 months.

This is a planning signal, not a commitment. Actual pace depends on author availability, 2026 V1 season demands, and which estimates land at the low or high end of their range. Section 8.7 calls out where the estimate is most vulnerable to expansion.

### 8.3 Phase 1 — Foundation

**Goal:** Infrastructure exists such that any subsequent feature can be built on top of it.

**Work:**

- Postgres on Railway or equivalent (Section 9 finalizes hosting)
- Migration tooling
- Auth system (users, farm_memberships, farm_invitations, global_roles)
- RLS policies on every farm-scoped table (initially empty; policy enforcement tested)
- CI pipeline with typecheck, unit test, and lint
- Logging, error tracking, and basic observability
- Deployment pipeline to a staging environment
- Minimal schema: farms, users, farm_memberships, orchards, orchard_blocks, block_plantings — enough to represent an orchard
- Minimal UI: create farm, add orchard, add block, add planting, log in as invited user

**Entry criteria:** Technology stack decisions (Section 9) made. Repo initialized.

**Exit criteria:** Author can create a farm, log in as a test user on a second farm, confirm RLS prevents cross-farm data access, deploy to staging.

**Estimate:** ~100–180 hours. Foundation is foundational — rushing it costs more later than slower upfront work.

### 8.4 Phase 2 — Closed-Loop Vertical Slice

**Goal:** One complete end-to-end path from model output to advisor recommendation to spray log to updated state. Proves the architecture works before breadth.

**Chosen slice: Apple scab.** Rationale: V1's scab model is production-grade and well-understood, Mills has published fixtures, scab is the signature use case, and the slice exercises nearly every subsystem (block-aware model, per-block state, stage-gated rules, inventory, history feedback).

**Work:**

- Port apple scab model to V2 with block-awareness
- Implement fixture tests with Mills 1944 + NH ascospore curve fixtures
- Scenario replay tooling (built once; reused for fire blight, codling moth, frost later)
- Run scenario replay on apple scab model against author's 2026 V1 data; produce replay review artifact
- Trigger layer with scab-specific triggers (infection window open/closing)
- Rule engine skeleton with registration, growth stage, seasonal max, PHI, REI, inventory rules implemented enough to evaluate scab candidates
- Catalogue seeded with ~10 scab fungicides, verified per strict gate
- Advisor UI: one-trigger session, candidate list with ranking and reasoning, override flow
- Application log with scab-spray logging; spray updates rotation state and kickback timers on next advisor invocation

**Entry criteria:** Phase 1 exit criteria met.

**Exit criteria:** Author can: open the advisor on an active scab trigger, see a ranked candidate list with source citations, log a spray of the top candidate, see inventory decrement, see next advisor invocation reflect the protected-period update.

**Estimate:** ~250–400 hours. This phase is where unknowns surface; it carries the highest estimate volatility.

### 8.5 Phase 3 — Breadth Build-Out

**Goal:** Extend the closed-loop pattern established in phase 2 to all surviving models, all rule categories, all four product domains, and the full catalogue.

**Work, in rough dependency order:**

1. **Model expansion.** Fire blight, frost, codling moth as remaining Tier-1 models (fixtures only at this phase; replay in phase 4). Surviving Tier-2 and Tier-3 models per Section 2.1 dispositions. SCOUTING-SURFACE content authoring.
2. **Rule engine expansion.** All 12 rule categories implemented per Section 4.6. FRAC/IRAC/HRAC rotation engine. Tank-mix compatibility including jar-test prompt and log. Nutrition rules (hard and advisory). HRAC for herbicides. Young-tree safety.
3. **Nutrition evaluators at maturity.** All five nutrition evaluators (leaf sufficiency, soil sufficiency, stage-driven program, symptom-driven, antagonism). Bitter-pit predictor upgrade. Nutrition catalogue (foliar nutrients + soil amendments).
4. **Weed advisor.** Weather-window evaluator, weed scouting log, herbicide catalogue with HRAC, user-initiated advisor flow.
5. **Catalogue bootstrap.** ~100 products through the strict-gate ingestion pipeline. V1 carry-forward re-verified.
6. **Trigger layer expansion.** All trigger types including coincidence triggers. Multi-trigger advisor sessions (per Section 4.11, V2-core).
7. **Orchard state completion.** Per-block phenology auto-progression. Variety and rootstock susceptibility registries populated. Per-block biofix. Photo-prompt flow.
8. **Weather pipeline expansion.** Station integration (hardware of choice per Section 9). Leaf-wetness and soil-temperature as first-class. Per-block microclimate offsets.
9. **Alerts pipeline upgrade.** Digest option, per-severity preferences, trigger-acknowledgment integration.
10. **Irrigation carry-forward.** V1 irrigation tables ported; water-balance cross-linked to phytophthora model.
11. **Scouting capture extensions.** Pheromone trap records, weed scouting, symptom observations feeding deficiency evaluators.
12. **Record layer completion.** Jar-test log, phenology photos, harvest date per planting, rule-violation audit log.
13. **Audit and compliance.** Spray/amendment/herbicide export. Rule-override log display. Model output archive with parameter-hash capture.

**Entry criteria:** Phase 2 exit criteria met; scenario replay tooling exists and has been exercised on at least one model.

**Exit criteria:** Every feature in Section 2 has a shipped implementation; catalogue bootstrap is at ~100 products with strict-gate published status; the author can run the full advisor flow across all four product domains on their own orchard.

**Estimate:** ~400–750 hours. Largest phase by hour count; highest risk of calendar overrun if estimates expand.

### 8.6 Phase 4 — Correctness and Hardening

**Goal:** Everything built in phases 2 and 3 meets the correctness and posture bars declared in Sections 6 and 7. Tier-1 scenario replay complete, discrepancy flows live, ToS and acknowledgments deployed.

**Work:**

1. **Tier-1 scenario replay reviews.** Fire blight, frost, codling moth (scab already done as part of phase 2). Per-model review artifact stored in repo; registry flags models as advisor-enabled.
2. **Parameter-drift detection wired into CI.** Snapshot fixtures for every model. Hash changes flagged in PRs.
3. **Model citations table reconciliation.** Startup reconciliation tested; last-reviewed dates backfilled.
4. **Catalogue currency mechanisms live.** Scheduled scraper re-runs, PMRA PCP cross-check, annual-review surfacing in admin UI.
5. **Discrepancy reporting flows live.** In-app report actions on product, recommendation, and model surfaces. Triage queue UI for author. Known-issue log published.
6. **Model correctness posture page published.** Public URL; linked from every recommendation surface.
7. **Per-recommendation disclosures verified on every surface.** Non-dismissible disclaimer, source citations, rule-evaluation summary.
8. **ToS drafted, hosted at stable URL, versioned.** Acknowledgment flow wired to user onboarding. `user_acknowledgments` captures full text at version.
9. **Data export and deletion flows built.** Export delivers within 7 days; deletion completes within 30 days.
10. **Incident response protocol documented.** Internal runbook for the response sequence in Section 7.10.
11. **Observability hardening.** Rule-evaluation audit depth verified; model run archival tested; trigger deduplication under load tested.
12. **Performance validation.** Context assembly within 500ms budget across realistic block sizes. `rule_evaluations` partitioning tested under simulated volume.

**Entry criteria:** Phase 3 exit criteria met. Catalogue bootstrap complete at ~100 products.

**Exit criteria:** Every commitment in Sections 6 and 7 operational. Tier-1 scenario replay artifacts exist and are positive. ToS live. Discrepancy flow working on test reports.

**Estimate:** ~150–300 hours.

### 8.7 Phase 5 — Invited-Beta Onboarding

**Goal:** First invited grower beyond the author successfully using V2 through a real spray decision cycle. Subsequent growers onboarded as catalogue coverage permits.

**Work:**

1. **Author runs V2 on own orchard for a full pre-bloom + bloom period.** Catches surfaces not exercised in staging. Bugs fixed; posture validated in real use.
2. **First invited grower selection and onboarding.** Grower whose V1 history (if any) or spray practice the author is already familiar with. Catalogue gap analysis — every product in their practice is in V2 catalogue at Published status, or added via the ingestion pipeline.
3. **Invitation flow exercised.** Acknowledgment captured; farm created; orchard and blocks configured; weather station connected if applicable.
4. **First invited grower runs V2 for their own spray cycle.** Author observes, available for triage, captures bugs and friction points.
5. **Post-cycle debrief.** What worked, what didn't, what needed author hand-holding. Feedback feeds phase 6 (continuous improvement) backlog.
6. **Second invited grower onboarded** once first-grower debrief is actionable.
7. **Ongoing: catalogue coverage per grower.** Each subsequent grower's first-use gate is "every product you use is Published in V2." If not, either the grower waits or the author accelerates ingestion for that grower's product set.

**Entry criteria:** Phase 4 exit criteria met.

**Exit criteria:** Three invited growers have completed at least one full advisor → recommendation → application → history-feedback cycle. Discrepancy reports are manageable within the 7-day commitment. V2 is stable enough that author can take a day off without breakage.

**Estimate:** ~100–200 hours, concentrated on bug fixes, per-grower catalogue work, and support.

### 8.8 The 2026 V1 Season in Parallel

Pre-launch development overlaps V1's 2026 season. V1 is not paused. The 2026 season plays several roles:

- **Success-metric baseline.** Per Section 1.8 criteria 3 and 4, targets are TBD until 2026 baseline is in hand. Baselines are captured at season end.
- **Tier-1 scenario replay input.** The 2026 weather, bloom progression, spray history, and observed outcomes are the replay corpus. Phase 4's replay reviews depend on this data being captured in a replayable form — which requires V1 to store hourly weather, daily model outputs, and spray-log entries with timestamps adequate for reconstruction. V1's existing storage is adequate; no V1 changes required.
- **V1 catalogue as V2 migration input.** V1's ~30 seeded products migrate in at phase 3 and are re-verified.
- **Author's own operational continuity.** Author sprays their own orchard through 2026 using V1. V1 must stay working and trustworthy.

**Implication:** Development hours compete with 2026 V1 season demands. Heavy V1 maintenance during 2026 extends V2 calendar. Plan conservatively during April–August 2026.

### 8.9 Dependency Chains and Critical Path

Some work can parallelize; some cannot. The critical path — the longest chain of dependent work — dictates calendar minimum.

**Critical path nodes:**

1. Foundation (phase 1) → everything else
2. Scab vertical slice (phase 2) → validates architecture; must finish before phase 3 confidently scales
3. Scenario replay tooling (built in phase 2, reused in phase 4)
4. Catalogue bootstrap (phase 3; bounded-duration work but non-parallelizable with replay because both are author-hours)
5. Tier-1 replay reviews (phase 4; requires 2026 V1 data, requires tooling, requires models ported and fixture-passing)
6. ToS drafted + acknowledgment flow (phase 4; small but launch-blocking)
7. Invited-beta first-grower readiness (phase 5; requires all of the above)

**Parallelizable work:**

- Schema, auth, RLS, CI (phase 1) — parallel lanes possible
- Multiple rule categories (phase 3) — parallel after rule engine skeleton
- Catalogue ingestion (phase 3) — concurrent with development but same author's time
- Nutrition and weed advisors (phase 3) — can lag core pesticide advisor
- Correctness hardening (phase 4) — parallel with late phase 3 work

**Implication for calendar planning.** Author-single-threaded work (catalogue ingestion, scenario replay reviews, model correctness) cannot be parallelized by adding more effort to other things. If the author has 20 hours per week, those 20 hours are a hard constraint on total throughput.

### 8.10 Risk Register

Known risks to the plan, with mitigations:

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Catalogue ingestion takes longer than 60–100 hours | Medium | Medium | Bootstrap scope is hybrid (Section 5.2); trim further if ingestion becomes bottleneck. Accept launch with a tighter published set and add on-demand. |
| Scenario replay on one or more Tier-1 models reveals substantial defects | Medium | High | Time-box replay remediation at 20 hours per model; if unresolvable, de-enable the model at advisor level and ship with the remaining. Model can be shipped later post-launch. |
| 2026 V1 season requires heavy author attention | High | Medium | Accept calendar slip. V2 ships when ready, not when aspirational. |
| OMAFRA scraper breaks on site changes | Medium | Low | Scraper is an accelerator, not the source. Manual entry from label is always the fallback. |
| Hardware weather station integration is more complex than expected | Medium | Low | Weather station is additive to V1's working Open-Meteo + Env Canada pipeline. Station integration can slip to post-launch. |
| Model fixture sourcing is hard for specific models | Medium | Medium | Constructed fixtures with explicit tagging (Section 6.3) are the escape valve; preserves regression protection even without published worked examples. |
| Rule engine complexity (four product domains) blows the estimate | Medium | High | Phase 2 scab slice exercises the pattern; scaling is then architectural, not speculative. If it blows the estimate anyway, scope-trim rule categories (defer tank-mix-compatibility advanced rules, defer herbicide rules) to post-launch. |
| ToS self-drafting misses something material | Low | High | Self-draft is risk-accepted at invited-beta scale (Section 7.11). Post-launch legal review will catch issues before they become incidents in practice. |
| Invited grower finds V2 unusable in real use | Medium | High | Phase 5 starts with the author using V2 in real season before inviting anyone. First invited grower is someone the author knows well enough to debug with. |
| Author injury, illness, or life event | Low | High | Out of scope for plan; acknowledged reality of single-operator project. |
| Scope creep from invited growers during phase 5 | Medium | Medium | Known-issue log and feature backlog absorb requests. No in-flight scope expansion without explicit decision. |

### 8.11 What's Deliberately Not in the Plan

To keep the plan honest about what's not being built pre-launch:

- **Mobile application.** API surface is exposed (Section 2.11); the app is post-V2.
- **Automated PDF label extraction.** Manual entry stays authoritative (Section 5.6).
- **Cross-farm analytics or benchmarking.** Data boundaries per Section 7.2; no cross-farm visibility.
- **Substitution-scoring in ranking.** Post-V2 enhancement per Section 4.7.
- **Scenario replay for non-Tier-1 models.** Post-V2 (Section 6.10).
- **Multi-jurisdiction support.** Ontario only (Section 7.13).
- **Public signup and marketing site.** Invited-beta only (Section 1.9).
- **Formal SLA or uptime commitments.** Out of ToS (Section 7.2).
- **Commercial liability insurance.** Personal/professional umbrella for invited-beta (Section 7.10).
- **Lab PDF ingestion for tissue tests.** Manual entry (Section 5.6).
- **Automated phenology classification from photos.** User confirmation only (Section 2.8).

Each of these is a real product opportunity; none is a V2 launch requirement.

### 8.12 Success Criteria for Each Phase's Completion

Phase exit criteria restated as observable conditions, for "is this phase done" decisions:

| Phase | Done when… |
|---|---|
| 1. Foundation | Two test users on two farms prove RLS isolation. CI is green. Staging deploys work. |
| 2. Closed-loop slice | Author completes a scab advisor → spray-log → next-advisor cycle on own orchard via V2. Scab scenario replay review artifact exists. |
| 3. Breadth build-out | Every model, rule, advisor surface, catalogue entry enumerated in Sections 2–5 is shipped. Author runs V2 across all four product domains. |
| 4. Correctness and hardening | All Tier-1 models advisor-enabled. ToS live. Discrepancy reporting live. Model correctness page published. Data export/delete flows working. |
| 5. Invited-beta | Three invited growers have completed a full advisor cycle. Author can take a day off. Discrepancy queue is within 7-day SLA. |

### 8.13 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| V2 ships as a single release with internal phases, not user-facing V2.0 / V2.1 | Earlier decision; phasing is about sequencing development, not cutting releases. |
| Development is guided by the work, aspiration is 2027 season | Hard launch dates pressure exactly the correctness work V2's posture depends on. State the aspiration; don't commit the date. |
| Phase 2 vertical slice is apple scab | Best-understood Tier-1 model with published fixtures; exercises nearly every subsystem; proves architecture before breadth. |
| Scenario replay tooling is built in phase 2 and reused in phase 4 | Amortizes the tooling cost across Tier-1 models; also exercises the tooling on a real case before it's depended on. |
| Catalogue bootstrap runs through phase 3, not phase 5 | Catalogue is on the critical path; deferring it to onboarding blocks phase 5 and hides its true duration. |
| Pre-launch budget 950–1800 hours with 20% buffer | Rolled up from Sections 2–7; volatility acknowledged; Section 8.7 identifies expansion-prone lines. |
| Invited-beta onboarding is per-grower gated by catalogue coverage | Honors Section 5.15 open question #7; honest about what "bootstrap is complete" means for each new grower. |
| 2026 V1 season runs concurrent with development; author attention is the constrained resource | Single-operator reality; plan accepts calendar slip over cutting correctness. |
| Risk register is published as part of the plan, not kept private | Pre-commits to mitigations; shared with Claude Code so future sessions don't rediscover the same risks. |
| Each phase has observable exit criteria | Prevents "we're basically done" ambiguity. A phase is done when its criteria are met, not when it feels done. |

### 8.14 Open Questions Deferred from This Section

1. **Which invited grower is first.** Identified candidates exist in the author's network; formal selection happens at phase 5 entry. Criteria: known personally, certified applicator, willing to serve as early-feedback participant, practice aligns with catalogue bootstrap coverage.
2. **Weather station hardware purchase timing.** Resolved: deferred to 2027. Station integration code ships in V2 phase 3; hardware plugs in when purchased.
3. **Hosting migration from V1's Railway setup to V2 infrastructure.** V1 is on Railway with SQLite. V2 needs Postgres; likely stays on Railway or moves to a Postgres-native platform. Section 9.
4. **Staged vs. big-bang cutover from V1 to V2.** V1 continues through at least the author's 2026 season. Beyond that — does V1 keep running as a fallback, or is V2 the only system? Likely V1 shuts down once V2's author-use cycle (phase 5 step 1) is stable.
5. **Author's commitment pace.** Planning estimates assume 20h/week sustained. If actual availability is lower (e.g., 10h/week during bloom), the calendar expands proportionally. No commitment captured here; this is acknowledged reality.
6. **Phase overlap calibration.** Exit criteria are hard; phase entry can happen before full exit of the prior phase in limited cases. What's safe overlap vs. rushed? Operational judgment during execution.
7. **Automated deployment cadence.** How often pre-launch code lands on staging, and how often staging promotes to eventual production. Probably continuous to staging, milestone-based to production. Defer to Section 9 and Phase 1 implementation.

---

## 9. Technology Decisions

This section commits to a primary stack choice for each technology decision with a one-line alternative noted where another option is defensible. The framing is "recommended with alternatives" — Claude Code executes against the primary unless a stated alternative proves better during implementation.

Choices reflect: V1's existing TypeScript/Next.js familiarity, Postgres + RLS from Section 3, multi-tenant from day one, Canadian hosting preference where practical, and the correctness-and-audit posture from Sections 6 and 7.

### 9.1 Language and Runtime

**Primary: TypeScript on Node.js (Node 22 LTS at time of V2 development).**

Reasoning: V1 is already TypeScript/Next.js. Continuity reduces ramp time, preserves V1 model code for port, and leverages the author's existing familiarity. Typed contracts (context objects, rule evaluations, model outputs) are a Section 4 and Section 6 commitment; TypeScript delivers them natively. Node 22 LTS is widely supported and covers the V2 timeline.

**Alternative: Python.** Strong in the scientific-computing space; some agricultural modeling libraries are Python-native. Rejected because V1 port cost outweighs any library benefit and the author's V1 work is TypeScript.

### 9.2 Web Framework

**Primary: Next.js (continuing from V1, upgraded to latest stable at V2 start).**

Reasoning: V1 uses Next.js; continuity. App Router for V2, not Pages Router — Next.js has fully transitioned and V2 is a rebuild. Server Components for data-heavy surfaces (advisor, dashboard) where latency matters. API routes continue for the mobile-ready API surface (Section 2.11).

**Alternative: Remix or SvelteKit.** Similar capability profile; higher switching cost from V1 without clear benefit for V2's workload.

### 9.3 Database

**Primary: PostgreSQL 16 or later.**

Committed in Section 3.18. Reasons restated: row-level security (Section 3.15), JSONB for model outputs and rule details, concurrent writes from cron and users, partitioning for retention management.

**Alternative: None for V2.** SQLite was fine for V1; not fine for V2's multi-tenant + RLS + concurrency profile. MySQL/MariaDB lack RLS polish comparable to Postgres.

### 9.4 ORM and Query Layer

**Primary: Drizzle ORM.**

Reasoning: Drizzle is TypeScript-native, produces clean SQL (no hidden N+1 queries or unexpected query plans), supports Postgres-specific features (JSONB, partitioning, RLS-compatible session variables) without fighting the ORM. Schema is defined in TypeScript with type safety flowing to queries. Good migration tooling.

**Alternative: Prisma.** Excellent DX and widely adopted. Rejected because Prisma's relational query builder has historically struggled with some Postgres-specific patterns (RLS, partitioning, raw SQL fallback paths). Drizzle's closer-to-SQL approach fits V2's schema sophistication better. If Drizzle's maturity becomes a problem in specific areas, Prisma is a viable fallback; the schema translation is mechanical.

**Secondary alternative: Raw SQL with a typed client (e.g., Postgres.js + zod validation).** Maximum control; higher boilerplate. Keep in mind for very hot query paths where ORM overhead matters.

### 9.5 Hosting and Deployment

**Primary: Railway for application hosting and Postgres.**

Reasoning: V1 is on Railway. Railway supports Postgres with reasonable defaults, volumes for file storage, cron, and private networking. Pricing is predictable at invited-beta scale. Canadian data residency is not guaranteed but Railway's US-East region is operationally close enough for Ontario users; PIPEDA considerations are addressed via the ToS commitments (Section 7.2) rather than geographic hosting.

**Alternative: Fly.io with a managed Postgres provider (Neon or Supabase).** Fly offers more control and better global deployment; costs scale similarly. Would require splitting application and database hosts. Viable if Railway's Postgres limits hit.

**Secondary alternative: Vercel (app) + Neon (Postgres).** Next.js-native on Vercel, but Vercel's background job model (cron, long-running tasks) is weaker than Railway's, and V2 has substantial cron work (weather refresh, model runs, scraper runs, alert evaluation).

**On Canadian data residency.** PIPEDA does not strictly require Canadian hosting; it requires accountable handling. The ToS discloses the hosting region (Section 7.2); users with strict residency needs can decline. Not a launch gate.

### 9.6 Authentication

**Primary: Clerk.**

Reasoning: V2 needs user auth, invitation flow, MFA support, session management, and good Next.js integration without running an auth service. Clerk provides all of these with minimal code, supports invitation-based signup (Section 3.2.4 aligns), and exposes webhooks for the `users` mirror table. Pricing at invited-beta scale is modest.

**Alternative: Auth.js (NextAuth).** Open-source, fully self-hosted. More configuration work; MFA and invitation flows require more custom code. Fits if Clerk's pricing or data-residency posture becomes a blocker.

**Secondary alternative: Supabase Auth.** Strong if the database is already on Supabase; less compelling when Postgres is hosted elsewhere.

**RLS integration.** Whichever auth provider, the app layer sets `current_setting('app.current_farm_id')` per request based on the authenticated user's active farm membership. Auth provider provides identity; application layer handles tenant scoping.

### 9.7 Frontend

**Primary: React (via Next.js App Router) with Tailwind CSS and shadcn/ui.**

Reasoning: V1 is React-based. Tailwind is conventional for rapid Next.js development. shadcn/ui provides accessible, unstyled-by-default components that can be themed without fighting a design system.

**Alternative: Unopinionated — author uses whatever component library, if any, they're most productive with.** The point is to ship fast; not a place to overthink.

**State management.** Server state via React Query (TanStack Query). Client state via React's own useState / useReducer plus Zustand if global client state emerges. No Redux.

**Form handling.** React Hook Form + Zod for validation. Zod schemas double as API contract validators.

### 9.8 API Surface

**Primary: tRPC for internal app-to-app calls; REST (OpenAPI-documented) for the mobile-ready API surface.**

Reasoning: tRPC gives typed calls between Next.js server and client without API contract ceremony. But the mobile API surface (Section 2.11) is a separate concern — mobile clients will not run tRPC naturally. Expose REST endpoints for mobile-relevant operations (log application, get recommendations, acknowledge alert, fetch block state), documented via OpenAPI, tested independently.

**Alternative: REST-everywhere (no tRPC).** Simpler mental model; more boilerplate inside Next.js. Fine if tRPC's adoption overhead isn't worth it.

**Secondary alternative: GraphQL.** Overkill for V2's use cases. Reserved for if mobile client needs grow unusually complex.

### 9.9 File Storage

**Primary: Cloudflare R2 for label PDFs, scouting photos, phenology photos.**

Reasoning: S3-compatible, no egress fees, reasonable pricing, good Next.js integration via AWS SDK. File paths in the database (Section 3.11) reference R2 object keys.

**Alternative: AWS S3.** More widely known; egress costs matter when photos are displayed in the app. R2 has the better economics.

**Secondary alternative: Railway volumes.** Acceptable for label PDFs (small, infrequent access). Insufficient for photos (larger, frequent read, benefits from CDN).

**CDN for photos.** Cloudflare's CDN is automatic with R2. Photos served via signed URLs with short-lived expiry for privacy.

### 9.10 Weather Station Hardware

**Decision: Weather station purchase deferred to 2027. V2 launches using V1's existing Open-Meteo + Environment Canada pipeline.**

Rationale: the author's operational priority through 2026 is running V1 and planning V2; hardware evaluation, purchase, and installation are deferred to 2027 when V2 is closer to launch and the author's attention can be focused on the integration.

**What this means operationally:**

- **V2 launches with V1's weather sources.** Open-Meteo primary, Environment Canada secondary, with leaf-wetness estimated from humidity + precipitation and no direct soil-temperature measurement. This is the V1 posture continued.
- **Station integration code still ships in V2.** The `weather_stations` table (Section 3.6.1), station polling infrastructure, source-priority conflict resolution, and per-block microclimate offsets are all built during phase 3. At launch, the only registered station type is public-API sources; `on_orchard_hardware` is a registered option awaiting real hardware.
- **When station is purchased in 2027, the plug-in is trivial.** Add a `weather_stations` row with `station_type = 'on_orchard_hardware'`, configure credentials in `weather_stations.config`, and the polling infrastructure picks it up. No V2 code change required for the basic integration.

**When the time comes — primary recommendation: Davis Instruments Vantage Pro2 with a WeatherLink Live gateway, plus leaf-wetness and soil-temperature sensors.** Reasoning unchanged: well-established in agricultural monitoring, Canadian distribution, documented WeatherLink API, supports the measurements V2 wants first-class. Cost ~$1,500–2,500 CAD per station fully equipped.

**Alternatives when the time comes.**
- **Ambient Weather WS-5000** (~$500–700 CAD). Consumer-grade; good for air temp, humidity, rainfall; leaf wetness weak or absent. Acceptable compromise if budget matters and scab/fire blight accept estimated leaf wetness.
- **Ecowitt stations** (~$400–800 CAD). Similar capability profile to Ambient.
- **Meter Group ATMOS 41 with ZL6 logger** (~$3,500+ CAD). Research-grade; overkill for V2.
- **DIY (WeeWX + custom sensors).** Cheapest in parts, expensive in time. Not recommended.

**Integration posture (when station is live).** V2 polls the station's local gateway at 15-minute intervals. Local gateway preferred over vendor cloud API for reliability (survives internet outages).

### 9.11 Scraper

**Primary: TypeScript scraper (Playwright or Cheerio-based).**

Reasoning: Same language as the app. Code sharing for types. Playwright handles OMAFRA's Hub if it renders dynamically; Cheerio is lighter if static.

**Alternative: Python with requests + BeautifulSoup.** Familiar in scraping communities. Rejected only to avoid the second language in the codebase for what's ultimately a small module.

**Runtime.** Scheduled via Railway cron (weekly in-season, monthly off-season). Stateless; reads OMAFRA, writes JSON staging files to R2 for admin review before promotion (Section 5.4).

### 9.12 CI / CD

**Primary: GitHub Actions for CI; Railway deploy-on-merge for CD.**

Reasoning: GitHub-hosted code is the assumption. Actions is free at V2 scale, well-integrated, and the fixture gate (Section 6.7) is straightforward to implement as a required check. Railway's git-push-to-deploy is simple; staging and production environments distinguished by branch.

**Alternative: Any of CircleCI, GitLab CI, Bitbucket Pipelines.** Equivalent capability; GitHub Actions wins on integration-with-where-the-code-is.

**Branch strategy.** `main` deploys to staging automatically. Tagged releases deploy to production. Feature branches via PR with required CI checks (typecheck, lint, unit tests, fixture tests per Section 6.7).

### 9.13 Testing Framework

**Primary: Vitest for unit and integration tests; Playwright for end-to-end.**

Reasoning: Vitest is fast, TypeScript-native, compatible with most Jest APIs. Playwright is the modern choice for browser E2E.

**Alternative: Jest.** Older and slower; widely familiar.

**Fixture tests** (Section 6.4) are Vitest tests that load fixture JSON and assert model outputs. Scenario replay (Section 6.10) is a separate harness running against persisted 2026 V1 data; not strictly a "test" in the Vitest sense but uses the same runtime.

### 9.14 Observability

**Primary: Sentry for error tracking; Axiom or Better Stack for structured log aggregation.**

Reasoning: Sentry is the standard for Next.js error tracking. Structured logs are a Section 4.13 commitment (per-rule evaluation duration, rule-override patterns, model run failures). Axiom has a generous free tier and good Next.js integration.

**Alternative: Datadog, New Relic, Honeycomb.** Enterprise-grade; overkill for V2.

**What must be observable.** Every advisor session (duration, candidate count, outcome). Every model run (success, failure, duration). Every trigger fired. Every alert attempted and its status. Every rule override. Queries hitting context-assembly 500ms budget (Section 4.10).

### 9.15 Email and SMS

**Primary: Resend for email; Twilio for SMS.**

Reasoning: V1 uses Resend; continuity. Twilio is the standard for Canadian SMS delivery; costs are modest at invited-beta scale.

**Alternative: Postmark (email).** Strong deliverability reputation; different pricing. Resend works; not a reason to change.

**Opt-in discipline.** SMS requires explicit opt-in per user via alert preferences (Section 3.8.1). Email requires ToS acceptance (Section 7.3).

### 9.16 Secrets Management

**Primary: Railway's built-in environment variable management.**

Reasoning: Simple; matches hosting platform. Secrets rotated via Railway dashboard; access limited to project members.

**Alternative: Doppler or Infisical.** Better for multi-environment or multi-project scenarios; overkill for V2's posture.

**What's a secret.** Weather station API keys, auth provider credentials, Resend/Twilio keys, database connection strings, Clerk secret key, Cloudflare R2 credentials. Never committed to the repo.

### 9.17 Development and Local Tooling

- **Package manager.** pnpm (fast, deterministic, disk-efficient).
- **Node version manager.** fnm or nvm; Node 22 LTS pinned via `.nvmrc`.
- **Formatter.** Prettier with a minimal config; enforced in CI.
- **Linter.** ESLint with TypeScript rules; enforced in CI.
- **Type checking.** `tsc --noEmit` in CI; strict mode on.
- **Local database.** Docker Compose with Postgres 16; `drizzle-kit` for migrations; seed script for a minimal dev dataset (one farm, one orchard, one block, one product).
- **Local secrets.** `.env.local` mirrors Railway variables; committed `.env.example` documents required keys.

### 9.18 Migration from V1 to V2

**Primary: ETL script written in TypeScript, run once per farm onboarding.**

Reasoning: V1 is SQLite on Railway; V2 is Postgres. The ETL reads V1's SQLite file, transforms per Section 3.16, writes to V2's Postgres. Run on Railway as a one-off job, not in CI.

**Dry-run mode.** ETL supports `--dry-run` which produces a report of what would be migrated without writing. Used to validate against each farm's V1 data before committing.

**V1 tail.** After V2 launches, V1 remains available for read-only access for at least one full season as a reference. V1 is retired after the author's first full V2 season is complete (Section 8.14 #4).

### 9.19 Monitoring Uptime

**Primary: Better Stack uptime monitoring with a public status page.**

Reasoning: External uptime monitoring distinct from Sentry. Status page is useful for invited growers to know whether an issue is V2-wide or their own. Better Stack's free tier covers V2's scale.

**Alternative: UptimeRobot or Statuspage.io.** Equivalent; Better Stack has modern tooling and a clean status page.

### 9.20 Photo and PDF Handling Libraries

- **PDF rendering in the admin UI** (Section 5.5 side-by-side view): `react-pdf` or Mozilla's PDF.js. Primary: PDF.js via react-pdf wrapper.
- **Photo optimization** (thumbnail generation for gallery): `sharp` on upload.
- **PDF text extraction** (OCR search assist per Section 5.6): `pdf-parse` for embedded text; `tesseract.js` for OCR where PDFs are scanned. Primary: pdf-parse first, tesseract.js fallback.

### 9.21 Summary Table

Consolidation for quick reference:

| Concern | Primary | Alternative |
|---|---|---|
| Language | TypeScript (Node 22 LTS) | Python |
| Web framework | Next.js App Router | Remix, SvelteKit |
| Database | Postgres 16+ | none for V2 |
| ORM | Drizzle | Prisma, raw SQL |
| Hosting | Railway | Fly.io + Neon, Vercel + Neon |
| Auth | Clerk | Auth.js, Supabase Auth |
| Frontend | React + Tailwind + shadcn/ui | — |
| API surface | tRPC (internal) + REST (mobile) | REST everywhere |
| File storage | Cloudflare R2 | AWS S3, Railway volumes |
| Weather hardware (purchase deferred to 2027) | Davis Vantage Pro2 + WeatherLink Live (when time comes) | Ambient WS-5000, Ecowitt, Meter ATMOS 41 |
| Scraper | TypeScript (Playwright or Cheerio) | Python |
| CI/CD | GitHub Actions + Railway deploy | CircleCI, GitLab CI |
| Testing | Vitest + Playwright | Jest |
| Error tracking | Sentry | Datadog, Honeycomb |
| Logs | Axiom or Better Stack | Datadog |
| Email | Resend | Postmark |
| SMS | Twilio | — |
| Secrets | Railway env vars | Doppler, Infisical |
| Package manager | pnpm | npm, yarn |
| Uptime monitoring | Better Stack | UptimeRobot |

### 9.22 Decisions Captured in This Section

| Decision | Rationale |
|---|---|
| TypeScript + Next.js continuity from V1 | Preserves V1 familiarity; TypeScript delivers the typed contracts V2 needs for rules and models. |
| Postgres 16+ with Drizzle ORM | RLS, JSONB, partitioning are Section 3 commitments; Drizzle fits closer to SQL than Prisma without sacrificing type safety. |
| Railway for hosting V2 continues V1 choice | Pricing predictable at invited-beta; supports Postgres, volumes, cron, private networking. Data residency handled via ToS disclosure, not geographic guarantee. |
| Clerk for auth | Invitation flow, MFA, session management out of the box; saves significant build time. |
| Cloudflare R2 for file storage | No egress fees materially affects photo-heavy workloads; S3-compatible for easy library support. |
| Davis Vantage Pro2 as recommended weather station (for 2027 purchase) | Best match for V2's first-class leaf-wetness and soil-temperature requirements among commercial offerings. Purchase deferred to 2027; V2 launches on V1's public-API weather pipeline. |
| Station integration code ships in V2 but not connected at launch | Keeps 2027 hardware integration a configuration change, not a code change. |
| tRPC internal + REST mobile | Internal DX benefits of tRPC preserved; mobile clients get a language-agnostic REST surface. |
| GitHub Actions for CI with fixture-gate checks | Makes the Section 6.7 correctness gate operational, not aspirational. |
| Sentry + Axiom for observability | Both Section 4.13 and Section 6 require structured error tracking and audit-quality logs. |
| pnpm for package management | Faster CI, less disk, no strong reason not to. |

### 9.23 Open Questions Deferred from This Section

1. **Railway's Postgres size limits.** Railway's managed Postgres has tier-based limits. At invited-beta scale the smallest tier likely suffices; monitor as partitioning and volume grow. If a tier-up is required, budget impact is modest.
2. **Clerk vs. self-hosted auth at scale.** Clerk's per-MAU pricing is favorable at invited-beta but grows if V2 ever broadens. Revisit before audience expansion.
3. **R2 signed-URL TTL and caching strategy.** Short-lived URLs are privacy-preserving but recompute per request. Cache at app-layer with a TTL shorter than the URL's validity. Implementation detail.
4. **Playwright vs. Cheerio for the scraper.** Depends on OMAFRA Hub's rendering behavior. Start with Cheerio; upgrade to Playwright if the Hub requires JS rendering.
5. **Mobile app framework choice when that project starts.** Not a V2 decision. React Native (matches web) or native (Swift/Kotlin) is the likely fork.
6. **Weather station installation and maintenance responsibility.** Author installs on own orchard; invited growers install their own. V2 provides guidance but not hardware support. Flag for invited-beta onboarding docs.
7. **Database backup strategy.** Railway's managed Postgres includes backups but not point-in-time recovery at all tiers. If PITR matters for V2's audit posture, upgrade tier or add a secondary backup (pg_dump to R2 daily). Section 10.
8. **Which specific Postgres partitioning tool.** Native declarative partitioning is in Postgres 11+. pg_partman automates maintenance. Start with native; add pg_partman if retention management becomes manual toil.
9. **Automated testing of RLS policies.** Verify that RLS actually prevents the queries it's supposed to prevent. Requires a test harness that sets `app.current_farm_id` to various values and asserts access. Phase 1 work; flagging here.
10. **Editorial tooling for SCOUTING-SURFACE content.** Static markdown vs. headless CMS. Likely markdown for V2 (Section 2.13). Revisit if content editing becomes a bottleneck.

---

## 10. Consolidated Open Questions and Post-V2 Work

This section is a registry. It pulls every open question from Sections 1–9 into one place, categorizes by when it needs resolution, and adds post-V2 enhancements that sections deferred explicitly. Claude Code and future author sessions should start here when asking "what still needs deciding."

Questions are tagged by urgency:

- **🔴 Launch-blocking** — must resolve before first invited-beta grower is onboarded.
- **🟡 Phase-gated** — must resolve at a specific phase boundary (blocks progress past that phase, not launch).
- **🟢 Post-launch** — can wait until after first invited-beta season produces signal.
- **🔵 Speculative / research** — open-ended, no fixed resolution; revisit as evidence accumulates.
- **✅ Resolved** — listed here for traceability; no action needed.

### 10.1 Launch-Blocking (🔴)

Must have an answer before invited-beta onboarding begins (phase 5).

| # | Question | Source | Resolution path |
|---|---|---|---|
| L1 | First invited grower selection | 8.14 #1 | Phase 5 entry decision. Criteria: known personally, certified, willing to serve as early feedback, practice aligns with catalogue coverage. |
| L2 | Hosting migration plan from V1 to V2 | 8.14 #3 | Phase 4 exit: decide whether V1 stays online as fallback read-only for author's first V2 season. |
| L3 | ToS self-draft final language | 7.13 | Phase 4 work; must be hosted at stable URL before any grower onboarding. |
| L4 | Jurisdiction for dispute resolution in ToS | 7.13 #1 | Likely Ontario; lock in ToS. |
| L5 | Known-issue log publication format | 7.13 #9 | Markdown in-repo with public mirror is simplest. Pick during phase 4. |
| L6 | Catalogue ingestion bootstrap completeness for first grower | 5.15 #7 | Resolved at grower-specific level: every product in the first grower's practice is Published in V2 before they onboard. Flow operationalized in phase 5. |
| L7 | `nutrition_runs.input_test_id` polymorphism pattern | 3.19 #1 | Implementation decision during phase 1 schema build. Two nullable FKs (`input_leaf_test_id` + `input_soil_test_id`) likely cleaner than polymorphic pair. |
| L8 | Late-season nitrogen cutoff default | 4.16 #5 / 6.15 #7 | Research OMAFRA, Ontario Apple Growers' association, local extension guidance. Mid-August is starting point; confirm before hard rule ships. |
| L9 | Photo storage backend | 3.19 #3 | Resolved to Cloudflare R2 in Section 9.9. No action. |
| L10 | Demo farm for onboarding | 2.16 #3 | Resolved: cut. ✅ |
| L11 | Printable REI signage for block gates | 2.16 #4 | Resolved: out of scope for V2. ✅ |
| L12 | Photo prompt default cadence | 2.16 #5 | Resolved: user picks on enable. ✅ |
| L13 | Fixture licensing review | 6.15 #1 | Self-draft ToS posture (Section 7.11) softens the urgency. Fixtures cite sources; do not reproduce proprietary tables verbatim. Revisit if an issue arises. |
| L14 | `catalogue_requests` triage workflow for grower product requests | 5.15 #6 | Process defined in Section 5.9; resolution detail is "author triages within 7 days." Confirm capacity holds before each new grower onboards. |
| L15 | Legal review of ToS | 7.13 | Resolved: self-drafted with legal-review-planned posture (Section 7.11). Legal review scheduled post-launch, not gating. ✅ |

### 10.2 Phase-Gated (🟡)

Must resolve at a specific phase boundary. Does not block launch; blocks progress past the phase.

**Phase 1 — Foundation**

| # | Question | Source |
|---|---|---|
| P1.1 | Automated testing of RLS policies | 9.23 #9 — set up the test harness during phase 1 schema build |
| P1.2 | Local development environment | 9.17 — pnpm, Docker Compose Postgres, seed script |
| P1.3 | Database backup strategy (including PITR) | 9.23 #7 — decide at phase 1; cheaper to set up early |
| P1.4 | Which specific Postgres partitioning tool (native vs pg_partman) | 9.23 #8 — start native; add pg_partman only if manual toil emerges |

**Phase 2 — Scab Vertical Slice**

| # | Question | Source |
|---|---|---|
| P2.1 | Scenario-replay scenario selection methodology | 6.15 #9 — formalize before first replay |
| P2.2 | Replay review artifact format standardization | 6.15 #10 — template authored before first replay |
| P2.3 | Snapshot fixture sensitivity calibration | 6.15 #2 — tune during first Tier-1 model ship |

**Phase 3 — Breadth Build-Out**

| # | Question | Source |
|---|---|---|
| P3.1 | Nutrition model output shape specifics | 4.16 / 6.15 #11 — finalize at nutrition evaluator implementation |
| P3.2 | Fixture sourcing for Gubler-Thomas upgrade (powdery mildew) | 6.15 #8 — research before powdery mildew implementation |
| P3.3 | Weed species source-of-truth registry | 2.16 #6 — research before weed species registry populated |
| P3.4 | HRAC group data source | 2.16 #7 — ingestion research before herbicide catalogue bootstrap |
| P3.5 | Young-tree age threshold authority per product | 2.16 #8 — captured during catalogue ingestion per product |
| P3.6 | Jar-test protocol authority | 2.16 #9 — pick one OMAFRA or extension protocol before shipping jar-test flow |
| P3.7 | Variety and rootstock susceptibility source | 2.16 #2 / 5.15 #2 — research before registries populated |
| P3.8 | `rule_evaluations` volume under real load | 3.19 #2 — revisit partitioning and retention if advisor run frequency scales up |
| P3.9 | `weather_daily` materialized view vs. refreshed table | 3.19 #6 — benchmark during phase 3 |
| P3.10 | Coincidence trigger rules authoring | 4.16 #6 / 2.16 #10 — author before coincidence triggers ship |
| P3.11 | Playwright vs Cheerio for scraper | 9.23 #4 — start with Cheerio; upgrade based on OMAFRA Hub behavior |
| P3.12 | Editorial tooling for SCOUTING-SURFACE content | 9.23 #10 — markdown is default; revisit if content editing bottlenecks |
| P3.13 | Catalogue request volume capacity | 5.15 #6 — monitor during phase 5 grower onboarding |
| P3.14 | Tank mix groups when one component is a soil amendment | 3.19 #7 — enforce via application layer; decision finalized during phase 3 rule engine work |

**Phase 4 — Correctness and Hardening**

| # | Question | Source |
|---|---|---|
| P4.1 | Model deprecation fallback behavior | 6.15 #4 — registry-flag-driven skip with visible note |
| P4.2 | OMAFRA outreach timing about scraping | 5.15 #3 — after phase 4; before phase 5 |
| P4.3 | Incident notification scope criteria | 7.13 #7 — rule written before first incident possible |
| P4.4 | Retention policy for photos specifically | 7.13 #5 — tune at storage tier review |
| P4.5 | Aggregate analytics scope concrete definition | 7.13 #10 — define before V2 ToS is published |

**Phase 5 — Invited-Beta Onboarding**

| # | Question | Source |
|---|---|---|
| P5.1 | Author's actual week-over-week commitment pace | 8.14 #5 — calibrates calendar during execution |
| P5.2 | Phase overlap calibration | 8.14 #6 — operational judgment |
| P5.3 | Weather station installation and maintenance responsibility | 9.23 #6 — invited-beta onboarding docs |

### 10.3 Post-Launch (🟢)

Can wait until after first invited-beta season. Revisit when signal from real use accumulates.

| # | Question | Source |
|---|---|---|
| PL1 | Success-metric baselines (criteria 3 and 4 from Section 1.8) | 1.11 #3 — revisit Q4 2026 with V1 baseline data |
| PL2 | Per-farm rule-enable overrides | 4.16 #1 — revisit if growers need to suppress specific rules |
| PL3 | Rule precedence when rules conflict | 4.16 #3 — revisit if UI surfaces bad conflicts |
| PL4 | Rate suggestion within recommendations | 4.16 #4 — post-V2 enhancement; depends on real usage signal |
| PL5 | Multi-trigger merge window default | 4.16 #7 — confirm with 2026 V1 retrospective or early V2 data |
| PL6 | Substitution-ranking panel (cost-vs-efficacy) | 4.7 — post-V2 enhancement; requires finer efficacy data than OMAFRA's 1–5 scale |
| PL7 | Scenario replay extension to non-Tier-1 models | 6.10 — natural follow-up once replay tooling exists |
| PL8 | Parameters version retrospective (re-run 2026 with 2027 parameters) | 6.15 #5 — useful but non-trivial; post-V2 |
| PL9 | Grower-reported disagreement triage workflow | 6.15 #3 — calibrate after real volume is known |
| PL10 | Public-audience expansion trigger (criteria for moving beyond invited-beta) | 1.11 #4 — revisit after first V2 season |
| PL11 | Agronomist partnership trigger | 6.12 — appropriate when broadening beyond certified users |
| PL12 | Commercial liability insurance carrier and coverage | 7.13 #3 — before any audience expansion |
| PL13 | Jurisdiction-specific ToS variants | 7.13 #4 — if V2 ever onboards users outside Ontario |
| PL14 | Certification external verification vs self-attestation | 7.13 #2 — if Ontario implements a digital verification mechanism |
| PL15 | Clerk vs self-hosted auth at scale | 9.23 #2 — revisit before audience expansion |
| PL16 | Railway Postgres tier | 9.23 #1 — monitor and upgrade as partitioning and volume grow |
| PL17 | Mobile app framework choice | 9.23 #5 — decided when the mobile project starts, not V2 work |
| PL18 | Bulk efficacy re-rating propagation | 5.15 #4 — handle when OMAFRA republishes with methodology changes |
| PL19 | Multi-product tank-mix recommendations directly (as opposed to jar-test flow) | 4.16 #2 — depends on accumulated per-farm jar-test data |
| PL20 | Orchardguard feature request: weather-station purchase and first integration | 9.22 — 2027 target |

### 10.4 Speculative / Research (🔵)

Open-ended. No fixed resolution path; revisit as evidence accumulates or conditions change.

| # | Question | Source |
|---|---|---|
| S1 | Multi-jurisdiction scope (non-Ontario apple regions) | 5.15 #5 / 7.13 |
| S2 | Organic production programs | 1.7 / 1.10 — excluded from V2; reconsider if strong demand |
| S3 | Non-apple crops (pears, peaches, cherries) | 1.3 / 1.7 — excluded from V2; similar models, different labels |
| S4 | Lab PDF ingestion for tissue tests | 1.7 / 5.6 — nice-to-have; complex and fragile |
| S5 | Automated ML-based phenology classification from photos | 1.7 — user confirmation is V2's posture |
| S6 | Cross-farm analytics or benchmarking | 1.7 / 7.2 — forbidden in V2 without opt-in; revisit with care |
| S7 | Tier-4 demoted content accuracy mechanism | 6.15 #6 — lightweight review checklist in PR template |
| S8 | Whether V2 eventually supplies farm-level aggregate dashboards with cross-block comparison | 2.3 — useful but not part of closed-loop thesis |
| S9 | Invited-grower peer triage as formal support mechanism | 6.15 #3 / 7.13 #6 — long-term mitigation for author capacity |
| S10 | Structured knowledge-base for spray program templates ("a Honeycrisp scab program") | 2.3 — expansion beyond V2's per-trigger advisor |

### 10.5 Notes on Living Document Maintenance

This document is the source of truth for V2 scope, architecture, and posture. Three maintenance practices:

- **Every decision is versioned via git commit.** When a decision is revised, the prior version is visible in history.
- **Open questions are only resolved by moving them to the Resolved column or removing them — never by silent deletion.** Traceability requires that resolutions are documented.
- **New questions that emerge during implementation get added to Section 10, not to the section where the topic lives.** Section 10 is the registry; source sections remain stable.

Future author or Claude Code sessions should start at Section 10 when asking "where are we." The roadmap is the open-question list. The plan is what's above it.

---

## Executive Summary

*This summary is at the end of the document so that re-reading the full plan before it is the default path. When re-orienting quickly in a future session, start here, then jump to Section 10 for the live work list.*

**Thesis.** V2 closes the loop V1 left open. V1 has the parts of a decision-support system (prediction, catalogue, spray log, inventory, weather); they do not talk at recommendation time. V2 connects them into a working cycle.

**Audience.** Certified Ontario apple growers, conventional and IPM, invited-beta only. Not public SaaS. Not organic. Not non-apple. Not unverified users.

**Core loop.** Prediction (weather + per-block state + nutrition + observations) → Trigger (state transitions fire events) → Advisor (candidates + context → ranked list) → Rules (four domains, hard and advisory, twelve categories) → Records (applications feed back into the next recommendation).

**Block-awareness is non-negotiable.** V1's models run orchard-wide; V2's run per-block. Every model, every rule, every recommendation reasons about a specific block's state, history, and plantings.

**Nutrition is inside the loop.** Same five stages as disease/pest. Stage-driven, test-driven, or symptom-driven triggers. Foliar nutrients treated like sprays; amendments as a parallel product class. Hard and advisory rules.

**Weeds are inside the loop.** Herbicide catalogue with HRAC groups, user-initiated weather-aware advisor, scouting-driven.

**Model scope trimmed from V1's 55 to ~18–20.** Tier-2 threshold models that were estimates-dressed-as-models become SCOUTING-SURFACEs — weather-aware advisories without risk scores. Tier-1 models (scab, fire blight, codling moth, frost) rebuild with fixture + scenario-replay gates.

**Correctness gate has two layers.** Published fixtures required on every surviving model (mandatory for Tier 1; published-or-constructed for Tier 2/3). Scenario replay against 2026 V1 data required for Tier 1 before advisor-enabling.

**Catalogue has a strict quality gate.** Only products manually verified against their label PDF are recommendation-eligible. OMAFRA scraping is an accelerator; the label is authoritative. Bootstrap scope is ~100–150 products covering V1's seed catalogue and OMAFRA-recommended programs.

**Liability posture is shared-responsibility.** Formal ToS plus per-user acknowledgments at each material change. Every recommendation cites sources and disclaims. Hard-rule overrides are frictional; advisory overrides are logged with reason. Self-drafted ToS with legal-review-planned posture for launch.

**Stack:** TypeScript + Next.js + Postgres + Drizzle + Railway + Clerk + Cloudflare R2. Weather station (Davis Vantage Pro2 recommended) deferred to 2027; V2 launches on V1's Open-Meteo + Env Canada pipeline.

**Build plan:** Five phases — Foundation → Scab Vertical Slice → Breadth Build-Out → Correctness and Hardening → Invited-Beta. Pre-launch budget 950–1800 hours with 20% buffer. Guided by the work, not a fixed date. Aspirational target: 2027 Ontario apple season. Single-operator reality; calendar slip preferred over cutting correctness.

**2026 V1 continues in parallel.** The 2026 season gathers baseline data for success metrics and scenario-replay corpus. V1 is not paused; author attention is the constrained resource.

**What is NOT in V2 at launch:** public signup, mobile app, organic programs, non-apple crops, automated PDF extraction, cross-farm analytics, lab PDF ingestion, on-orchard weather hardware, commercial liability insurance, scenario replay on non-Tier-1 models, substitution-scoring ranking.

**Where the project stands as of this document:** V2 is not built. The plan is complete. V1 is running in production for the 2026 season. Next actions: commit this document to the V2 repo; begin V2 phase 1 work (Foundation) in parallel with V1's 2026 season; watch for 2026 season data that affects open questions (especially baselines and replay corpus).

---

> **End of planning document.** Sections 1 through 10 complete. This is a living document; revisions via git.

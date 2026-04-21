# OrchardGuard V2 — Glossary

> **Purpose.** Speed anyone (including Claude Code in a fresh session) into V2's domain language. This document mixes pomology, pesticide regulation, phytopathology, and V2-specific terminology. When a term means something specific in V2's codebase that's narrower or different from the general term, that's flagged.
> **Scope.** Ontario apple production, conventional and IPM. Organic-specific terms are omitted per V2's scope (Section 1.7 of the planning doc).

---

## A

**Active ingredient (a.i.).** The chemical in a pesticide that does the work. A product ("Captan 80 WDG") contains an active ingredient ("captan"). Rules that govern dosing and seasonal maxima often apply at the active-ingredient level, not the product level — two different products containing the same a.i. can share a seasonal max.

**Advisor (V2).** The subsystem that takes a trigger + block context + inventory and produces a filtered, ranked list of candidate products with reasoning. Section 2.3 and Section 4.11 in the plan. Not the same as "the app" — the advisor is one layer inside it.

**Advisory rule (V2).** A rule whose outcome is a warning the grower can override with a recorded reason. Contrast with **hard rule**. Section 4.5.

**Advisor-enabled (V2).** A Tier-1 model that has passed both fixture tests and scenario replay review, and is permitted to drive recommendations. A model can exist in the codebase without being advisor-enabled. Section 6.7.

**Ambient Weather.** Consumer-grade weather station manufacturer. Alternative to Davis. Section 9.10.

**Antagonism (nutrition).** Two nutrients in the plant's soil or tissue interfering with each other's uptake. Classic pairs: potassium–magnesium, calcium–boron, calcium–potassium, phosphorus–zinc. V2's nutrition evaluator flags antagonism.

**Application (V2).** One instance of product going on a block. Pesticides, herbicides, foliar nutrients, and soil amendments all log to the unified `applications` table. Section 3.5.1.

**Apple scab.** *Venturia inaequalis.* The signature apple disease in Ontario and most temperate apple regions. Infection periods driven by wet leaf surfaces plus temperature; V1 and V2 use the Mills Table plus the NH ascospore maturity curve.

**Ascospore.** Sexual spore of *V. inaequalis* (apple scab). Released from overwintered leaf litter during primary scab season. Maturity is modeled via a logistic curve on degree-day accumulation from green-tip.

**Author (V2).** The person maintaining the project and the catalogue. In V2, also a formal role in the RBAC model (`global_roles`). The author has permissions to curate reference data, manage catalogue ingestion, and triage discrepancy reports.

**Axiom.** Log aggregation service. V2 sends structured logs here for searchable history. Section 9.14.

## B

**Biofix.** The date at which degree-day accumulation for a pest's development begins. For codling moth, biofix is typically the date of sustained moth catches in pheromone traps. Each pest that uses degree-day timing has its own biofix; V2 supports per-block biofix per pest. Section 3.3.6.

**Bitter pit.** Calcium-related storage disorder in apples, especially Honeycrisp and Gala. Shows as brown lesions in the flesh, often post-harvest. Driven by variety susceptibility, vigor, crop load, and leaf calcium at key periods. V2 treats bitter pit as the archetype of nutrition-inside-the-loop.

**Block (V2).** A geographic subdivision of an orchard, usually with similar planting history, variety mix, and management. Sections 3.3.2 and 3.3.3. V2 is block-aware throughout — applications, models, triggers, advisor, and rules all scope to a block.

**Block-aware (V2).** A model, rule, or advisor surface that reads per-block state rather than orchard-wide state. V1's models are orchard-wide; V2's are block-aware.

**Bloom.** The flowering stage of apple development. Full bloom is when ~80% of the king blossoms are open. Bloom is the most fire-blight-vulnerable stage.

**Bloom stage.** One of a sequence of phenological stages from dormant through fruit-set. V2 uses an 8-stage enum carried forward from V1. Examples: dormant, silver-tip, green-tip, half-inch green, tight cluster, pink, bloom, petal-fall, fruit-set.

**Bootstrap catalogue (V2).** The initial set of ~100–150 products populated before V2 invites any grower. Defined by scope rule in Section 5.2, not as a fixed list.

## C

**Captan.** Broad-spectrum protectant fungicide, FRAC M4. Workhorse for apple scab programs.

**Catalogue (V2).** The database of products (pesticides, herbicides, foliar nutrients, soil amendments) with label-accurate data. Section 3.4. Source of truth for advisor recommendations.

**Cedar apple rust.** *Gymnosporangium juniperi-virginianae.* Fungal disease with a two-host lifecycle requiring both apple and juniper. Spores release during wet weather at bloom. V2 treats as a SCOUTING-SURFACE (no formal risk score).

**Cheerio.** Node.js library for parsing HTML. V2's OMAFRA scraper uses it for lightweight scraping. Section 9.11.

**Clerk.** Authentication-as-a-service. V2 uses it for user signup, invitation flow, MFA, session management. Section 9.6.

**Closed loop (V2).** V2's core architectural commitment: prediction → trigger → advisor → rules → records, each feeding the next. Section 1.5.

**Codling moth.** *Cydia pomonella.* The primary internal-feeding pest of apples globally. Larvae tunnel into the fruit. Modeled via degree-day accumulation from biofix base 10°C.

**Coincidence trigger (V2).** A trigger that fires when multiple underlying model states align on the same block within a defined window. Example: a scab infection window and a fire blight high-risk event overlapping. Section 2.2.

**Cougarblight.** Washington State University fire blight risk model (v5.1). Based on accumulated degree-hours in a 4-day rolling window, with inoculum-level adjustments. Part of V1 and V2.

**Conventional (production system).** Chemical pesticides and synthetic fertilizers permitted. V2 is conventional + IPM. Contrast with organic.

**Cultivar.** Synonym for variety. See also: Honeycrisp, McIntosh, Gala, Ambrosia.

## D

**Davis Instruments.** Weather station manufacturer. Vantage Pro2 is the recommended V2 station hardware (purchase deferred to 2027 per Section 9.10).

**Degree-day (DD).** A heat accumulation unit used to predict biological development. Computed per day as (max_temp + min_temp)/2 − base_temp, with several variations (sine method, etc.). Pests and diseases with temperature-driven development use DD thresholds. Example: codling moth 1st generation egg hatch peak is ~250 DD from biofix, base 10°C.

**Degree-hour (DH).** Similar concept to DD but at hourly resolution. CougarBlight fire blight model uses DH base 15.5°C with specific temperature caps.

**Disposition (V2).** A tag on every V1 artifact recording how it carries forward to V2: NEW, REBUILD, KEEP-AS-IS, UPGRADE, SCOUTING-SURFACE, CUT, or DEMOTE. Section 2 of the planning doc.

**Drizzle.** TypeScript ORM used in V2. Section 9.4.

## E

**Efficacy rating.** A score of how effective a product is against a target. OMAFRA publishes 1–5 star ratings; V2 stores these in `product_efficacy`. Efficacy drives recommendation ranking but is not a hard rule.

**Environment Canada.** Canadian government weather service. V2 uses their station observations as a secondary weather source (primary is Open-Meteo, tertiary when V2 has one would be an on-orchard station). Section 2.7.

**ETL.** Extract-Transform-Load. V2's V1→V2 data migration is an ETL script. Section 9.18.

## F

**Farm (V2).** The top-level tenant entity. A farm may contain one or more orchards. Multi-farm isolation is the V2 starting point. Section 3.2.1.

**Farmer Pesticide License.** Ontario credential allowing a farm operator to purchase and apply Schedule 2 pesticides on their own farm. V2's invited-beta audience holds either this or the Grower Pesticide Safety Certificate.

**Fire blight.** *Erwinia amylovora.* Bacterial disease fatal to young trees on susceptible rootstocks. Vectored by bees during bloom and by rain-spread afterward. Risk models: CougarBlight and MaryBlyt.

**Fixture test (V2).** A test case for a model drawn from a published source (or explicitly constructed where no published example exists). Running the model on the fixture's inputs must produce outputs matching the fixture's expected values within tolerance. Section 6.4.

**FRAC.** Fungicide Resistance Action Committee. Numbers fungicides by mode of action. FRAC group 11 = QoIs (strobilurins). FRAC M4 = multi-site (e.g. captan). Resistance management requires rotating across FRAC groups.

**Frost risk.** V2 model. Temperature-threshold-based kill probability per bloom stage. Green-tip tolerates ~-6.7°C; open bloom kills at ~-2.2°C (10% kill).

**Fruit-set.** Bloom stage after petal fall when fruitlets are visible. Last stage in V2's 8-stage enum.

## G

**Gala.** Apple variety. Mid-season harvest. Moderately susceptible to bitter pit; bitter-rot-prone.

**GitHub Actions.** CI/CD platform. V2 runs typecheck, linting, unit tests, and fixture tests on every PR. Section 9.12.

**Grower Pesticide Safety Certificate.** Ontario credential for pesticide applicators. One of two certifications that qualify a user for V2's invited beta. Self-attested, per Section 7.3.

**Gubler-Thomas.** Powdery mildew risk model from UC Davis. V2 upgrades V1's threshold-based mildew model to Gubler-Thomas.

## H

**Hard rule (V2).** A rule that blocks a recommendation and cannot be casually overridden by the grower. Examples: PHI (pre-harvest interval), REI (re-entry interval), label-prohibited growth stage, seasonal maximum per label, late-season nitrogen cutoff. Contrast with **advisory rule**. Section 4.5.

**HoneyCrisp.** Apple variety. Mid-late season. Highly susceptible to bitter pit. Vigor management and calcium management matter.

**HRAC.** Herbicide Resistance Action Committee. Numbers herbicides by mode of action. Parallel to FRAC for fungicides and IRAC for insecticides.

## I

**Inventory lot (V2).** One specific batch of product in the farm's physical stock, identified by lot number or purchase date. Section 3.5.2. V2 tracks inventory at lot level for FIFO deduction and lot-specific expiry.

**IPM.** Integrated Pest Management. A production posture that uses monitoring (scouting, traps), thresholds, and targeted intervention rather than calendar-based spraying. V2 supports IPM alongside conventional.

**IRAC.** Insecticide Resistance Action Committee. Numbers insecticides by mode of action.

## J

**Jar test.** Benchtop test of tank-mix compatibility: combine the products at the intended rates in a small volume of water and observe for precipitation, phase separation, or other incompatibility signs. V2 surfaces a jar-test prompt when proposed mixes have unknown compatibility in the catalogue. Section 3.5.5.

## K

**Kaolin clay.** Mineral clay sold as a foliar barrier (Surround WP is the common brand). Reduces sunburn, suppresses some insect feeding. Section 2.1.1.

**Kickback activity (fungicide).** Ability of a fungicide to suppress an infection that has already begun. Expressed as hours from start of wetness during which the fungicide retains curative effect. Different scab fungicides have different kickback windows.

## L

**Label.** The legal document that accompanies a pesticide and defines its permitted uses, rates, target pests, PHI, REI, seasonal maxima, tank mix restrictions, etc. V2 treats the label as the authoritative source for catalogue data. Section 5.1.

**Leaf wetness.** The duration of moisture on leaf surfaces. Critical input to scab and fire blight models. Directly measurable with a leaf-wetness sensor; estimated from humidity + precipitation otherwise. V2 supports both when station hardware is available (deferred to 2027).

## M

**Mancozeb.** Broad-spectrum protectant fungicide, FRAC M3. Workhorse for apple disease programs.

**MaryBlyt.** Fire blight risk model (v7.1). Uses a 4-condition gate (open blossoms, temperature met, wetting event, degree-hours accumulated) to fire an "epiphytic infection point" trigger. Combined with CougarBlight in V2.

**McIntosh.** Apple variety. Ontario classic. Highly scab-susceptible; moderate fire blight susceptibility.

**Merivon.** Combo fungicide, FRAC 7 + 11 (fluxapyroxad + pyraclostrobin). Used in scab and summer disease programs.

**Microclimate.** Per-block or sub-orchard variation in temperature, humidity, or wind from the orchard average. V2 supports microclimate overrides per block. Section 3.3.4.

**Mills (Mills Table).** W.D. Mills, 1944. Classic table of scab infection periods: duration of leaf wetness required at a given mean temperature to produce light, moderate, or severe infection. V1 and V2 implement the Modified Mills Table.

**M.9.** Apple rootstock. Dwarfing; widely planted. Highly fire-blight-susceptible — this susceptibility is one of the reasons V2 tracks rootstock per planting.

**Mode of action.** How a pesticide kills the target at the biochemical level. Products sharing a mode of action select for the same resistance mechanisms; FRAC/IRAC/HRAC numbers group products by mode.

**Modifier (rule outcome).** A rule's produced evaluation has one of four outcomes: `pass`, `warn`, `block`, `not_applicable`, plus a severity modifier (`hard`, `advisory`, `informational`). Section 4.4.

## N

**Nectria canker.** Fungal canker disease, *Neonectria galligena.* Infection through wounds during cool wet periods in fall and spring.

**Next.js.** React-based web framework. V2 continues V1's use of it. Section 9.2.

**Nutrition (V2).** Fertility management as a closed-loop participant in V2. Covers foliar nutrients (sprays containing nutrients) and soil amendments (lime, gypsum, granular fertilizers). Section 2.1.3.

## O

**OMAFRA.** Ontario Ministry of Agriculture, Food and Rural Affairs. Publishes Crop Protection Hub and Publication 360 (fruit production recommendations). V2's primary guidance source after the product label.

**OMAFRA Crop Protection Hub.** Ontario's online resource for pesticide recommendations. V2 scrapes it as an ingestion accelerator. Section 5.4.

**OMAFRA Publication 360.** "Fruit Production Recommendations." Annual compendium of variety, rootstock, disease, pest, and nutrition guidance for Ontario tree fruit. V2's primary nutrition reference.

**Open-Meteo.** Free weather API. V2 continues V1's use of it as primary public weather source. Section 2.7.

**Orchard (V2).** A geographic property within a farm. A farm may have multiple orchards (separate parcels). Section 3.3.1.

**Organic.** Production system with restricted inputs (no synthetic pesticides, no synthetic fertilizers; OMRI-listed products only). Out of V2's scope per Section 1.7.

**Override (V2).** A grower's explicit act of proceeding with an application despite an advisory rule flagging a warn or block. Recorded in `rule_overrides` with reason. Hard rules are not casually overridable. Section 4.8.

## P

**PCP number.** Pest Control Product registration number issued by Health Canada's PMRA. Every registered pesticide has one. V2 stores it in `products.pcp_number`.

**Petal fall.** Stage when the petals have dropped. Often the reference point for PHI countdowns, codling-moth biofix, and scab transition from primary to secondary season.

**Phenology.** The study of developmental stages as they progress through the season. Apple phenology is typically tracked in 8–10 stages from dormant through fruit-set. Driven by accumulated temperature.

**Phenology photo (V2).** A user-uploaded photo of a branch tagged with the confirmed bloom stage. V2 uses optional photo prompts to validate DD-based auto-advance. Section 3.11.3.

**PHI (Pre-Harvest Interval).** The number of days that must elapse between the last application of a pesticide and harvest. Labeled per product. V2's PHI rule is hard. Section 4.6.5.

**Pheromone trap.** A trap baited with a species-specific sex pheromone, used to monitor pest pressure and establish biofix. V2 captures per-block trap records. Section 3.11.4.

**Playwright.** Browser automation library. Used for V2's end-to-end tests and optionally for scraping JavaScript-heavy sites. Section 9.11.

**Plum curculio.** *Conotrachelus nenuphar.* Native North American weevil. Attacks fruit around petal fall. DD-based timing model.

**PMRA.** Pest Management Regulatory Agency, Health Canada. Federal pesticide regulator. Maintains the Public Registry of registered pesticides. Source of truth for the legal label.

**Postgres / PostgreSQL.** V2's database. Section 9.3.

**Powdery mildew.** *Podosphaera leucotricha.* Fungal disease that thrives in warm, humid, non-free-water conditions (opposite of scab). Chalky white coating. Managed with fungicides and resistant varieties.

**Published fixture.** A fixture test drawn from a published source (paper, extension worked example). Contrast with **constructed fixture**, which is author-built where no published example exists. Section 6.3.

## Q

**Quarterly PCP check (V2).** V2's commitment to cross-check every published product's PCP number against PMRA's Public Registry once per quarter, to catch de-registrations. Section 5.7.

## R

**Railway.** Hosting platform. V2's application and database host. Section 9.5.

**REI (Re-Entry Interval).** Hours after application during which workers cannot enter the treated area without PPE. Labeled per product; some labels specify per-work-type REI (hand harvest, hand thinning, scouting). V2's REI rule feeds the Go/No-Go panel. Section 4.6.6.

**Replay (V2).** Short for scenario replay — running a V2 model against 2026 V1 season data and reviewing whether V2's outputs are defensible. Required for Tier-1 models before advisor-enabling. Section 6.10.

**Resend.** Email-sending service. Section 9.15.

**Resistance management.** Strategies (rotation across modes of action, avoiding back-to-back same-group applications, seasonal maxima) that delay the development of pest or pathogen populations resistant to the product. FRAC, IRAC, and HRAC guidance is centered on resistance management.

**RLS (Row-Level Security).** A Postgres feature that applies access filters at the database level rather than only in application code. V2 uses RLS to enforce farm isolation as a second line of defense. Section 3.15.

**Rootstock.** The lower tree component onto which a variety is grafted. Determines vigor, cold hardiness, fire blight susceptibility, and other traits. Common rootstocks: M.9, M.26, G.41, G.11, B.9.

**Rule (V2).** A function that evaluates a candidate product against a context and returns a pass/warn/block outcome. Rules come in twelve categories. Section 4.6.

**Rule engine (V2).** The V2 subsystem that runs rules on candidates and aggregates outcomes into verdicts. Section 4.2.

## S

**Scab.** Short for apple scab.

**Scenario replay.** See **Replay**.

**Scouting.** Direct in-field observation of disease symptoms, pest presence, weed populations, or nutrient deficiencies. V2 treats scouting observations as trigger inputs. Section 2.5.

**Scouting-surface (V2).** A disposition tag for V1 conditions that don't merit a formal model in V2 but are still worth surfacing. A scouting-surface shows current weather, scouting guidance, OMAFRA program reference, and related context, without producing a risk score. Section 2.1.1.

**Secondary scab season.** Period after primary scab season ends (roughly after petal-fall through summer). Infection driven by conidia (asexual spores) rather than ascospores. Risk declines but doesn't vanish.

**Sentry.** Error tracking service. Section 9.14.

**Severity (rule).** A rule's declared weight: `hard`, `advisory`, or `informational`. Not the same as a rule's `outcome` (pass/warn/block). Section 4.5.

**shadcn/ui.** React component library. Unstyled primitives that can be customized. Section 9.7.

**SKU.** A specific packaged unit of a product (different trade formulations of the same a.i. are different SKUs with different PCP numbers). V2's catalogue is SKU-level, not a.i.-level.

**Spray.** Informal synonym for application. In V2's code, `applications` is the unified table; "spray" is used colloquially but not as a named entity.

**Station priority (V2).** When multiple weather sources report data for the same timestamp, the source with higher `weather_stations.priority` wins. Section 2.7.

## T

**Tank mix.** A combination of products sprayed together in one tank. V2 groups these via `tank_mix_groups`. Compatibility rules apply. Section 3.5.3.

**Tank mix template.** A named, saved tank mix configuration that a grower reuses across the season. Section 3.5.4.

**Target slug.** V2's string identifier for a pest, disease, weed, or deficiency. Examples: `apple_scab`, `codling_moth`, `late_season_nitrogen`, `common_ragweed`. Used throughout the catalogue, rules, and triggers.

**Tier (model).** V2 groups models into tiers based on evidential strength. Tier 1: published algorithms with real consequence (scab, fire blight, codling moth, frost). Tier 2: surviving models beyond Tier 1 (bitter pit, powdery mildew, etc.). Tier 3: DD-based timing models sharing the DD accumulator. Tier 4: static informational content (demoted). SCOUTING-SURFACE: weather-aware advisories without risk scores.

**ToS (Terms of Service).** V2's legal framing document. Self-drafted with legal-review-planned posture. Section 7.

**Tree fruit.** The OMAFRA / USDA category that includes apples, pears, peaches, plums, cherries. V2 is apple-only; other tree fruit uses different rules.

**Trigger (V2).** An event fired by a state transition (model output changing, test result coming in, scouting observation logged). Triggers feed the advisor. Section 2.2.

**tRPC.** TypeScript-native API layer for in-app client-server calls. Section 9.8.

**Twilio.** SMS delivery service. Section 9.15.

## U

**Underlayer / groundcover.** The orchard floor management layer — weeds, grass, mulch, cover crops. V2's weed advisor reasons over this. Section 2.1.4.

**Urea.** Nitrogen fertilizer. Foliar urea at leaf-drop suppresses overwintering scab via leaf-litter decomposition. Soil urea applied in spring.

## V

**Variety.** Synonym for cultivar. Honeycrisp, McIntosh, Gala, etc. V2 tracks variety per planting (multiple varieties per block possible). Section 3.3.3.

**Vitest.** TypeScript test runner. Section 9.13.

## W

**Weather source (V2).** The origin of a weather observation or forecast. Sources: Open-Meteo, Environment Canada, and (post-2027) on-orchard station hardware. Each has a priority. Section 3.6.1.

**Weed pressure.** A subjective or scouted measure of weed density and aggressiveness in the orchard. V2's weed advisor reads scouting observations for pressure. Section 3.11.6.

## Z

**Zero Data Retention (ZDR).** A posture where a service does not log or retain customer requests. V2 does not take a formal ZDR posture; the ToS documents what is retained and for how long. Section 7.8.

---

> **End of glossary.** Add entries as new terminology surfaces during development. Cross-reference to planning doc sections where possible.

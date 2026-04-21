# OrchardGuard V2 — Scenario Replay Methodology

> **Purpose.** Specify how Tier-1 model scenario replay is done. Referenced by planning doc Section 6.10.
> **Status.** Methodology draft. Finalize before first Tier-1 replay run (Phase 2 for apple scab).
> **Companion to.** Scenario replay artifact template (`replay-review-template.md`).

---

## 1. Purpose of Replay

Scenario replay is V2's mechanism for catching model defects that fixture tests cannot. Fixtures use clean synthetic inputs drawn from published papers. Real weather data has noise, gaps, odd edge cases, and unusual patterns; a model that passes fixtures can still behave poorly on real data.

Replay takes **2026 V1 season data** — weather, orchard state, bloom progression, spray history, observed outcomes — and runs the V2 model against it. The author then reviews whether the V2 outputs are defensible.

**Replay is not a comparison to V1.** V1's output is context, not the benchmark. The comparison target is:

1. What the published algorithm says should happen at those conditions (fixture-level expectation extended to real data)
2. What OMAFRA program-level guidance recommends for those conditions
3. What actually happened in 2026 — did the disease/pest materialize, did the grower spray, was the decision correct in hindsight

V2 must do better than V1 in aggregate, not match V1.

## 2. Which Models Get Replay

Replay is required for **Tier 1 models only**:

- Apple scab
- Fire blight
- Codling moth
- Frost risk

Replay is encouraged for other models post-launch as the tooling is reusable, but not required for Tier 2 or Tier 3 pre-launch. This follows the tier-asymmetric correctness approach of Section 6.

A Tier-1 model can be merged and live in the codebase with fixture tests passing, but does not drive advisor recommendations until scenario replay is complete and reviewed. This is the **merge-eligible vs. advisor-enabled** distinction (Section 6.7). Enforcement: `model_citations.advisor_enabled` flag.

## 3. Data Inputs for Replay

Replay depends on 2026 V1 data being captured in a replayable form. V1's existing storage is adequate; no V1 changes required. Required inputs:

- **Hourly weather data** for the author's orchard for the full 2026 season (from V1's `weather_hourly` table). Source: Open-Meteo + Environment Canada, the same sources V2 uses at launch. Hardware-station data is not in scope (station purchase deferred to 2027 per Section 9.10).
- **Daily weather aggregates** (derivable from hourly).
- **Bloom stage progression** — V1's `orchards.bloom_stage` timestamps plus any manual transitions logged.
- **Application history** — V1's `spray_log` entries with dates, products, targets, rates.
- **Observed outcomes** — scouting observations, disease/pest incidence, any harvest-end qualitative notes. Authors' own memory is acceptable for the first replay pass; formal scouting records are preferred where they exist.
- **Orchard configuration** — block structure, plantings, variety mix, rootstocks, known juniper proximity, etc.

The V1 → V2 ETL script (Phase 5 deliverable per planning doc Section 9.18) produces these inputs in V2-compatible form. For pre-launch replay (Phase 2–4), a lightweight extract can be used without waiting for the full ETL.

## 4. Scenario Selection Methodology

Each Tier-1 replay reviews **10–20 scenarios** drawn from the 2026 season. Scenarios are single days or short windows. Selection aims for coverage across phenology, weather variety, and key decision points.

### Required coverage per model

**Apple scab:**

- At least one day from each phenology stage from green-tip through primary scab season end
- Every detected primary scab infection event in 2026 (every row in V1's `scab_infection_log` — verify V1 captures these)
- At least 2 borderline-wet-period days (wetness near but not at Mills thresholds)
- At least 1 long-wetness-period day (severe infection territory)
- At least 1 day where the grower applied a fungicide (to validate kickback timer behavior)
- At least 1 day after petal fall when primary scab season has ended

**Fire blight:**

- Every day during 2026 bloom where temperature triggered a CougarBlight or MaryBlyt flag
- At least 2 borderline days (conditions near but not meeting the 4-condition gate)
- At least 1 day during pre-bloom with high heat (to test the anti-alarmism posture)
- At least 1 day post-bloom with high heat (symptoms-appear projection)
- If 2026 had any fire blight pressure in the author's area, specific days matching that pressure

**Codling moth:**

- Biofix date itself
- 50 DD post-biofix (egg hatch begins)
- 100 DD (early hatch; scouting threshold)
- 250 DD (peak egg hatch; spray window)
- 450 DD (first generation end)
- First trap catch of second generation (second biofix)
- At least one day where the grower sprayed for CM (validates protected-period logic)

**Frost risk:**

- Every day in 2026 with a low temperature below +2°C between silver-tip and petal-fall
- At least one severe frost event if any occurred
- At least one forecast-warned-but-didn't-materialize event (validates anti-false-alarm behavior)

### Selection process

1. Author pulls the raw data list of candidate scenarios meeting the "required coverage" criteria.
2. Author selects a subset of 10–20 scenarios covering the criteria. When there are many candidates (e.g., every day during bloom for fire blight), pick a representative spread rather than every single day.
3. Scenario list is committed to the replay artifact file at the top, with rationale for each selected scenario.

## 5. Running the Replay

The scenario replay harness (tooling, built in Phase 2) takes a scenario specification and runs the V2 model against the 2026 data slice corresponding to that scenario.

Harness expected behavior:

- Loads 2026 V1 data for the specified date and the preceding window (typically 7–14 days depending on model).
- Constructs a V2 context object matching what the model would have seen on that day in 2026.
- Runs the V2 model.
- Captures the full output (risk level, risk score, intermediate values, triggered events).
- Writes the output to the replay artifact alongside the scenario definition.

Harness code lives in `tools/scenario-replay/` (separate from the main app; not deployed to production).

## 6. Review Against Expectations

For each scenario, the author records three evaluations in the replay artifact:

### 6.1 Published-algorithm check

Does V2's output match what the algorithm, faithfully implemented, should produce for these inputs? This is the fixture check extended to real data.

- **Match:** Algorithm behavior looks correct.
- **Near-miss:** Output close to expected but differs by a margin that merits inspection (note tolerance and why).
- **Mismatch:** Output diverges from algorithm expectation. Investigate before any advisor enablement.

### 6.2 OMAFRA guidance check

Does V2's output align with OMAFRA's program-level recommendation for these conditions? OMAFRA publishes per-stage scab program guidance, fire blight action-threshold guidance, etc. V2's risk-level call should be consistent with OMAFRA's stated action thresholds.

- **Consistent:** V2 and OMAFRA would recommend similar action.
- **More cautious:** V2 recommends action where OMAFRA would not, or recommends higher urgency.
- **Less cautious:** V2 does not recommend action where OMAFRA would, or recommends lower urgency. **Flag for investigation.**

### 6.3 Ground-truth check

Did the condition actually materialize in 2026? Was V2's implicit recommendation correct in hindsight?

- **Correct call:** V2 predicted the outcome that occurred.
- **Correct call, different reason:** V2 arrived at the right call via a questionable path.
- **False alarm:** V2 predicted risk that didn't materialize (acceptable at low frequency; unacceptable if pervasive).
- **Missed event:** V2 did not predict a condition that did materialize. **High-priority investigation.**

Ground truth is imperfect. An unsprayed block with no observed disease does not prove the model was right to predict low risk — the risk simply didn't materialize. The 2026 V1 data includes whether the author sprayed, which provides information about V1's call but not about true risk. Authors should document uncertainty in the ground-truth column rather than fabricating certainty.

## 7. Overall Model Judgment

After reviewing all scenarios, the author produces an **overall judgment** for the model:

- **Advisor-enable.** Model's behavior is defensible across the scenario set. Minor issues are documented and do not block enablement. The `model_citations.advisor_enabled` flag is flipped to true in the registry-update PR that merges the replay artifact.
- **Advisor-enable with noted limitations.** Model is mostly defensible, but has specific known behaviors (e.g., "tends to be over-cautious during pre-bloom warm spells") that growers should be aware of. Behaviors go in the known-issue log (planning doc Section 7.6). Advisor-enabled with a note surfaced in the per-recommendation disclosure.
- **Do not advisor-enable yet.** Replay surfaces problems that must be investigated and fixed before enablement. Issues documented; PRs track resolution. Model stays in codebase for other purposes (model_runs archive, research) but does not drive recommendations.

## 8. Artifact Storage

Every replay run produces an artifact at `replay/<model_slug>_2026.md` using the template in `replay-review-template.md`. The artifact is committed to the repo alongside the model.

When a model is re-replayed (e.g., after a parameters change, or after station hardware integration in 2027), the new artifact is versioned: `replay/<model_slug>_2026_v2.md`, `replay/<model_slug>_2027.md`, etc. Earlier artifacts are retained.

The PR that enables a Tier-1 model includes both (a) the artifact and (b) the registry flag flip.

## 9. Re-Replay Triggers

A Tier-1 model triggers re-replay in any of these cases:

- **Parameters version bump** (detected via snapshot-fixture hash change at CI, Section 6.8).
- **Algorithm change** that goes beyond a pure refactor (reviewer judgment).
- **Source revision** — OMAFRA or upstream paper publishes a material update to the algorithm.
- **Annual review** — 12 months since last replay. Not strictly re-run; annual review may conclude "no changes needed" without re-running.
- **Incident response** — if a recommendation failure traces to the model.

Re-replay can reuse the existing scenario set or expand it with new coverage.

## 10. Post-V2 Extensions

Natural extensions beyond V2 launch:

- **Replay for non-Tier-1 models.** Tooling is reusable; extend to Tier-2 surviving models as bandwidth allows.
- **Station-data replay pass.** Once 2027+ data includes on-orchard station measurements, re-replay Tier-1 models with improved leaf-wetness and soil-temperature inputs.
- **Cross-farm replay.** As invited growers accumulate their own seasons of data, replay against other farms' data checks for site-specific behavior. Requires explicit consent per Section 7.2 data-use.

## 11. Checklist Before First Replay

- [ ] 2026 V1 data is accessible in a readable form (SQLite file or extract).
- [ ] V2 model is merged with fixture tests passing.
- [ ] Scenario-replay harness tooling exists and has been smoke-tested.
- [ ] Scenario list drafted per Section 4 coverage requirements.
- [ ] Replay artifact template (`replay-review-template.md`) in the repo.
- [ ] Time budget: 4–8 hours per Tier-1 model per Section 6.11.

## 12. Related Files

- Replay review artifact template — `replay-review-template.md`
- Planning doc Section 6.10 — Scenario Replay for Tier-1 Models
- Planning doc Section 6.11 — V1 Model Carry-Forward
- Planning doc Section 6.7 — The Gate in Practice
- Planning doc Section 6.12 — Author's Blind-Spot Problem

---

> **End of methodology.** First use: Phase 2, apple scab.

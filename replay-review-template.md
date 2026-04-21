# Scenario Replay — <Model Name> — 2026 Season

> **Model slug.** `<slug>`
> **Algorithm.** <e.g., Modified Mills Table + NH ascospore maturity curve>
> **Parameters version at replay.** <YYYY-MM-DD or tag>
> **Replay run date.** <YYYY-MM-DD>
> **Reviewer.** <author name>
> **Overall judgment.** [ ] Advisor-enable   [ ] Advisor-enable with noted limitations   [ ] Do not advisor-enable yet

---

## 1. Scope of This Replay

<Which 2026 season was used (full season? partial?). Which orchard (the author's, presumably). Any data quality caveats — gaps in weather, missing scouting records, etc.>

## 2. Scenario Set

Total scenarios reviewed: <N>

Coverage against methodology requirements (`scenario-replay-methodology.md` Section 4):

| Required coverage | Scenario(s) covering it |
|---|---|
| <requirement 1> | <scenario ID> |
| <requirement 2> | <scenario ID> |
| ... | |

### Scenarios

Each scenario is a single day or short window. Listed in chronological order.

---

### Scenario S01 — <short title>

**Date / window:** <YYYY-MM-DD> (or range)
**Phenology stage at this point:** <e.g., green-tip, pink, petal-fall>
**Rationale for inclusion:** <why this scenario is part of the review set>

**Inputs summary**
- Weather window: <describe the weather conditions the model is seeing — temps, wetness, etc.>
- Orchard state: <bloom stage, relevant block context>
- Recent applications: <what V1 applied in the prior N days, if relevant>
- Active triggers at this point: <what other V2 models would be firing>

**V2 output**
```
<paste the full model output object — risk level, risk score, intermediate
 values, triggered events, etc.>
```

**Evaluation**

| Check | Result | Notes |
|---|---|---|
| Published-algorithm check | ☐ Match ☐ Near-miss ☐ Mismatch | <one-line rationale> |
| OMAFRA guidance check | ☐ Consistent ☐ More cautious ☐ Less cautious | <one-line rationale; cite OMAFRA section if possible> |
| Ground-truth check | ☐ Correct call ☐ Correct call, different reason ☐ False alarm ☐ Missed event ☐ Uncertain | <one-line rationale> |

**Reviewer notes.** <any specifics worth recording — why the output was surprising, where it matches expectation, what V1 did differently, anything that affects the overall judgment>

---

### Scenario S02 — <short title>

<...same structure...>

---

### Scenario S03 — <short title>

<...>

---

<... additional scenarios as needed, 10–20 total ...>

---

## 3. Aggregate Patterns

<After reviewing all scenarios individually, summarize patterns across them.>

- **Where V2 consistently does well:** <e.g., "handled borderline wetness days with appropriate calls based on Mills thresholds">
- **Where V2 deviates from expectation, and how often:** <e.g., "in 3 of 18 scenarios V2 called 'moderate' where OMAFRA guidance suggested 'light' — all three involved wet periods with gaps bridged by the 2-hour tolerance">
- **Where V2 differs from V1 output meaningfully:** <e.g., "V2 now accounts for block-level juniper proximity in cedar rust; V1 was orchard-wide">
- **Where ground truth confirmed V2's calls:** <e.g., "the two severe-infection predictions matched observed scab lesions on unsprayed rows">

## 4. Issues Surfaced

Issues that investigation may resolve. Each gets a severity tag.

| # | Severity | Description | Scenario(s) | Proposed action |
|---|---|---|---|---|
| I1 | <high | medium | low> | <short description of the issue> | <which scenarios showed it> | <e.g., "investigate parameter X", "add edge-case test", "file known-issue log entry"> |
| I2 | | | | |

**High-severity issues block advisor-enablement.** Medium-severity issues are advisor-enable-with-noted-limitations candidates. Low-severity issues become backlog items.

## 5. Comparison to V1

Not the benchmark, but worth noting where V2's block-awareness, rebuilt parameters, or refined algorithm changes behavior compared to V1's 2026 recommendations.

- **Cases where V2 would have advised action and V1 did not:** <...>
- **Cases where V1 advised action and V2 would not:** <...>
- **Cases where both matched:** <approximate count; detail only if noteworthy>

This comparison is informational. V2's defensibility is not measured by agreement with V1.

## 6. OMAFRA Cross-Reference

Spot-check of V2 outputs against OMAFRA's current program recommendations for <target>:

- <URL to OMAFRA reference page>
- <Specific program stage or guidance that was cross-checked>
- <Result: V2 aligns / diverges in noted ways>

## 7. Known Limitations

Behaviors the model has that are acceptable but worth surfacing to growers:

- <e.g., "tends to be more cautious than V1 during pre-bloom warm spells due to CougarBlight temperature cap behavior">
- <e.g., "cannot distinguish between genuine leaf-wetness and sensor-estimated leaf-wetness; estimates may over- or under-call on specific weather patterns until station hardware is added">

These go into the public known-issue log (planning doc Section 7.6) if advisor-enabled.

## 8. Decision

<Final overall judgment statement, tying together the scenarios, aggregate patterns, issues, and limitations.>

**Recommendation:** [ ] Advisor-enable   [ ] Advisor-enable with noted limitations   [ ] Do not advisor-enable yet

**If advisor-enabled with noted limitations:** list the specific limitations that will appear in user-facing disclosures and the known-issue log.

**If do not advisor-enable yet:** list the specific investigation tasks that must resolve before re-review.

## 9. Next Steps

- [ ] Merge this artifact
- [ ] Update `model_citations.advisor_enabled` registry (if advisor-enabling)
- [ ] File known-issue-log entries for any "noted limitations" or "medium-severity" issues
- [ ] Open backlog items for any "low-severity" issues or planned improvements
- [ ] Set next-review calendar reminder for 12 months out (or sooner if parameters change)

---

> **End of replay artifact.** Commit alongside the model source file and the PR that flips the registry flag.

# OrchardGuard V2 — Model Citation Header Template

> **Purpose.** Standard format for the citation block at the top of every model source file. Read at startup by the model runner to populate `model_citations` table (planning doc Section 6.2).
> **Status.** Template for use from Phase 2 onward. First applied to apple scab model during Phase 2 vertical slice.

---

## When to Use

Every model file in `lib/models/*` carries this header as its first content block, before imports. Applies to:

- Tier 1 models (scab, fire blight, codling moth, frost)
- Tier 2 surviving models (powdery mildew, sooty blotch/flyspeck, phytophthora, bitter pit, sunburn, water core)
- Tier 3 surviving pest DD-based models
- Nutrition evaluators

Does **not** apply to:

- Weed evaluators (not epidemiological models)
- SCOUTING-SURFACE advisory content (not code)
- Demoted Tier-4 reference pages

## Required Fields

| Field | Required? | Notes |
|---|---|---|
| Model | yes | Human-readable name ("Apple Scab Infection") |
| Slug | yes | Matches `model_citations.model_slug` and DB usage |
| Tier | yes | 1, 2, or 3 |
| Algorithm | yes | Short name of the algorithm being implemented ("Modified Mills Table + NH ascospore maturity curve") |
| Primary source | yes | Full citation of the canonical reference |
| Primary source URL | if available | Link to the reference or its abstract |
| Secondary sources | optional | Supporting papers, extension publications |
| OMAFRA reference | recommended | Link to OMAFRA's page or program covering this target |
| Parameters version | yes | Date tag or version string bumped when parameters change |
| Last reviewed against source | yes | Date + reviewer name; governs annual re-verification |
| Advisor-enabled | Tier 1 only | Boolean; flips true only after scenario replay review (Section 6.10) |

## Template

```typescript
/**
 * Model: <human-readable name>
 * Slug: <snake_case_slug>
 * Tier: <1 | 2 | 3>
 * Algorithm: <short algorithm description>
 *
 * Primary source:
 *   <full citation — authors, year, title, journal/publisher, page/chapter>
 *   URL: <link if available>
 *
 * Secondary sources:
 *   <citation>
 *   <citation>
 *
 * OMAFRA reference:
 *   <title of OMAFRA page or program>
 *   URL: <link>
 *
 * Parameters version: <YYYY-MM-DD or semantic tag>
 * Last reviewed against source: <YYYY-MM-DD> by <author or reviewer name>
 *
 * Advisor-enabled: <true | false>   // Tier 1 only
 *
 * Notes:
 *   <any model-specific implementation notes worth calling out —
 *    assumptions, limitations, known edge cases, relationship to other models>
 */
```

## Worked Example — Apple Scab

```typescript
/**
 * Model: Apple Scab Infection
 * Slug: apple_scab
 * Tier: 1
 * Algorithm: Modified Mills Table + NH ascospore maturity curve
 *
 * Primary source:
 *   MacHardy, W. E. (1996). Apple Scab: Biology, Epidemiology, and Management.
 *   APS Press. Chapter 7, Table 7.2 (Mills infection periods).
 *   URL: https://my.apsnet.org/APSStore/ProductDetail.aspx?ProductCode=42692
 *
 * Secondary sources:
 *   Gadoury, D. M., & MacHardy, W. E. (1982). A model to estimate the maturity
 *   of ascospores of Venturia inaequalis. Phytopathology, 72(7), 901–904.
 *
 *   Mills, W. D. (1944). Efficient use of sulfur dusts and sprays during rain
 *   to control apple scab. Cornell Extension Bulletin 630.
 *
 * OMAFRA reference:
 *   OMAFRA Publication 310, Apple Scab Management.
 *   URL: https://www.ontario.ca/page/apple-scab
 *
 * Parameters version: 2026-03-14
 * Last reviewed against source: 2026-03-14 by N. [author last name]
 *
 * Advisor-enabled: false
 *
 * Notes:
 *   Kickback hour values (Syllit 48, Nova 72, Inspire Super 96) are sourced
 *   from product labels, not from the Mills reference. These feed rule-engine
 *   kickback rules and are decoupled from the infection algorithm itself.
 *
 *   Wet-period bridging logic (2-hour gap tolerance) follows V1 implementation
 *   per V1 audit Section 10. Change with care.
 */
```

## Reconciliation with `model_citations` table

At application startup, a reconciler reads every model file's header and writes to `model_citations`:

- Insert if `model_slug` is new.
- Update if header differs from current row.
- Log an event if `parameters_version` changed (this is the drift signal — see Section 6.8 snapshot fixtures).
- Log an event if `last_reviewed_against_source` is older than 365 days — surfaces in the admin UI as "needs re-verification" (Section 6.13).

The reconciler does **not** delete rows. A model removed from the codebase leaves its row behind with no associated file; stale rows surface in a weekly admin report.

## Enforcement

- CI validates that every `lib/models/*/model.ts` file contains a parseable header block.
- CI validates that `model_slug` values are unique across files.
- CI validates that Tier 1 models with `advisor-enabled: true` have a corresponding replay review artifact at `replay/<slug>_2026.md` (Section 6.10).
- Missing or malformed header fails the CI build.

## Related Files

- Replay review artifact template — see `replay-review-template.md`
- Rule authoring template — see `rule-authoring-template.md`
- Planning doc Section 6.2 — Citation Structure
- Planning doc Section 6.7 — The Gate in Practice

---

> **End of template.** Copy and fill when adding a new model.

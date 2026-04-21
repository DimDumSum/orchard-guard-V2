# OrchardGuard V2 — Known-Issue Log

> **Purpose.** Public log of known inaccuracies, defects, and limitations in V2, with status and expected resolution. Referenced by planning doc Sections 7.6 and 7.13 #9.
> **Status.** Template and initial structure. First real entries land when V2 begins invited-beta.
> **Audience.** Invited growers, the author, and Claude Code sessions needing to know what's known-broken.
> **Location.** Lives in the repo at `docs/known-issues.md`; mirrored to a publicly-readable URL linked from every recommendation surface and from the ToS.

---

## How Entries Get Here

An entry lands in this log when:

- A defect is identified but not yet fixed (catalogue data error, rule bug, model defect, etc.)
- A limitation is inherent and grower-facing (e.g., "nutrition guidance is based on OMAFRA Pub 360; evidence base is thinner than for disease/pest models")
- A scenario-replay review surfaces a noted-limitation that affects advisor-enabled behavior
- An incident response (see `incident-response-runbook.md`) identifies a class of issue that's worth publicly disclosing

An entry leaves this log when:

- The issue is fully resolved (moves to the archive below)
- The issue is reclassified as not-a-defect after investigation

Entries are **never silently deleted**. Resolution moves an entry to the archive; reclassification updates status and keeps the entry.

## Entry Format

```markdown
## I-NNNN — <Short Title>

- **Status:** Open | In Progress | Resolved | Reclassified
- **Severity:** Critical | High | Medium | Low
- **Affects:** <which subsystem — model, rule, catalogue, UI, etc.>
- **Reported:** YYYY-MM-DD by <user or author>
- **First seen in:** <release version or date>
- **Expected resolution:** <target date or "post-V2 enhancement" or "no fix planned — limitation">

### Description

<Plain-language description of the issue. What goes wrong, under what conditions,
and what the user-visible effect is. Avoid jargon that a certified grower without
software background wouldn't know.>

### Current workaround

<What the grower should do in the meantime. "None — treat affected recommendations
with extra care" is a valid answer if that's truthful.>

### Root cause

<Brief technical note, if known. Link to the incident report or investigation
if one exists.>

### Tracking

<Link to the PR, issue, or investigation that's working toward resolution.>
```

## Entry Numbering

Sequential and stable. `I-0001`, `I-0002`, etc. Numbers never reuse. Numbers do not renumber when entries move to archive.

## Severity Definitions

- **Critical.** Affects recommendation correctness in a way that could cause grower harm if not known. User-facing disclosure is mandatory on affected surfaces.
- **High.** Affects recommendation correctness in a way that a grower should know. User-facing disclosure recommended; at minimum, the log entry is public.
- **Medium.** Incorrect or degraded behavior that doesn't threaten harm but should be fixed.
- **Low.** Cosmetic or edge-case issues.

## Open Issues

<!-- Placeholder. Populate as issues arise. -->

## Limitations (Permanent or Long-Term)

Known trade-offs baked into V2's scope or posture. Not defects — documented so growers know the boundaries.

### I-L001 — Weather station hardware deferred to 2027

- **Status:** Open limitation
- **Severity:** Medium
- **Affects:** Scab and fire blight model accuracy; pre-emergent herbicide timing
- **Reported:** <date V2 ships>
- **First seen in:** V2 launch

#### Description

V2 launches using public weather data (Open-Meteo and Environment Canada). Leaf wetness is estimated from humidity and precipitation rather than directly measured; soil temperature is approximated from air temperature. Direct measurement would materially improve apple scab and fire blight models and is the intended approach when an on-orchard station is added in 2027.

#### Current workaround

Growers should treat leaf-wetness-driven recommendations as informed estimates and cross-reference their own on-orchard observations when possible.

#### Root cause

V2 scope decision. Station purchase deferred to 2027 per planning doc Section 9.10.

#### Tracking

Post-V2 work item. See planning doc Section 10 — post-launch category.

### I-L002 — Nutrition evidence base is thinner than disease/pest models

- **Status:** Open limitation
- **Severity:** Medium
- **Affects:** Nutrition advisor recommendations
- **Reported:** <date V2 ships>
- **First seen in:** V2 launch

#### Description

V2's nutrition evaluators use OMAFRA Publication 360 sufficiency ranges and accumulated field experience as their source of truth. Published point-estimates-with-uncertainty for leaf-analysis triggers are less available than the equivalent research for scab or fire blight models. Most nutrition rules are advisory; hard nutrition rules (late-season nitrogen cutoff, labeled seasonal caps, pH overshoot) require fixture support and are defensible.

#### Current workaround

Growers should weigh nutrition recommendations alongside their own observations and knowledge of their orchard's history. Nutrition recommendations always cite OMAFRA Pub 360 where applicable.

#### Root cause

State of published nutrition evidence for Ontario apple production. Acknowledged in planning doc Section 6.9.

#### Tracking

Ongoing. If improved sources emerge, entries are added to the catalogue ingestion tracker.

### I-L003 — Catalogue coverage is grower-scoped at bootstrap

- **Status:** Open limitation
- **Severity:** Low
- **Affects:** Catalogue completeness for new invited growers
- **Reported:** <date V2 ships>
- **First seen in:** V2 launch

#### Description

Bootstrap catalogue covers ~100–150 products. A new invited grower may use a product that isn't yet Published. Onboarding for that grower waits until all their products are in the catalogue — either from the bootstrap set, or added through the request flow. This may delay onboarding by days or weeks depending on the product set.

#### Current workaround

New growers submit their full product list during the pre-onboarding intake. The author prioritizes those products in the catalogue ingestion pipeline.

#### Root cause

Intentional scope decision to avoid the 150–300-hour pre-launch burden of populating "all registered products." Planning doc Section 5.2.

#### Tracking

Catalogue grows organically as growers join. No target completion date.

## Resolved Issues (Archive)

<!-- Resolved issues move here with their resolution date and a link to the fix PR or commit. -->

## Reclassified Issues

<!-- Issues that turned out not to be defects go here with the reclassification rationale. -->

---

## Related Files

- Planning doc Section 7.6 — Model and Catalogue Correctness Disclosures
- Planning doc Section 7.13 #9 — Known-issue log format open question (now resolved by this document)
- Incident response runbook — `incident-response-runbook.md`
- Release checklist — references this log

---

> **End of known-issue log template.** Populate as issues arise. Public URL links from the app.

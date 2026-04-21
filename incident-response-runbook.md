# OrchardGuard V2 — Incident Response Runbook

> **Purpose.** Procedure for responding when something goes wrong — when V2 produces a recommendation that contributes to a crop-loss event, an off-label application, or another consequential outcome. Referenced by planning doc Section 7.10.
> **Status.** Initial draft. Refine after first live use.
> **Audience.** The author / project operator. Also referenced in the grower onboarding runbook as what-happens-if-something-goes-wrong.

---

## 1. What Counts as an Incident

For V2's purposes, an incident is any event where:

- A grower believes a V2 recommendation contributed to crop loss, off-label application, a regulatory violation, or significant economic harm
- A rule or model failure in V2 caused the advisor to suggest something defensibly wrong
- A catalogue-data error caused a recommendation that violated label requirements
- An outage or data issue prevented a grower from acting on time-sensitive information

Note: not every grower complaint is an incident. "The app suggested X and I disagreed" is feedback, not an incident. An incident has an actual consequence.

## 2. Severity Levels

| Severity | Description | Example |
|---|---|---|
| **P1 Critical** | Actual harm occurred; regulatory or safety implications | Off-label application made based on V2 recommendation; visible crop loss attributed to V2 recommendation |
| **P2 High** | Near-miss or significant defect; grower had to override V2 to avoid harm | Grower caught a wrong PHI in the catalogue before applying; V2 recommended something that would have violated label |
| **P3 Moderate** | Incorrect behavior without consequence | Model output was wrong but grower noticed and ignored it; alert didn't fire when it should have |
| **P4 Low** | Correctness concern | A catalogue field is wrong but in a way that doesn't affect recommendations; a rule citation points to a wrong source |

P1 and P2 run the full protocol in this document. P3 and P4 route through the known-issue log and the 7-day discrepancy response SLA without escalation.

## 3. Immediate Response — First 24 Hours

### 3.1 Receive the report

A grower reports an incident via:
- Direct contact (email, phone, text) to the author
- In-app discrepancy report (Section 7.9)
- Third-party notification (rare; e.g., another grower or industry contact raising concern)

### 3.2 Acknowledge receipt

Within 4 hours for P1, within 24 hours for P2. Even if the response is "I'm investigating," the grower needs to know the report was received and is being taken seriously.

```
I've received your report and am treating this as a serious matter. I'm
pulling the context now. I will have initial findings to you within 48 hours.
In the meantime, please document what happened (photos, notes, timestamps)
as thoroughly as you can while events are fresh.
```

### 3.3 Preserve evidence

From V2's side:

- [ ] Pull the relevant `advisor_sessions` row(s)
- [ ] Pull all associated `rule_evaluations` rows
- [ ] Pull all associated `model_runs` rows
- [ ] Pull the `applications` row for the spray in question
- [ ] Pull the catalogue state for the product(s) involved at the relevant timestamps
- [ ] Pull any `rule_overrides` rows associated
- [ ] Pull `triggers` and `alerts` rows for the window
- [ ] Pull relevant `weather_hourly` data for the window
- [ ] Pull `block_states` history for the block
- [ ] Take a snapshot of the current catalogue data for the affected products

Everything above is dumped into a preservation folder (`incidents/<YYYY-MM-DD>-<short-slug>/evidence/`) and committed to a private repo (not public), along with a checksum file so integrity is provable later.

### 3.4 Verify the report

- [ ] Does V2's archive corroborate the grower's account? (What did V2 actually show them? When?)
- [ ] Does the grower's spray log match what was sprayed? (Were they acting on V2's recommendation or on something else?)
- [ ] Was V2's recommendation defensible given the catalogue state and model state at the time?
- [ ] Was there a catalogue data error, a rule bug, a model bug, or was V2 correct and the grower's interpretation incorrect?

## 4. Investigation — First Week

### 4.1 Determine root cause

One of these is the answer:

- **Catalogue data error.** A product's PHI, seasonal max, rate, or other field was wrong in V2.
- **Rule logic bug.** A rule evaluated incorrectly given correct inputs.
- **Model defect.** A model produced output inconsistent with its algorithm.
- **Model limitation.** Model produced output consistent with its algorithm, but the algorithm itself was inadequate for the situation (e.g., leaf-wetness estimation wrong because of unusual weather patterns).
- **UI/UX problem.** The recommendation was correct but the grower misinterpreted what V2 showed.
- **User action beyond recommendation.** The grower did something V2 didn't recommend (e.g., applied at a different rate, mixed with an incompatible product).
- **External factor.** Weather data was wrong, label source had updated without V2 catching it, etc.
- **Combination.** More than one factor.

Root cause determination requires reading the archived evidence carefully. Do not short-circuit this. Write up findings in the incident report (see 4.3).

### 4.2 Contain further exposure

If the root cause could affect other growers:

- [ ] Review which other growers (if any) received the same recommendation in the relevant window
- [ ] If catalogue error: fix in the catalogue immediately, flag `label_ingested_at` to reflect re-verification needed
- [ ] If rule bug: disable the rule via registry (Section 4.9) if possible, or hotfix with tests
- [ ] If model defect: disable the model at advisor level (Section 6.15 #4 — registry-flag-driven skip)
- [ ] If model limitation: add to known-issue log with user-facing disclosure
- [ ] If weather data issue: flag the source, add disclosures to affected surfaces

### 4.3 Write the incident report

`incidents/<YYYY-MM-DD>-<short-slug>/report.md` with structure:

```markdown
# Incident <YYYY-MM-DD> — <short title>

- Severity: P1 | P2 | P3 | P4
- Reported by: <grower>
- Reported at: <timestamp>
- Acknowledged at: <timestamp>
- Root cause identified at: <timestamp>
- Resolution: <brief summary>

## Timeline
<What happened, when, from V2's perspective and from the grower's.>

## What V2 Showed
<Exact V2 output at the relevant time. Pasted from the archive, not paraphrased.>

## What the Grower Did
<Exact spray log or action taken.>

## What the Catalogue / Models Said
<Relevant data state at the time.>

## Root Cause
<One of the categories from 4.1, with justification.>

## Was the Recommendation Defensible?
<Explicit yes/no with reasoning.>

## Who Else Was Exposed?
<List of affected users, or "only the reporting grower".>

## Remediation
<What was changed in V2. Links to PRs, catalogue updates, registry flag changes.>

## Notification
<Who was notified, when, and with what message. Copies of the messages.>

## Lessons and Prevention
<Honest reflection on what could prevent the next incident of this class.>
```

### 4.4 Notify affected parties

- [ ] The reporting grower gets the full report (or a clearly-written summary)
- [ ] Other affected growers (if any) get a notification via their alert channel
- [ ] Known-issue log gets an entry (public)
- [ ] Model correctness posture page updated if a model or catalogue issue drove the incident

### 4.5 Fix and verify

- [ ] Remediation PR merged
- [ ] Tests added that would have caught the issue
- [ ] Regression test scenarios added
- [ ] Post-fix, re-run the advisor on the incident's conditions; verify the correct output now emerges
- [ ] Close the incident

## 5. Post-Incident

### 5.1 Retrospective (within 2 weeks)

A short written retrospective that is not just "what went wrong" but:

- What worked about the response itself
- What was slower than it should have been
- What monitoring or testing would have caught this earlier
- Whether V2's correctness posture (Section 6 and 7) holds up under this incident or needs adjustment

Stored in `incidents/<YYYY-MM-DD>-<short-slug>/retrospective.md`.

### 5.2 Policy updates if needed

- Does the ToS need adjustment? (Unlikely for a single incident; patterns of incidents might drive an update.)
- Does the fixture gate or replay methodology need tightening?
- Does the catalogue ingestion pipeline need an added check?
- Does the release checklist need a new item?

Any policy change is its own PR with its own reasoning.

## 6. Insurance and Legal Considerations

V2's invited-beta posture accepts that commercial liability insurance is not in place (planning doc Section 7.10). Incident response operates under that understanding.

For P1 incidents:

- [ ] Consult the legal review plan (Section 7.11) — does this incident's nature warrant advancing the lawyer engagement?
- [ ] If so: halt new grower onboarding until legal review is complete
- [ ] If the incident might have broader legal implications (e.g., a regulatory inquiry), follow the lawyer's guidance on communications

## 7. Insurance Claim (If Applicable)

V2's personal/professional umbrella coverage may apply depending on the incident:

- [ ] Review the umbrella policy's coverage of software-related incidents
- [ ] If coverage potentially applies: notify the insurer per their timing requirements (typically within X days of the incident)
- [ ] Preserve evidence per insurer's guidance

## 8. Regulatory Reporting

For incidents involving off-label applications or other regulatory-adjacent events, Ontario's pesticide regulatory framework may require reporting by the applicator (not by V2). V2 does not report on growers' behalf; V2 documents the recommendation state so the grower can accurately complete any required reporting.

- [ ] Ensure the grower has the V2 recommendation archive for their own reporting
- [ ] Do not file reports on the grower's behalf; the grower is the legal entity

## 9. Communication Templates

### To the reporting grower (acknowledgment)

See 3.2 above.

### To the reporting grower (root cause determined)

```
I've completed the investigation. Here's what I found:

<1-paragraph plain summary of what happened and why>

Full incident report is attached. Key points:

- [Whether V2 behaved as designed, or where it failed]
- [What was changed to prevent recurrence]
- [What this means for future recommendations]

I'm sorry this happened. [If V2's defect caused harm, acknowledge directly.
Do not use legalese to evade. Honesty here builds the trust V2 depends on.]

If you have questions about the findings or want to discuss further, please
reach out. I'm also available to walk through the report on a call.
```

### To other affected growers (if applicable)

```
I'm writing to let you know about an incident involving OrchardGuard that
may have affected recommendations you received between <dates>.

<Short description of the issue and the fix>

What you should do:
- <Specific guidance, e.g., "Review any applications of <product> logged
  during that window and verify against the current label">
- <If relevant, "Expect a catalogue update to appear in OrchardGuard in the
  next <timeframe>">

Full details are in the incident report at [link to public known-issue log].

I'm available to discuss if you have questions.
```

## 10. Related Files

- Planning doc Section 7.10 — Insurance and Incident Response
- Known-issue log (format TBD per Section 7.13 #9)
- Grower onboarding runbook — references this document
- Release checklist — for post-fix verification

---

> **End of incident response runbook.** May you never need it.

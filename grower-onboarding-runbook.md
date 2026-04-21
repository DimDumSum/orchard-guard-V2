# OrchardGuard V2 — Grower Onboarding Runbook

> **Purpose.** Step-by-step procedure the author follows when inviting a new grower into invited-beta. Referenced by planning doc Section 8.7 (Phase 5).
> **Status.** Draft. Will harden during Phase 5 after first grower onboarding.
> **Who uses it.** The author / project operator.

---

## 1. Philosophy

Onboarding an invited grower is not a sign-up flow — it is a curated start. Each grower enters V2 because the author invited them personally. The author knows enough about their operation to pre-configure much of it, and more importantly, the author has a **catalogue readiness gate** (Section 5.15 #7): every product the grower uses in practice must be Published in V2's catalogue before they onboard.

This runbook is deliberately slow. A rushed onboarding where the grower's actual products are missing from the catalogue produces "the app doesn't know about X" friction on day one, and that kills trust fast.

## 2. Pre-Onboarding (Weeks Before)

### 2.1 Grower selection

Candidate criteria (all must be true):

- Known personally, or via direct introduction by someone trusted
- Holds a current Ontario Grower Pesticide Safety Certificate or Farmer Pesticide License
- Operates a conventional or IPM apple orchard in Ontario
- Willing to serve as early-feedback participant (not silent; actually reports issues)
- Comfortable with shared-responsibility posture (tool is advisory; grower is legally responsible)
- Not expecting organic-specific support (V2 is out of scope for organic)

Criteria that weigh in the selection (not absolute):

- Similar scale and variety mix to the author's operation, or deliberately different for feedback diversity
- Spray practice aligns with V2's existing catalogue coverage
- Grower has reasonable access to weather data (Env Canada station or Open-Meteo works fine; on-orchard station not required since station purchase is deferred to 2027)

### 2.2 Pre-interview

Informal conversation with the candidate grower to confirm:

- [ ] They've been told the tool is invited-beta, not polished SaaS
- [ ] They understand the shared-responsibility framing
- [ ] They're willing to acknowledge the ToS and to report discrepancies
- [ ] They're prepared for the onboarding to take a few hours of their time across a few sessions

### 2.3 Intake form

Gather from the grower (a conversation or short form):

- [ ] Farm name
- [ ] Orchard name(s) and locations (lat/lng)
- [ ] Block structure — names, areas, soil type, irrigation system
- [ ] Planting details per block — varieties, rootstocks, year planted, tree counts, spacing
- [ ] Expected harvest dates per block-variety
- [ ] Primary varieties grown and their planting shares
- [ ] Their 2026 (or recent) spray program — products used, targets
- [ ] Any nutrition program — soil tests, leaf tests, amendments, foliars used
- [ ] Any herbicide program — products, targets
- [ ] Known disease/pest history per block
- [ ] Irrigation configuration if applicable
- [ ] Preferred alert channels (email, SMS)
- [ ] Preferred alert cadence (immediate vs. digest)

### 2.4 Catalogue readiness check

For each product in the grower's spray, nutrition, and herbicide programs:

- [ ] Check if it's in V2's catalogue at `ontario_registered = true, label_ingested_at set`
- [ ] For any product not yet published:
  - [ ] Add to the catalogue ingestion queue
  - [ ] Run through the full pipeline: Discovery → Scrape → Label Attached → Manual Entry → Verified → Published
  - [ ] Estimate: 30–60 minutes per product

Onboarding does not proceed until every one of the grower's active products is Published. This gate is absolute.

If the grower's program includes an out-of-V2-scope product (organic-only, non-apple-registered, etc.), they need to know before onboarding so they understand why V2 won't recommend that product.

### 2.5 ToS readiness

- [ ] Current ToS version is hosted at its stable URL
- [ ] Acknowledgment text matches the ToS version
- [ ] `user_acknowledgments` table is wired up and tested

## 3. Onboarding Session — Part 1 (Setup)

Roughly 1–2 hours with the grower, synchronous or async-assisted.

### 3.1 Create the invitation

- [ ] Admin UI: create `farm_invitations` row for the grower's email
- [ ] Invitation email sent (via Resend)
- [ ] Confirm the grower received the email

### 3.2 Grower acceptance

- [ ] Grower clicks invitation link, lands on signup flow
- [ ] Grower creates Clerk account
- [ ] Grower reads the ToS
- [ ] Grower reads and acknowledges each of the per-user acknowledgments (Section 7.3):
  - [ ] Certification self-attestation
  - [ ] Decision-support-not-prescription understanding
  - [ ] Legal responsibility acceptance
  - [ ] Model-output uncertainty acknowledgment
  - [ ] ToS acceptance
- [ ] `user_acknowledgments` row is written with full acknowledgment text
- [ ] `farm_memberships` row created with role `owner`

### 3.3 Farm configuration

With the grower on the call (or via screen-share):

- [ ] Create the farm in the admin UI
- [ ] Set farm timezone (typically `America/Toronto`)
- [ ] Create the orchard(s)
- [ ] Set orchard coordinates, elevation, microclimate defaults
- [ ] Configure weather sources (Open-Meteo + Env Canada; note that on-orchard station is 2027)
- [ ] Create blocks, confirm areas and soil types
- [ ] Create plantings within blocks (variety × rootstock × year × tree count × spacing)
- [ ] Set expected harvest dates per planting
- [ ] Populate disease history per block (fire blight, scab, summer diseases)
- [ ] Set preferred bloom stage (usually `dormant` at off-season onboarding)

### 3.4 Alert preferences

- [ ] Configure `alert_preferences` for the grower's user
- [ ] Set channel (email, and SMS if opted in)
- [ ] Set severity thresholds
- [ ] Set quiet hours if any
- [ ] Test an alert delivery (send a test trigger)

## 4. Onboarding Session — Part 2 (Data Import)

Roughly 1–2 hours. Can be same day as Part 1 or later.

### 4.1 Inventory

- [ ] Grower lists current product stock (or provides CSV)
- [ ] Each inventory lot entered into `inventory_lots` with lot number, quantity, expiry, lot-specific notes

### 4.2 Scouting and historical data (optional but useful)

- [ ] If the grower has recent soil tests: enter in `soil_tests`
- [ ] If recent leaf tests: enter in `leaf_tests`
- [ ] If pheromone traps are deployed: create `pheromone_traps` rows
- [ ] If recent scouting observations: enter in `scouting_observations`

### 4.3 Last-season reference data

Useful for the rule engine to have some history to reason from:

- [ ] Grower enters prior-season spray log summary (optional; can be a simple list rather than full entries)
- [ ] If V1 user: offer V1→V2 ETL (Phase 5 deliverable per planning doc Section 9.18) — automates this step
- [ ] For non-V1 growers, some history is entered manually; some is added as-it-happens in the new season

## 5. Onboarding Session — Part 3 (Walkthrough)

Roughly 30–60 minutes. Grower uses V2 on their own data while the author observes.

- [ ] Open the dashboard; review current conditions and any active triggers
- [ ] Open a block detail view; confirm variety, rootstock, phenology look right
- [ ] Walk through the advisor on an active trigger (or a seeded test trigger)
- [ ] Grower logs a test application (can be deleted after)
- [ ] Walk through the spray-plan view
- [ ] Show the REI Go/No-Go panel
- [ ] Show where to report discrepancies
- [ ] Show the ToS, model correctness posture page, and known-issue log
- [ ] Answer questions

## 6. Post-Onboarding (First 2 Weeks)

- [ ] Author checks in with grower at 3 days, 1 week, 2 weeks
- [ ] Any triage items from grower use are logged and resolved per the 7-day SLA
- [ ] First real advisor cycle is observed; author is on call if grower hits friction
- [ ] First real spray logging is observed

## 7. Post-Onboarding (First Season)

- [ ] Regular check-ins monthly
- [ ] All discrepancy reports acknowledged within 7 days
- [ ] Grower experience feedback captured in a private notes file
- [ ] At season end, conduct a debrief session covering: what worked, what didn't, what would they want changed

## 8. Second Grower

Second grower onboards only after first grower's debrief yields actionable findings. This pacing is deliberate — two simultaneous first-time growers means twice the triage load at the moment of highest product fragility.

## 9. Documentation for the Grower

Artifacts the grower receives (or has linked from the app):

- [ ] Link to the hosted ToS
- [ ] Link to the model correctness posture page
- [ ] Link to the known-issue log
- [ ] Short "getting started" guide (one page; where to click for common tasks)
- [ ] Emergency contact for the author (phone, email)
- [ ] Link to V2's privacy posture summary

## 10. If Something Goes Wrong

If the grower encounters a recommendation that contributes to a crop issue:

- [ ] Activate the incident response protocol (`incident-response-runbook.md`)
- [ ] Capture full context from the advisor session archive
- [ ] Notify the grower formally of findings
- [ ] Update the known-issue log if the incident surfaced a defect

## 11. Checklist Summary

A grower is onboarded when every item in sections 3, 4, and 5 is complete. Bootstrapping a new grower should not skip steps to save time — skipping always costs more later.

## Related Files

- Planning doc Section 8.7 — Phase 5 Invited-Beta Onboarding
- Planning doc Section 5 — Label Ingestion Workflow (catalogue readiness gate)
- Planning doc Section 7 — Liability and Disclaimer Approach
- Incident response runbook — `incident-response-runbook.md`
- Release checklist — `release-checklist.md`

---

> **End of onboarding runbook.** Walk through for every invited grower.

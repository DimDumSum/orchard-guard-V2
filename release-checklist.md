# OrchardGuard V2 — Release Checklist

> **Purpose.** Gate between "code merged to main" and "code running in production." Enforces planning doc Sections 6.7 (model gate) and 7 (liability posture) at deploy time.
> **Status.** Initial draft. Expect refinement in Phase 1 as CI/CD pipeline is wired up.
> **Who runs this.** The author, before promoting any change from staging to production.

---

## When to Use

Every production deploy. Not every staging deploy — staging runs automatically on merge to `main`. Production deploys happen on tagged releases after this checklist is walked.

Checklist is either:

- **Paper-walked** — author goes through manually, commits a signed-off checklist file to the release PR
- **CI-enforced** — once tooling catches up, most items become automated checks that fail the deploy if unmet

V2 launches with paper-walked; automation lands as the project matures.

---

## Pre-Deploy Checklist

### Code and tests

- [ ] All CI checks passing on the release commit
- [ ] Typecheck clean (`pnpm typecheck`)
- [ ] Lint clean (`pnpm lint`)
- [ ] Unit tests passing (`pnpm test`)
- [ ] Fixture tests passing (`pnpm test:fixtures`)
- [ ] E2E tests passing (`pnpm test:e2e`)
- [ ] No skipped tests without documented reason

### Model correctness gate (Section 6.7)

- [ ] Every Tier-1 model in the release has `advisor_enabled = true` in the registry
- [ ] Every Tier-1 model's `advisor_enabled = true` corresponds to an existing `replay/<slug>_<year>.md` artifact committed to the repo
- [ ] No Tier-1 model's citation header was modified without a corresponding replay re-run (or explicit "no algorithmic change" justification in the PR)
- [ ] Every model's `last_reviewed_against_source` is within 365 days; if not, the model is either scheduled for re-review or explicitly disabled

### Catalogue gate (Section 5.8)

- [ ] Every product that shipped as published in this release has `ontario_registered = true` and `label_ingested_at` set
- [ ] No product in the advisor candidate set has `label_ingested_at` older than 365 days without being scheduled for re-verification
- [ ] `catalogue_ingestion_runs` has no untriaged entries older than 14 days
- [ ] De-registered products from recent PMRA quarterly check are flagged correctly

### Rule engine

- [ ] All rules in `lib/rules/registry.ts` have passing unit tests
- [ ] Every hard-severity rule has citation populated
- [ ] No rules were disabled without documented reason
- [ ] Rule additions or modifications covered by tests that exercise the change

### Schema and data

- [ ] Any schema migrations in this release were reviewed; generated SQL is in `db/migrations/`
- [ ] Migration dry-run against staging succeeded
- [ ] RLS policies on new tenant-scoped tables tested (isolation confirmed)
- [ ] No schema changes that break existing `model_runs`, `trigger`, or `applications` history
- [ ] Any data migrations required by schema changes are idempotent and re-runnable

### User-facing disclosures (Section 7.4)

- [ ] Every recommendation surface renders the non-dismissible disclaimer
- [ ] Every recommendation card shows source citations (model + product)
- [ ] Every recommendation card shows the rule-evaluation summary
- [ ] ToS version on the hosted ToS page matches the version stored in `user_acknowledgments` for new invitations
- [ ] Per-recommendation parameters version is displayed

### Liability and legal

- [ ] ToS document at stable URL is current
- [ ] No material ToS changes in this release that should have triggered re-acknowledgment but didn't
- [ ] If material ToS changes, re-acknowledgment flow is tested and gates writes for unacknowledged users
- [ ] Acknowledgment text in the invitation flow matches the current ToS version

### Observability

- [ ] Sentry DSN is set in production env vars
- [ ] Structured-log transport to Axiom/Better Stack is working
- [ ] Alerting rules for deploy-time smoke tests are in place (e.g., "no advisor session completed in 1 hour" warns)
- [ ] Uptime monitor for production URL is active

### Operational readiness

- [ ] Rollback plan documented in the release PR (how to revert if something goes wrong after deploy)
- [ ] Database backup taken immediately before deploy
- [ ] Release notes drafted (summary of what's shipping, what changed, known issues)
- [ ] Affected invited growers notified if the release impacts their workflow

### External integrations

- [ ] Resend API key valid and sending limit not exhausted
- [ ] Twilio credentials valid (if SMS alerts enabled)
- [ ] Cloudflare R2 bucket accessible; signed URLs functioning
- [ ] Clerk production instance configured with correct webhooks
- [ ] Weather API sources (Open-Meteo, Env Canada) reachable

### Post-V2 checks (flagged to remind us when they become relevant)

- [ ] (2027) Weather station polling is working if station hardware is active
- [ ] (Post-launch) Discrepancy reports queue is within the 7-day SLA

---

## Deploy

Once the checklist is clean:

1. Tag the release in git: `git tag vYYYY.MM.DD.N && git push origin vYYYY.MM.DD.N`
2. Trigger production deploy (or let tag-triggered CD handle it)
3. Monitor Sentry for 15 minutes post-deploy
4. Smoke-test the critical paths: login, advisor invocation, application logging
5. Update the release log (`docs/releases.md`) with what shipped

## Rollback

If issues surface within the monitoring window:

1. Railway: roll back to previous deployment from the Railway dashboard
2. Git: revert the release commit on main if the code caused the issue
3. Database: if a migration was part of the release and must also roll back, apply the down-migration; if no down-migration exists, restore from the pre-deploy backup

Post-rollback: write an incident note in `docs/incidents/` describing what happened and what changed.

## After-Deploy Checklist

- [ ] Production is returning successful responses
- [ ] No Sentry errors above baseline in the first hour
- [ ] Smoke-tested a real advisor session end-to-end
- [ ] Release notes published to the release log
- [ ] Any affected users notified that the release shipped
- [ ] Known-issue log updated if anything new emerged

---

## Related Files

- Planning doc Section 6.7 — The Gate in Practice
- Planning doc Section 7 — Liability and Disclaimer Approach
- Dev environment setup — for local testing before release
- Incident response runbook — `incident-response-runbook.md`

---

> **End of release checklist.** Walk before every production deploy.

# OMAFRA Outreach — Draft Disclosure Letter

> **Purpose.** A draft letter to OMAFRA disclosing OrchardGuard V2's use of publicly-available Crop Protection Hub data, offering to discuss preferred mechanisms for structured access, and committing to stop scraping if requested. Referenced by planning doc Section 5.4.
> **Status.** Draft. Send when V2 is in invited-beta and has something concrete to show (post-Phase 4).
> **Audience.** OMAFRA's Tree Fruit extension program leadership, or the technical contact for the Crop Protection Hub if one is publicly identified.

---

## Send Timing

Per planning doc Section 5.15 #3: **after** invited-beta launch, not before. Rationale: OMAFRA is more likely to engage with a tool that exists and has shown value than with a speculative planning exercise. Also: reaching out while still in-development means either stopping work indefinitely while waiting for response, or ignoring the response if it comes slowly. Better to send once there's a working product to discuss.

## Channel

Email is appropriate. OMAFRA publishes contact information for the Agricultural Information Contact Centre (AICC) and for specific tree fruit extension staff. Send to the most specific relevant contact; cc AICC for routing.

If email goes unanswered for 30 days, follow up once. If still unanswered, continue operating under the politeness-norms scraping posture documented in planning doc Section 5.4 and retry contact in 6 months.

## Draft Letter

```
Subject: OrchardGuard — a decision-support tool for certified Ontario apple
         growers that references OMAFRA Crop Protection Hub

Dear <OMAFRA Tree Fruit Contact>,

I'm writing to introduce OrchardGuard, a decision-support application I've
built for my own Ontario apple orchard and a small number of certified
grower colleagues in invited beta. The tool helps us integrate weather-
driven disease and pest risk models, product label data, and OMAFRA
guidance into timely recommendations for each block of our orchards.

I wanted to disclose our use of publicly-available OMAFRA data and invite a
conversation about whether OMAFRA has a preferred mechanism for structured
access.

## What OrchardGuard Does

OrchardGuard is a decision-support tool, not a prescription service. It
runs published algorithms (Mills Table for apple scab, CougarBlight and
MaryBlyt for fire blight, degree-day accumulations for codling moth and
other pests) on local weather data, checks product labels and resistance-
management guidelines, and surfaces recommendations that the grower
verifies against authoritative sources before acting.

Every recommendation carries a citation and a non-dismissible disclaimer
that the grower is the legal applicator. Users are all certified under the
Ontario Grower Pesticide Safety Certificate or the Farmer Pesticide
License.

## How We Reference OMAFRA

We use two OMAFRA sources:

1. **Crop Protection Hub.** We fetch product recommendation and efficacy
   data to accelerate manual entry into our product catalogue. Every
   field that governs a recommendation (PHI, REI, seasonal maximums,
   tank-mix restrictions) is subsequently verified manually against the
   authoritative product label on the PMRA Public Registry before the
   product is marked recommendation-eligible.

2. **Publication 360.** We reference the fruit production recommendations,
   particularly the nutrition sufficiency ranges for leaf and soil
   analysis, in our nutrition advisor.

Our fetches from the Crop Protection Hub are rate-limited (no more than
one request every two seconds), identify OrchardGuard in the user-agent
header with contact information, respect robots.txt and any access
controls, and run as scheduled batches (weekly during growing season,
monthly off-season) rather than continuously.

We do not redistribute bulk OMAFRA data beyond our own tool; we reference
it and link users back to the authoritative pages.

## What We'd Like to Ask

1. **Does OMAFRA have a preferred mechanism for structured access?** If a
   data feed, API, or file export exists that we should use instead of
   scraping, we would prefer that. We would rather fetch data the way
   OMAFRA wants it fetched.

2. **Is our current approach acceptable?** If our scraping creates load
   concerns or is otherwise problematic, we will adjust or stop. A
   conversation would be welcome.

3. **Are there ways we can give back?** We capture data on how Ontario
   growers actually use product recommendations (anonymized, aggregate).
   If that kind of data would be useful to OMAFRA's extension work, we
   would be happy to discuss sharing arrangements with appropriate
   anonymization and user consent.

## What OrchardGuard Is Not

For full transparency:

- We are not a commercial SaaS. V2 is an invited-beta tool for a small
  group of known growers. We do not charge for access.
- We do not replace professional extension or agronomy advice. Our
  recommendations cite OMAFRA as a source for growers to verify against.
- We do not claim OMAFRA endorsement. If our use of OMAFRA references
  creates any impression of endorsement, please let us know and we will
  clarify on our site.
- We do not currently support organic production, non-apple crops, or
  users outside Ontario.

I would welcome any questions, feedback, or direction. I am available by
email at [author email] and by phone at [author phone]. I am also happy
to share access to the tool or to walk through it with OMAFRA staff if
that would be useful.

Thank you for the work OMAFRA does on behalf of Ontario fruit growers.
The Crop Protection Hub and Publication 360 are genuinely valuable
resources; I want to make sure our use of them is respectful and aligned
with OMAFRA's goals.

Sincerely,

[Author name]
OrchardGuard
[project email]
[author phone]
[project website / GitHub if public]
```

## Things to Customize Before Sending

- [ ] Confirm OMAFRA contact name or route to AICC
- [ ] Update [author email] to the project contact address
- [ ] Update [author phone] if the author wants a phone number on file
- [ ] Update [project website / GitHub if public] to whatever is appropriate
- [ ] Confirm the ToS URL is hosted and could be linked if asked
- [ ] Confirm the model correctness posture page is hosted and could be linked if asked

## Follow-Up Actions After Send

- [ ] Record the send date in the known-issue log or a dedicated outreach log
- [ ] Calendar reminder for 30-day follow-up
- [ ] If OMAFRA responds with a preferred data access mechanism, switch to it
- [ ] If OMAFRA requests we stop scraping, pause scraping immediately and lean on manual catalogue work while alternative is arranged
- [ ] If OMAFRA has no preference, continue operating under current posture and revisit annually

## Related Files

- Planning doc Section 5.4 — Scraping OMAFRA
- Planning doc Section 5.14 — Decisions (scraping posture)
- Planning doc Section 5.15 #3 — OMAFRA outreach timing open question

---

> **End of outreach letter draft.** Send once V2 is in invited-beta.

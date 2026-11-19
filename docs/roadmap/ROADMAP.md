# Roadmap Q1–Q2 2027 — Direct Digital Channel

## 1. Prioritization method

We use **WSJF** (weighted shortest job first): cost of delay ÷ job size. Cost of delay = business value + time criticality + risk reduction, each from 1 to 10. Job size uses the team estimates (person-weeks).

| Item | Business value | Time criticality | Risk reduction | Cost of delay | Size (pw) | WSJF |
|---|---|---|---|---|---|---|
| BR-01 Home Essentials (built once) | 10 | 9 | 3 | 22 | 50 | 0.44 |
| TD-02 Direct issuance in Mexico | 3 | 6 | 8 | 17 | 5 | 3.4 |
| TD-06 Nimbus deployment credential | 2 | 5 | 9 | 16 | 4 | 4.0 |
| TD-05 Contract tests | 4 | 6 | 6 | 16 | 3.5 | 4.6 |
| TD-10 Feature flags | 5 | 7 | 5 | 17 | 2.5 | 6.8 |
| TD-07 SLOs and alerts | 4 | 6 | 7 | 17 | 2.5 | 6.8 |
| TD-01 Consolidate Process APIs | 6 | 5 | 4 | 15 | 9 | 1.7 |
| TD-03 One BFF | 6 | 6 | 3 | 15 | 7 | 2.1 |
| TD-04 Mexican frontend | 5 | 4 | 5 | 14 | 10 | 1.4 |

## 2. Capacity

| Source | Q1 | Q2 | Total |
|---|---|---|---|
| Internal (6 developers × 12 weeks × 0.8) | 58 pw | 58 pw | 116 pw |
| Nimbus (4 × 12), Mexico only today | 48 pw | 48 pw | 96 pw |

We will ask that the renewal of 31 January allows Nimbus to work on the shared codebase (`channel-bff`, `web-quote-ui`, `home-quote-papi`) for the Mexican deployment, under our standards. Without that change, Nimbus capacity cannot be used for Home Essentials.

## 3. Plan

| Period | Internal team | Nimbus |
|---|---|---|
| Q1, sprints 1–2 | INC-0907 actions: `quote-xapi` per country, timeouts and circuit breakers (4 pw); SLO alerts TD-07 (2.5 pw); TD-11 secrets in Chile (0.5 pw) | Restore cache and fix retries in `quote-papi-mx` (1 pw); TD-06 move to Meridian pipelines (4 pw) |
| Q1, sprints 3–6 | BR-01: `home-quote-papi`, Home Essentials in `channel-bff` and `web-quote-ui` with country configuration (30 pw); TD-05 contract tests (3.5 pw); TD-10 feature flags per country (2.5 pw) | TD-02 Mexican issuance through `sf-policy-sapi-mx` (5 pw); from February, Mexican configuration and UAT of Home Essentials (10 pw) |
| Q2, sprints 7–8 | BR-01 hardening, Mexican launch support (10 pw) | Mexican launch support (8 pw); TD-12 upgrade (3.5 pw) |
| Q2, sprints 9–12 | BR-01 Colombia and Chile (10 pw); TD-01 consolidation, first part (9 pw); TD-08 retention (1.5 pw) | Start moving the Mexican auto journey to `channel-bff` (TD-03) |

Internal total ≈ 73.5 pw of 116 pw; the rest covers estimate errors and the BR-02 spike (see `FEASIBILITY_GENAI.md`).

**Milestones:** Mexico 30 April, Colombia 31 May, Chile 30 June 2027 — each launch behind a per-country flag, never in the last three days of the month.

## 4. What we will not do in this period

- Move all Mexican journeys out of `mx-quote-bff`: only Home Essentials starts on the shared stack; the rest is H2.
- Migrate the whole Mexican frontend (TD-04): Home Essentials screens are built in `web-quote-ui`; the old Mexican screens stay until H2.
- Launch the GenAI assistant to customers (see `FEASIBILITY_GENAI.md`).
- Run TD-09 as a project: the coverage gate on new code handles it.

## 5. Risks

| Risk | Mitigation |
|---|---|
| The renewal does not include the shared codebase | Escalate to the Director in December; fallback: the internal team builds the Mexican parts and Colombia moves to May–June |
| Product configuration in the three orgs arrives late | Agree dates with the Salesforce platform team in December; Mexico first |
| Mexico launches first with the least mature stack | Launch checklist, per-country flag, load test at 3× before launch |
| Month-end peaks during launches | No launches in the last three days of the month |

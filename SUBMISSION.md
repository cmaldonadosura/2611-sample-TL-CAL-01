# Submission

| Item | Value |
|---|---|
| Name | TL-CAL-01 (calibration sample) |
| Date submitted | 2026-11-30 |

## Summary

I propose one codebase per component deployed once per country, so that Home Essentials is built once and one country cannot take down the others, which is what happened in INC-0907. Salesforce is reached only through the API-led layers, with a daily rating-table cache that keeps the calls per quote around 1.3. The incident was caused by release 3.8.0 of `quote-papi-mx` and amplified by the shared `quote-xapi`; the first sprints of 2027 fix that before the launches. The roadmap launches Home Essentials in the three dates requested and depends on widening Nimbus's scope in the renewal. For the GenAI assistant I recommend an advisor-facing spike, not a customer launch in Q1.

## Time spent

| Part | Time |
|---|---|
| Reading the material | 30 min |
| A — Architecture, standards and governance | 2 h 15 min |
| B — Quality and automation | 50 min |
| C — Reliability and operations | 1 h 30 min |
| D — Pull request review and mentoring | 1 h |
| E — Roadmap and feasibility | 1 h 10 min |
| F — Responsible AI | 25 min |
| **Total** | **7 h 40 min** |

## The three decisions I am most confident about

1. Deploying each component once per country (ADR-0001).
2. The rating-table cache in the Process API instead of reading tables on every quote (ADR-0002).
3. Not launching the GenAI assistant to customers in Q1.

## The decision I am least sure about

Launching Mexico first. It is the date the business wants, but Mexico has the least mature stack and depends on the Nimbus renewal. If the renewal is late, I would propose to launch Colombia first.

## What I would do with more time

Make the pipeline progressive by country with automatic rollback, and write the API standards in more detail.

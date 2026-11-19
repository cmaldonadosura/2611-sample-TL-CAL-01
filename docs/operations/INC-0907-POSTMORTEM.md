# Postmortem: INC-0907 — Quote journey degraded in Mexico, Colombia and Chile

**Date:** 30 September 2026 | **Duration:** 15:25–17:20 UTC (≈ 2 h) for quotes; payments in Mexico delayed until 18:20 | **Severity:** SEV2 | **Status:** Draft

## Summary

A release of `quote-papi-mx` deployed the night before month-end removed the rating-table cache and added retries. During the Mexican peak the extra Salesforce calls slowed down the MX org, the slow Mexican requests filled the shared `quote-xapi`, and Colombian and Chilean requests were rejected even though their back ends were healthy. Rolling back the release solved the problem.

## Triage — how it should have been done

- **Severity:** SEV2, not SEV3. Quoting, the main feature of the channel, was degraded for many users in three countries, and payment processing in Mexico was affected. It was not SEV1 because the service was not down for everyone.
- **Impact:** ≈ 9,400 failed quote requests in Mexico, ≈ 1,500 in Colombia and ≈ 700 in Chile (`INC-0907.md`), 2,310 contact center calls, and Mexican payment notifications delayed from 16:55 to 18:20.
- **Roles:** an incident commander who is not the person investigating (here the on-call engineer did both until 16:18); Nimbus, the Salesforce platform team and Payments should have been called at the start, not when the problem was already in their area.

## Timeline (UTC)

| Time | Event | Evidence |
|---|---|---|
| 29 Sep 23:40 | `quote-papi-mx` 3.8.0 deployed: cache disabled, 4 attempts without backoff | `TIMELINE.md`; `quote-papi-mx.log` first line |
| 30 Sep 14:00 | Mexican calls per quote already ≈ 8.3 (normal ≈ 1.3), but low traffic hides it | `channel-metrics.csv` 14:00 MX: 5,021 calls for 605 requests |
| 15:00 | Mexican month-end peak starts; `pg-channel-co` report starts | `channel-metrics.csv`; `TIMELINE.md` |
| 15:15–15:20 | MX channel above 40 req/s (44, then 52); Salesforce p95 rises from ≈ 1 s to 2.6 s | `channel-metrics.csv` |
| 15:25 | First Mexican 504s | `quote-xapi.log` 15:27:09 |
| 15:35 | `quote-xapi` in-flight limit reached; Mexico holds 93% of the slots; CO and CL start to queue | `shared-metrics.csv` 15:35; `quote-xapi.log` 15:34:51 |
| 15:38 | First `QUEUE_FULL` rejections for CO and CL | `quote-xapi.log` 15:38:02 |
| 15:41 | Colombian contact center reports | `incident-channel.md` |
| 15:52 | SEV3 declared | `incident-channel.md` |
| 16:00–16:05 | Key rotation: 401s for five minutes | `quote-xapi.log` 16:00–16:05 |
| 16:20 | `quote-xapi` scaled from 2 to 4 replicas | `shared-metrics.csv` |
| 16:40 | Errors back in CO and CL; MX worse | `channel-metrics.csv` |
| 16:55 | MX org at 85%; Payments pauses Mexican consumers | `incident-channel.md` |
| 17:05 | Rollback to 3.7.2 | `quote-papi-mx.log` 17:05 |
| 17:20 | Mitigated | `channel-metrics.csv` |

## Root cause

1. **Release 3.8.0** disabled the rating-table cache, so every Mexican quote read six tables from Salesforce: ≈ 8 calls per quote instead of ≈ 1.3 (`quote-papi-mx.log` 14:05: `sfCalls=8`; `sf-api-usage-daily.csv`: 6.24 calls per quote on 30 September against 1.3 the previous days).
2. At 3× traffic, the Mexican channel went above the 40 req/s guidance and Salesforce slowed down. The **retries without backoff** multiplied the calls (≈ 13 per quote from 15:40), which slowed Salesforce more: a retry storm (`quote-papi-mx.log` 15:35:37–15:36:08).
3. `quote-xapi` is **one shared pool** for the three countries with a 30 s timeout. Slow Mexican requests took 91–96% of the in-flight slots (`shared-metrics.csv`), so Colombian and Chilean requests were queued or rejected, while `quote-papi-co` and `quote-papi-cl` answered in ≈ 600–700 ms the whole time (`channel-metrics.csv`; `quote-xapi.log` 15:36:18: `queuedMs=11231 upstreamMs=598`).
4. **Scaling out made it worse:** more replicas let more Mexican requests in, Salesforce received ≈ 95 req/s, and the new capacity filled again in 15 minutes (`shared-metrics.csv` 16:20–16:40).
5. The extra calls consumed the Mexican budget (250,000 per 24 h) around 16:00, and the org allocation reached 85% at 16:55, so Payments paused (`incident-channel.md` 16:55, 16:58).

### Hypotheses ruled out

| Hypothesis | Why not |
|---|---|
| `pg-channel-co` CPU | The report runs every month-end; application queries stayed at 37–44 ms (`pg-channel-co.log`); errors continued after the CPU went down at 16:00 |
| Key rotation | Only 401s, 0.41% for five minutes (`shared-metrics.csv` 16:00); the incident started at 15:25 with 504 and 503 |
| "Only month-end traffic" | With 3.7.2 the peak would have been ≈ 8 req/s (1,900 quotes per 5 min × 1.3 ÷ 300) |

## 5 whys

1. Why did quotes fail in three countries? → The shared `quote-xapi` was saturated by Mexican requests.
2. Why did Mexican requests take so long? → Salesforce was slow and each request retried many calls.
3. Why were there so many calls? → 3.8.0 removed the cache and added retries without backoff.
4. Why did 3.8.0 reach production before month-end? → Nimbus releases are not reviewed by the cell, there is no performance test at peak volume and no canary.
5. Why could one country take down the others? → `quote-xapi` was designed as one shared pool, without isolation per country.

## What went well

- The rollback was fast once decided (5 minutes).
- The DBA and the identity team answered quickly and ruled out their components.

## What went poorly

- Recent changes were not checked first: 3.8.0 was in the deployment notifications but was considered only at 17:00.
- Scaling out increased the load on Salesforce.
- Severity was set too low and nobody coordinated with Payments until they had to pause.
- No alert fired; the contact center detected the incident.

## Action items

| Action | Owner | Priority |
|---|---|---|
| Restore the rating-table cache in `quote-papi-mx`, refreshed daily | Nimbus | P0 |
| Retries with exponential backoff and jitter, maximum one retry | Nimbus; cell standard | P0 |
| Deploy `quote-xapi` once per country; timeout to the Process API ≤ 3 s; circuit breakers | Technical Lead, integration developer | P0 |
| SLO alerts per country (see `SLOS.md`) | Cell | P1 |
| Nimbus releases through Meridian pipelines, with release notes reviewed by the Technical Lead and a load test at 3× | Technical Lead, vendor manager | P1 |
| Incident runbook: commander role, check recent changes first, do not scale out before understanding the bottleneck | Channel manager | P1 |
| Agree consumption limits for the MX org with Payments and the Salesforce platform team | Technical Lead | P2 |

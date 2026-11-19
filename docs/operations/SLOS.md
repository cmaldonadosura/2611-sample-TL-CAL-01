# SLIs and SLOs — Direct Digital Channel

## 1. Journeys, SLIs and SLOs

The SLOs apply **per country** (MX, CO and CL each have their own budget) over a rolling 28-day window.

| Journey | SLI | Measured at | SLO |
|---|---|---|---|
| Get a quote — availability | Valid quote requests answered with a non-5xx response ÷ all valid quote requests | `channel-bff`, per country (server-side), plus synthetic checks every minute from Azure Monitor | 99.5% |
| Get a quote — latency | Quote requests answered in under 2.5 s ÷ all quote requests | `channel-bff`, per country | 95% |
| Buy and issue | Purchases with the policy issued and the documents sent within 3 minutes ÷ all purchases | Event `policy.issued` minus event `purchase.accepted`, per country | 99% |

"Valid" excludes 4xx caused by the client (validation errors, expired sessions).

## 2. Error budget policy

| Budget consumed (28 days) | What the cell does |
|---|---|
| < 50% | Normal delivery |
| 50–100% | The Product Owner and the Technical Lead review the top causes in the next sprint planning; at least 20% of the sprint goes to reliability work for that country |
| > 100% | Feature releases for that country are frozen until the budget recovers; only fixes and reliability work are deployed; a postmortem is required for every incident that consumed more than 10% of the budget |

## 3. Alerting

| Alert | Condition | Action |
|---|---|---|
| Quote errors | 5xx rate > 5% for 5 minutes in a country | Page the on-call engineer |
| Quote latency | p95 > 2.5 s for 10 minutes in a country | Page the on-call engineer |
| Issuance delay | Any purchase without policy after 10 minutes | Ticket to the cell, page if more than 20 in an hour |
| CPU and memory | > 85% for 15 minutes | Ticket |

## 4. Safe deployments

- Blue/green deployments with deployment slots in App Service and revisions in Container Apps.
- After the switch, the developer validates the main journeys manually for 15 minutes; if something fails, we switch back to the previous slot.
- Changes are deployed outside the last three days of the month.
- Nimbus should follow the same strategy for the Mexican components.

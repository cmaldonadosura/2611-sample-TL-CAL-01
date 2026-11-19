# ADR-0002: How quoting and issuance reach Salesforce

| Item | Value |
|---|---|
| Status | Proposed |
| Date | 2026-11-30 |
| Deciders | Technical Lead (Direct Digital Channel), Salesforce platform team, Payments Integration Technical Lead |
| Related | C2, C4, C6; `SALESFORCE_CONSTRAINTS.md`; INC-0907; ADR-0001 |

## Context

Rating runs in Salesforce. Each org is available ≈ 97% of the time, asks for less than 40 requests per second per integration user, and has a daily allocation shared with Payments, whose consumers pause at 85% of the org allocation. In INC-0907, reading the rating tables on every quote multiplied the calls per quote by six and exhausted the Mexican budget. The business accepts tables up to 24 hours old if the version is shown.

## Decision drivers

1. Respect the API budget and the org allocation (C6).
2. Latency at month-end peaks (C2).
3. Integration through the platform (C4).
4. Keep rating logic in one place.

## Options considered

### Option A — `channel-bff` calls Salesforce directly (C4 exception)

| Dimension | Assessment |
|---|---|
| Complexity | Low at first |
| Cost | Low |
| Availability and isolation | No API Manager policies (rate limiting, spike control) |
| Security and privacy | Salesforce credentials in the channel |
| Fit with team and vendor | Fast to build, but repeats TD-02 |

### Option B — API-led path with a rating-table cache in the Process API

| Dimension | Assessment |
|---|---|
| Complexity | Medium |
| Cost | Medium |
| Availability and isolation | API Manager policies apply; ≈ 1.3 calls per quote |
| Security and privacy | Credentials only in the System API |
| Fit with team and vendor | Needs MuleSoft work, our bottleneck |

### Option C — Rating engine in Azure using the published tables

| Dimension | Assessment |
|---|---|
| Complexity | High: duplicates rating logic outside Salesforce |
| Cost | High |
| Availability and isolation | Quotes independent of the org |
| Security and privacy | Neutral |
| Fit with team and vendor | Needs agreement with the Salesforce platform team; risk of two sources of truth for prices |

## Decision

Option B. Rating goes `quote-xapi` → `home-quote-papi` (with the daily rating-table cache) → `sf-policy-sapi-<country>`. Issuance is asynchronous through the event broker (see `ARCHITECTURE.md`). Exceptions to C4 are only allowed through an ADR approved in the architecture forum, with an expiry date and a budget.

## Trade-off analysis

Option A is the fastest way to meet the Mexican date but reproduces the conditions of INC-0907 (consumption invisible to the API Manager, no spike control). Option C would remove the dependency on Salesforce for quotes, but it creates two places where prices are calculated, which is a business risk we cannot accept without the Salesforce platform team. Option B keeps one source of truth for rating and reduces the calls per quote to ≈ 1.3.

## Consequences

- **Becomes easier:** staying within the budget; applying rate limiting and spike control in the API Manager; showing the table version.
- **Becomes harder:** more MuleSoft work for a cell with one MuleSoft developer; cache invalidation if the pricing team publishes outside 05:00 UTC.
- **To revisit:** if the platform team offers a rating service with higher availability, or if quote availability stays below target for two months.

## Action items

1. [ ] Agree the new Mexican budget with the platform team and Payments.
2. [ ] Rate-limit and spike-control policies on `sf-policy-sapi-*`.

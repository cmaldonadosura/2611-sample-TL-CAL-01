# Engineering standards — Direct Digital Channel

## 1. Test strategy

| Level | What | Where | Who writes it | When it runs |
|---|---|---|---|---|
| Unit | Business logic, mappings, country configuration | Frontends, BFF, MuleSoft (MUnit) | Developer of the change | Every commit |
| Component | Service with its dependencies mocked | BFF, Process APIs | Developer | Every pull request |
| Contract | Consumer-driven contracts (Pact) between frontends and `channel-bff`, and between `channel-bff` and `quote-xapi`; OpenAPI validation of MuleSoft APIs against the published contract | All interfaces we own | Consumer writes the contract, provider verifies | Pull request and before deployment |
| End to end | Critical journeys per country (quote, buy) | UAT | QA engineer | Before each production release |
| Load | Peak traffic tests | UAT | QA engineer with developers | Before major releases |
| Security | DAST on the frontends and the BFF | UAT | Security office tooling | Monthly and before launches |

Test data is synthetic; production data is never copied to lower environments. Nimbus follows the same strategy for the Mexican components.

## 2. Quality gates

| Gate | Threshold | Stage | If it fails |
|---|---|---|---|
| Unit tests | 100% pass | Pull request | Merge blocked |
| Coverage of new code | ≥ 80% | Pull request | Merge blocked |
| SAST | No new high or critical findings | Pull request | Merge blocked |
| Dependency scan (SCA) | No known critical vulnerabilities | Pull request and nightly | Merge blocked; nightly opens a ticket |
| Secret scanning | No secrets | Pull request (push protection) | Push rejected |
| IaC scan | No high findings | Pull request | Merge blocked |
| Contract verification | All consumer contracts pass | Before deployment | Deployment blocked |
| Performance budget | p95 of the quote endpoint ≤ 2 s in UAT load test | Before production | Release blocked |

**Exceptions:** a failed gate can be waived only by the Technical Lead, with a ticket that has an owner and an expiry date of at most 30 days.

## 3. Code review and branching

- Trunk-based development with short-lived branches (less than two days).
- Two approvals for shared components (`channel-bff`, contracts, shared configuration), one of them from a code owner; one approval for the rest.
- Pull requests that include AI-generated code must say so in the description, and the reviewer applies the same standard as for any code.

## 4. Nimbus Digital (C8)

Requirements for the renewal on 31 January 2027:

| Requirement | How we verify it |
|---|---|
| Code in Meridian repositories and deployments only through Meridian pipelines | No other deployment credential exists; pipeline history |
| Same quality gates and definition of done | Gate reports in every pipeline run |
| Coverage of new code ≥ 80%, reported per release | Pipeline report |
| Release notes reviewed by the Technical Lead before production | Approval recorded in the pipeline |
| Participation in incidents within 30 minutes during business hours | Incident records |
| Rotation of the current shared credential and removal of the Nimbus CI access | Security office check |

## 5. Delivery targets (end of Q2 2027)

| Metric | CO now → target | CL now → target | MX now → target | Reasoning |
|---|---|---|---|---|
| Deployment frequency | Weekly → twice a week | Every two weeks → weekly | Monthly → every two weeks | One step per country; MX depends on the move to Meridian pipelines |
| Lead time | 6 → 4 days | 11 → 6 days | 24 → 10 days | Shared pipeline and contract tests remove UAT rework |
| Change failure rate | 12% → 10% | 18% → 12% | 31% → 15% | Contract tests (TD-05) and feature flags (TD-10) |
| Time to restore | 3 → 2 h | 5 → 2 h | 9 → 4 h | SLO alerts (TD-07) and rollback per country |

## 6. Golden path

A template repository for new services with: a Spring Boot skeleton with the security configuration, the pipeline template, and a README.

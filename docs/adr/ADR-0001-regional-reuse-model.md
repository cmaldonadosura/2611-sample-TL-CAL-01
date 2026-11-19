# ADR-0001: Regional reuse model for the Direct Digital Channel

| Item | Value |
|---|---|
| Status | Proposed |
| Date | 2026-11-30 |
| Deciders | Technical Lead (Direct Digital Channel), Director of Digital Channels, Security office |
| Related | C1, C3, C5, C8, C9; INC-0907; ADR-0002 |

## Context

Today the quote journey exists in three variants and the only shared component, `quote-xapi`, is one deployment for the three countries. In INC-0907 the Mexican peak saturated it and Colombia and Chile failed although their back ends were healthy. BR-01 must be built once (C3) and launched in three countries in Q2 2027. Legal requires separation of personal data between countries (C5), and Nimbus maintains Mexico until at least the renewal (C8).

## Decision drivers

1. Isolation between countries (C1).
2. Build once, configure per country (C3).
3. Separation of personal data (C5).
4. A model that Nimbus can work in (C8).
5. Cost (C9).

## Options considered

### Option A — One regional deployment with country configuration

| Dimension | Assessment |
|---|---|
| Complexity | Low: one deployment per component |
| Cost | Lowest |
| Availability and isolation | Weak: a problem in one country affects all (INC-0907) unless strong bulkheads are added |
| Security and privacy | Data of the three countries handled by the same runtime |
| Fit with team and vendor | Nimbus would deploy a component that also serves Colombia and Chile |

**Pros:** cheap, simple releases. **Cons:** blast radius, harder data separation, one release affects three countries.

### Option B — One codebase, one deployment per country ("deployment stamps")

| Dimension | Assessment |
|---|---|
| Complexity | Medium: three deployments per component, same pipeline |
| Cost | Higher than A |
| Availability and isolation | Strong: each country has its own runtime |
| Security and privacy | Each country's runtime only accesses its own org and database |
| Fit with team and vendor | Nimbus can deploy the Mexican stamp without touching CO and CL |

**Pros:** isolation, data separation, independent releases per country. **Cons:** more infrastructure and configuration to manage; risk of configuration drift.

### Option C — Keep one codebase per country

**Pros:** no migration. **Cons:** contradicts C3; this is the current problem (TD-01, TD-03).

## Decision

Option B: one codebase per component, deployed once per country, with country differences in configuration and in Salesforce product rules.

## Trade-off analysis

Option A is cheaper, but INC-0907 showed what a shared runtime costs us, and bulkheads inside one deployment are harder to get right than separate deployments. Option C does not meet C3. Option B costs more in infrastructure, but it gives isolation and data separation by construction, and it lets Mexico move to the shared code while Nimbus keeps deploying only Mexico.

## Consequences

- **Becomes easier:** isolating incidents; releasing a change to one country first; meeting C5.
- **Becomes harder:** operating three deployments of each component; keeping configuration consistent; higher compute cost.

## Action items

1. [ ] Pipeline template with one stage per country (Part B).
2. [ ] Configuration schema validated at startup.
3. [ ] Split `quote-xapi` into three deployments (INC-0907 action).

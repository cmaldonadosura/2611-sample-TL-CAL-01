# Feasibility — BR-02 GenAI assistant in the quote journey

## 1. Recommendation

**No to a customer launch in Mexico in Q1 2027. Yes to a five-week spike with an internal, advisor-facing assistant, and a go/no-go decision at the end of March.** If the spike passes its criteria and Compliance and Security approve, a limited customer pilot can start in H2 2027.

## 2. Value — what the evidence says and does not say

| Evidence | What it supports | Limits |
|---|---|---|
| Abandonment 68% MX, 61% CO, 64% CL | There is room to improve | Not specific to coverage questions |
| 41% of ≈ 2,300 survey respondents did not understand coverages | Coverage understanding is a real driver | Self-selected sample of people who abandoned; we do not know if an assistant changes their decision |
| ≈ 18,000 calls per month about coverages × USD 4.10 | **Upper bound** of ≈ USD 74,000 per month if every call disappeared | Real deflection is unknown; typical deflection is a fraction of this |

**Missing information:** expected deflection and conversion uplift; Compliance's definition of "advice" in each country; the quality and structure of the PDF wordings; Security's path and time to approve customer-facing use; cost per conversation; differences of Spanish between countries.

## 3. Effort

- Customer-facing assistant with quote pre-filling: retrieval over the wordings, the assistant service, integration with the quote journey, evaluation, guardrails, monitoring, three countries → in my estimate 20–30 person-weeks, competing directly with BR-01.
- Spike as proposed below: ≈ 5 person-weeks of developer capacity plus ≈ 3 of the QA engineer, and four hours per week of a product expert.

## 4. Options

| Option | Value | Effort | Risk | Assessment |
|---|---|---|---|---|
| A. Customer assistant in Mexico in Q1 | High if it works | 20–30 pw | High: no customer-facing approval, regulated advice, hallucinated coverages | Not responsible now |
| B. Advisor-facing assistant (internal pilot) | Medium: faster, more consistent answers for advisors; learning for option A | ≈ 8 pw | Low–medium: allowed as internal pilot; advisors filter the answers | **Recommended as the spike** |
| C. Improve coverage explanations in Home Essentials (comparisons, plain language, FAQ) without an LLM | Medium | Small, inside BR-01 | Low | **Recommended in parallel** |
| D. Do nothing now | None | None | Lose the learning | Not recommended |

## 5. Spike (option B)

- **Scope:** Mexico, Home Essentials and auto wordings only; retrieval over the official PDFs; answers with citations to the clause; advisors only; no access to customer data; no pre-filling.
- **Time box:** five weeks, in March 2027, with one back-end developer and the QA engineer, plus four hours per week of a product expert.
- **Evaluation set:** 300 real questions from contact center logs (anonymized), with expected answers written by product experts, plus 100 red-team prompts that try to obtain a recommendation, other customers' data or answers outside the wordings.

| Go criterion | Threshold |
|---|---|
| Correct and cited answers on the evaluation set | ≥ 95% |
| Answers that recommend a coverage, in the red-team set | 0 |
| Personal data in prompts, answers or logs | 0 |
| p95 latency | ≤ 4 s |
| Cost per conversation | ≤ USD 0.05 |
| Advisors who would use it daily (survey, ≥ 20 advisors) | ≥ 70% |
| Written opinion of Compliance and Security on a customer pilot | Received |

Missing any threshold → **no-go** for customers; we report what failed and whether a second iteration makes sense.

## 6. Risks of a customer-facing LLM feature and required controls

| Risk (OWASP Top 10 for LLM applications, 2025) | Control |
|---|---|
| Prompt injection (LLM01) | Retrieval only over curated documents; the model has no tools with side effects; system prompt and user input separated |
| Sensitive information disclosure (LLM02) | No personal data sent to the model; masking before logging; data processing agreement and region of the LLM service checked |
| Excessive agency (LLM06) | Pre-filling the quote only proposes values; the customer confirms each field |
| Misinformation about coverages (LLM09) | Answers only from retrieved clauses with citations; "I don't know" outside the corpus; disclaimers; continuous evaluation with the set above |
| Unbounded consumption (LLM10) | Quotas per session and per day, rate limiting at the API Manager, cost alerts |
| Regulated advice | Instructions and output filters against recommendations; red-team tests per country; Compliance approval |

## 7. Effect on BR-01

The spike uses ≈ 5 pw of the ≈ 42 pw of internal developer slack in the roadmap and starts after the Home Essentials Process API is done, so it does not move the Mexican launch.

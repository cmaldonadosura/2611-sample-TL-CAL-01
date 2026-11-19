# Review of PR-388 — MX: Home Essentials quick quote

## 1. Findings

Paths: `home/` = `src/main/java/com/meridian/channel/home/`, `integration/` = `src/main/java/com/meridian/channel/integration/`.

| # | File | Line | Issue | Severity | Requested change |
|---|---|---|---|---|---|
| 1 | `home/HomeQuoteController.java` | 26–27 | Country and customer come from the headers `X-Country` and `X-Customer-Id`. Any logged-in user can send another customer's ID and receive their name and address (broken object-level authorization). | Critical | Use `CustomerContext.from(jwt)` as `QuoteController` does; remove the headers from the contract. |
| 2 | `home/HomeQuoteService.java` | 24 | Logs the customer's name, national ID and email at INFO. | Critical | Remove; log only the quote ID, the country and the correlation ID. |
| 3 | `src/main/resources/application-mx.yml` | 8 | Salesforce client secret committed in the configuration. | Critical | Move the secret to Key Vault and read it with a managed identity. |
| 4 | `integration/SalesforceRatingClient.java` | 15–18, 37, 54 | Calls the MX org directly, bypassing the API Manager and the System API. Violates C4 and consumes the MX budget invisibly, as in INC-0907. | Critical | Go through a Process API and `sf-policy-sapi-mx`. If the date requires a temporary exception, it needs an ADR. |
| 5 | `integration/SalesforceRatingClient.java` | 37–40 | The account ID from the header is concatenated into the Salesforce query: SOQL injection. | High | Validate the ID format and use a parameterized query. |
| 6 | `home/HomeQuoteService.java` | 30–37, 44 | Country rules (taxes, coastal surcharge, currency) hardcoded with `if`. Chile falls into the `else` and gets COP as currency. | High | Use `CountryProperties` and the product rules of the rating service; no country conditionals in shared code. |
| 7 | `src/main/java/com/meridian/channel/quote/QuoteResponse.java`, `api/channel-quote-api.yaml` | 9; 106, 112 | `premiumAmount` renamed to `totalPremium` in a contract used by the Colombian and Chilean frontends. Breaking change; the checklist says "No breaking changes". | High | Revert the rename. If the Mexican frontend needs another name, map it in the frontend. |
| 8 | `integration/SalesforceRatingClient.java` | 22 | `new RestTemplate()` without timeouts or circuit breaker. | Medium | Use the pattern of `QuoteExperienceClient`: timeouts and `@CircuitBreaker`. |
| 9 | `integration/SalesforceRatingClient.java` | 60–71 | A new OAuth token is requested on every call. | Medium | Cache the token until it expires (or remove the client with finding 4). |
| 10 | `src/test/.../home/HomeQuoteServiceTest.java` | 15–16, 30–33 | The test mocks `HomeQuoteService` itself and asserts its own stub; it tests nothing. The PR claims 80% coverage. | Low | Test the real service with the Salesforce client mocked: one case per country, rounding and errors. |

## 2. Verdict

**Request changes.** Before merge: findings 1, 2, 3 and 4 fixed, finding 7 reverted, and real tests (finding 10).

## 3. Refactoring proposal

- Use the Home Essentials Process API instead of calling Salesforce.
- Move taxes and the surcharge to the country configuration and the product rules.
- Revert the contract change.
- Rewrite the tests.
- Remove the customer data from the response.

## 4. Feedback to the author

> Hi Diego, thanks for this PR — it is the first Mexican feature in `channel-bff`, and that is an important step for all of us. The structure is clean, the request validation is good, and the endpoint is easy to follow. I know the Mexican date is tight, so I want to help you merge it quickly and explain the *why* behind each change, since these rules apply to every feature we build together.
>
> **Identity always comes from the token.** In `channel-bff` we never trust headers or the body to say who the customer is or which country they belong to: anyone can set a header. `CustomerContext.from(jwt)` gives you both, already validated. This is the most important fix, because today a customer could read someone else's data.
>
> **Personal data and secrets never go into logs or files.** Logs are copied and kept for a long time, and the Git history keeps secrets forever. Log technical IDs only, and keep secrets in Key Vault.
>
> **Shared code must not know about countries.** `channel-bff` serves three countries. When we write `if ("MX".equals(country))`, the next country falls into the `else` — today Chile would get Colombian pesos. Country differences go to `CountryProperties` and to the product rules in Salesforce.
>
> **We integrate through the platform.** I understand the six-week backlog for the Process API. Direct calls to Salesforce hide our consumption from the API Manager, and on 30 September that is exactly what took down quoting in three countries. If we really need a temporary exception, let's write a short ADR together and agree it with the Salesforce platform team; it is not something to decide inside a PR.
>
> **Contracts are promises to other teams.** Renaming a field breaks the Colombian and Chilean screens. We change contracts only in an additive way or with a new version.
>
> **About the tests:** the AI assistant produced a test that mocks the class it is testing, so it always passes. It happens to all of us with these tools; the rule is to read a generated test and ask "what bug would this catch?". If the answer is "none", it needs to change.
>
> The identity, logging, secret and integration points are blocking; the rest we can discuss. The rules are in `ARCHITECTURE.md` §8 and `ENGINEERING_STANDARDS.md`. Shall we pair tomorrow for an hour? I think we can solve most of it together.

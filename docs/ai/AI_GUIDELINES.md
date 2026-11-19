# Guidelines for the use of AI in software development — Direct Digital Channel

## 1. Purpose

AI assistants make us faster, but the author stays accountable for everything that reaches a pull request. These guidelines say what we can do with them and what we cannot.

## 2. Allowed tools

| Tool | Allowed for | Status |
|---|---|---|
| GitHub Copilot Business | Code completion and chat in the IDE | Approved |
| Meridian's LLM service in Azure | Documentation, analysis of non-production data | Approved for internal use |
| Any other tool | — | Not allowed until approved by Security; request it through the Technical Lead |

## 3. Data

| Never share | May share |
|---|---|
| Personal data of customers or employees | Code of the cell's repositories (approved tools only) |
| Secrets, tokens, credentials, connection strings | Public documentation |
| Production logs, incident data with customer information | Synthetic test data |
| Contracts or documents marked confidential | Anonymized questions about errors |

## 4. Review and accountability

- The author is accountable for AI-generated code exactly as for code written by hand. "The assistant wrote it" is never an explanation.
- The reviewer treats AI-generated code as untrusted code.
- **Tests generated with AI must verify behavior**: they must not mock the unit under test, and each test must answer the question "which bug would this catch?". PR-388 is the example of what to avoid.
- AI-generated code passes the same quality gates as any code.

## 5. Disclosure

The pull request template asks: "Was AI used for this change? For what?". The author answers in one line.

## 6. Security and intellectual property

- Check that every dependency suggested by an assistant exists and is the official package before adding it: assistants sometimes invent package names, and attackers publish packages with those names.
- Secret scanning runs on every push, so a secret pasted from an assistant is blocked.
- Do not accept large blocks of generated code that look copied from a public project without checking the license.

## 7. Rollout

We will present these guidelines in a sprint review, add them to the onboarding of new developers, and review them every six months.

## 8. Features that call an LLM

When the cell builds a feature that calls an LLM, the controls in `docs/roadmap/FEASIBILITY_GENAI.md` (section 6) apply.

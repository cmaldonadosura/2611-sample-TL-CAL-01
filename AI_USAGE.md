# AI usage declaration

## Summary

I used Claude as a sparring partner and for first drafts, and GitHub Copilot to write YAML faster. Every decision and every figure in the submission is mine, and I reviewed all generated text before keeping it.

## Tools

| Tool (name and version or plan) | Where it ran (web, IDE, CLI) | Data you shared with it |
|---|---|---|
| Claude (claude.ai, Pro plan) | Web | Parts of the assessment material (scenario, metrics, the PR diff); no personal data |
| GitHub Copilot | VS Code | The YAML files I was writing |

## Uses

| Part | What you asked the tool to do | What you kept | How you validated it | What you discarded and why |
|---|---|---|---|---|
| A | Challenge my choice between one regional deployment and one deployment per country | Two cons of option B that I had not written | Reviewed the output and compared it with the constraints | A suggestion to use a service mesh: too complex for the team |
| A | Generate the OpenAPI skeleton | The structure | Reviewed the output | The example values |
| B | Complete the Azure Pipelines syntax | Most of the template | Reviewed the output | — |
| C | Summarize the metrics CSV around 15:00–17:30 | The idea of comparing the in-flight share by country | Reviewed the output and checked the main numbers in the CSV | A claim that Salesforce returned 403 errors: not in the evidence |
| D | List possible issues in the diff | The issues I agreed with | Reviewed the output | Two suggestions about code style |
| E | Draft the risk table for the GenAI assistant | Structure and most risks | Reviewed the output against the OWASP list | — |
| F | None | — | — | — |

## Reflection

The tools saved time on syntax and on structure. They were less useful for the incident: the first summary of the metrics mixed up the timeline, so I went back to the files.

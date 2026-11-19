# Assumptions

| ID | Assumption | Why I needed it | Impact if it is wrong |
|---|---|---|---|
| AS-01 | The Salesforce platform team can configure Home Essentials in the three orgs by March 2027 | The launch dates depend on it | Launches move |
| AS-02 | The Nimbus renewal can include work on the shared codebase | The roadmap uses Nimbus capacity for Mexico on the shared stack | The internal team must build the Mexican parts; Colombia could launch first |
| AS-03 | The event broker can be used by the Process API for issuance | Asynchronous issuance (ADR-0002) | Another queue service in Azure |
| AS-04 | Business keeps accepting rating tables up to 24 hours old | The cache in ADR-0002 | More calls per quote; budget increase needed |

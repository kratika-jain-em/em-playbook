# Incident Management in Fintech 

## Context
 
Incident management is not unique to fintech, rather every distributed system products has some version of it. What is unique to fintech is what is at stake when it goes wrong.
 
In most software systems, an incident means degraded user experience, lost revenue, or reputational damage. 
In payment systems, an incident can also mean:
 
- Customers unable to pay rent, buy food, or access their wages
- Funds in an indeterminate state, neither in the sender's account nor the recipient's
- A regulatory notification obligation triggered within hours
- A chargeback liability window opening that cannot be closed
- Settlement files not submitted before the clearing cut-off, a cash flow event, not just a technical one
 
This changes the character of incident response because the urgency, the communication obligations, & the definition of "resolved", are all different. A payment system can return `200 OK` and still be in an incident state if funds are stuck in transit, or reconciliation is broken, or if a downstream bank has not received a file it was expecting.
 
Strong incident management in fintech is not about having a good runbook. It is about building a culture and a set of systems that treat operational failure as a first-class engineering concern, before the incident happens, during it, and in the weeks that follow.
 
---

## The Problem

The specific failure modes that can surface in fintech incidents:

### Severity misclassification
A payment failure affecting 0.5% of transactions gets logged as P3 because the volume looks small. The affected 0.5% turns out to be all transactions to a specific bank, or all transactions above a certain amount, or all transactions for a single large merchant. The blast radius was not what the percentage suggested. Fintech incidents require blast radius assessment, not just volume assessment.

### Ambigous ownership
Payment incidents cross team boundaries by nature. The failure might be in the payment service, the ledger service, the PSP integration, the reconciliation pipeline, or the bank's own systems. In the first minutes of an incident, nobody knows. If ownership is not pre-agreed, the first crucial minutes are spent establishing it, minutes that cost customer money.

### Communication vaccum
Engineers are heads-down in the incident. Nobody is talking to customers, to operations, to compliance, or to the bank relationship manager who is now fielding calls. The silence gets filled by speculation and escalation from people without context.

### Incomplete resolution
The service recovers. `200 OK` returns & the incident is marked as closed. Three hours later, the reconciliation team finds a break, transactions that were retried successfully but also captured originally, resulting in double charges. The incident was not resolved; it was paused.

### No regulatory awareness
The FCA's operational resilience rules and DORA both impose notification obligations for material incidents. In some cases, the clock starts at the point the incident began, not the point it was escalated. If engineering teams do not know which incidents trigger regulatory notification, the organisation finds out at the worst possible moment.
 
---

## The Incident Lifecycle
A well-managed incident moves through six distinct phases, each with specific engineering and leadership responsibilities.


 

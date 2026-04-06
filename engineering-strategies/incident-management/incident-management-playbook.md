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

```
Detection -> Classification -> Response -> Communication -> Resolution -> Post-mortem
```
### Phase 1 - Detection
Detection should come from your systems, not from customers. If customers are reporting payment failures before your alerts fire, the observability is insufficient.

Effective detection for payment systems requires alerting on:
 
- Payment success rate dropping below threshold.
- Authorisation decline rate spike by response code
- PSP connectivity and response time
- Settlement file submission confirmation, absence of success is an alert, not a non-event
- Reconciliation break count crossing threshold
- Ledger balance assertion failures
 
```java
// Example: payment success rate alert, fires when 5-minute success rate drops below 98%
// Do not alert on percentage alone, combine with minimum volume to avoid noise on low traffic
@Component
public class PaymentSuccessRateMonitor {
 
    private static final double ALERT_THRESHOLD = 0.98;
    private static final int MIN_VOLUME_FOR_ALERT = 50;
 
    public boolean shouldAlert(PaymentMetrics metrics) {
        if (metrics.totalAttempts() < MIN_VOLUME_FOR_ALERT) {
            return false; // insufficient volume to signal
        }
        return metrics.successRate() < ALERT_THRESHOLD;
    }
}
```
 
> The on-call engineer who receives the alert needs enough context to classify the severity immediately. Alert messages should state: what is failing, how many customers are affected (estimated), and what the last known good state of the system was.

### Phase 2 - Classification
Classification determines the response. Get it wrong and you either under-respond to a serious incident or exhaust your team on a false alarm.

A practical severity framework for payment systems:
 
| Severity | Definition | Response | Notify |
|---|---|---|---|
| **P1** | Core payment flow broken, customer money at risk, or funds in indeterminate state | Immediate all-hands, leadership alerted | Compliance, legal, bank relationship, CEO |
| **P2** | Significant payment degradation, >5% failure rate, or specific cohort fully blocked | Immediate engineering response, EM alerted | EM, product, ops |
| **P3** | Partial degradation, single integration failing, <5% affected, workaround exists | On-call response within 30 minutes | On-call EM |
| **P4** | Performance degradation, non-critical feature impacted, no customer money at risk | Next business day | Engineering |

### Phase 3 - Response
For P1 and P2 incidents, response means activating a war room, a synchronous, focused environment where the right people are working the problem together. For the response phase, the most important engineering decisions are:

#### Isolate before diagnose
If you can stop the bleeding, like disable a failing integration, or route traffic away from a broken PSP, or halt a batch job that is creating bad data, or reverse to last healthy system if release issue, do it before spending time understanding root cause. Stopping the harm and understanding the cause are separate activities.

#### Explicit role assignment
In the heat of an incident, implicit role assignment fails. Somebody needs to own: the technical investigation, the customer impact assessment, the communication stream, and the timeline documentation. These should be named, not assumed.

#### Log everything in real-time
Every hypothesis, every action taken, every system state observed, should be logged. It is not just for the post-mortem, but for the next people who might join the incident bridge 20 minutes in. Catching up from a live log is faster than catching up from conversation.

```
# Incident log format — simple, timestamped, in a shared doc or incident channel
 
14:32  [Kratika]  Alert fired: payment success rate 91.2% over 5 min window
14:34  [Kratika]  Confirmed: all Barclays-issued Visa debit failing with DE39 code 91
14:36  [James]    PSP dashboard shows Barclays BIN range flagged, not our system
14:40  [Kratika]  DECISION: disable automatic retry on code 91 to prevent scheme fine exposure
14:42  [James]    Retry disabled. Monitoring success rate.
14:45  [Kratika]  Success rate recovering — 96.8%. Barclays-issued cards still failing.
14:50  [Sarah]    Customer comms drafted, posted to status page
14:52  [James]    Confirmed PSP has opened incident with Barclays. ETA unknown.
```

### Phase 4 - Communication
Communication during an incident has multiple audiences with different needs and different urgency:

#### Customers/Clients
Need to know there is a problem, that it is being worked on, and what they should do in the meantime. Use simple & plain language, no technical detail, no speculation about cause or timeline. Update at regular intervals, even "still investigating, no update" is better than silence.

#### Operations & Customer Support
Need to know what is broken, which customers are affected, and what workarounds (if any) exist. They are fielding calls and need information, not just awareness.

#### Compliance & legal 
Need to know if this incident triggers regulatory notification obligations. They will make that call, but they need the facts from engineering to do so; do not delay this conversation.

#### Bank/PSP/Third-party Integrators
If the incident involves a third-party payment rail or bank, they may need to be contacted directly and may have information you do not. Do not wait for the formal support ticket to escalate.

#### Leadership 
Need a single clear status at regular intervals. The message should not contain raw technical detail, but the business impact, customer impact, estimated resolution, and what is being done.

> If possible and bandwidth available, allocate dedicated person per audience. Do not involve engineers directly in the communication while they are debugging the issue. Engineering managers/Product managers are usually this dedicated person. 

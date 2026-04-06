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
If you can stop the bleeding, like disable a failing integration, or route traffic away from a broken PSP, or halt a batch job that is creating bad data, or reverse to last healthy system if release issue, do it before spending time understanding root cause. Stopping the harm and understanding the cause should be treated as separate activities.

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

> If possible and bandwidth available, allocate dedicated person per audience. Do not involve engineers directly in the communication while they are debugging the issue. **Engineering managers/Product managers** are usually this dedicated person.

### Phase 5 - Resolution
In a financial + distributed system, resolution is not the same as *service recovery*, but includes following considerations:

- The root cause is identified and addressed or mitigated.
- The payment flow is functioning correctly for all affected as well as non affected customer.
- No funds are in an indeterminate state.
- Related services, like reconciliation, has been run and no breaks attributable to the incident exist.
- Any regulatory notification obligation has been assessed and acted on.

> Check on *related services* is very important, and often forgotten step in incident resolution. Building an end-to-end checklist of system's health is crucial to avoid this.

### Phase 6 - Post-mortem (Retrospective & future strategies)
This is the final phase, often called as retrospective, where teams learns and improve, without blame-shifts and finger-pointing. 

Beyond standard formats, a successful retrospective phase consists following: 

- **Timeline reconstruction**, minute by minute. Use the logs/status reports generated during the earlier phases.
- **Identify contributing factors**, not humans, but the systems, strategies, conditions etc, that led to the issue.
- **Quantify user impact**, not just the financial impact to the organisation and the users, but the number of users/region affected; compliants received as well.
- **Create action items**, a comprehensive and owned action points to avoid future failures, & track their progress in the coming sprints, not just sitting in archives or backlogs.

---

## Approaches to incident-response team

### Centralised incident commander
A single incident commander owns the response end-to-end, where all decisions flow through this. Engineers report findings to them & the commander decides actions and communications with stakeholders. 
 
**Fits:** Smaller teams, early-stage fintechs, organisations where most engineers have not yet built incident response muscle.
 
**Limitations:** Creates a single point of failure in the human layer. If the commander is unavailable or overwhelmed, response degrades. 

### Dedicated role-based response team
Roles are pre-assigned and trained: incident commander, technical lead, communications lead etc, & each person operates in their lane. The incident commander coordinates across roles but is not doing all the work themselves. 
 
**Fits:** Mid-size engineering organisations, teams with regular on-call rotation, organisations under FCA or DORA operational resilience requirements.
 
**Limitations:** Requires investment in training and regular exercises, and roles need to be practiced to be effective.

### Tiered response by severity
P4 and P3 incidents are handled by the on-call engineer alone, using runbooks. P2 activates a small dedicated response team, while P1 activates the full war room with leadership involvement.
 
**Fits:** All fintech organisations at any stage. 

**Limitations:** Severity classification is a part of incident management, without involving relevant people, there is a risk of misclassification. 

---

## Recommendations from personal experience

- **Define severity matrix early** because the worst time to debate whether something is P1 or P2 is during a live incident. Write the matrix, walk through it with your team, and stress-test it against real past incidents.
- **Treat the resolution checklist as a gate, not a formality** as one service-check missed can lead to a disaster.
- **Invest in post-mortem quality over frequency:** One constructive post-mortem that produces three implemented action items is worth more than ten documents that get filed and forgotten.
- **Know your regulatory notification obligations across the team:** Every engineer in a senior role at a regulated fintech should know: what severity triggers which notification, and what the notification timeline is. This is not only the compliance team responsibility, but a shared responsibility that starts with engineering awareness. 
- **Run tabletop exercises quarterly:** The war room patterns that work under real pressure are the ones that have been practiced. Implement scenario-based exercises, "a PSP has gone down during peak hours, here is what you know at minute zero", to build the muscle memory that matters when the real incident fires.

---

## Further Reading
 
- [Google SRE: Managing Incidents](https://sre.google/sre-book/managing-incidents/): The foundational framework for incident management. Not fintech-specific, but the role-based model and the principle of separating technical response from communication are directly applicable.
- [PagerDuty Incident Response Guide](https://response.pagerduty.com/): Practical and free. The post-mortem template and severity definitions are worth adapting for fintech context.
- [FCA — Operational Resilience Policy Statement PS21/3](https://www.fca.org.uk/publications/policy-statements/ps21-3-building-operational-resilience): The UK regulatory framework. Engineering teams in FCA-regulated firms should understand what "impact tolerance" means and how incident severity maps to it.
- [DORA — ICT-Related Incident Classification](https://www.eba.europa.eu/regulation-and-policy/operational-resilience/digital-operational-resilience-act-dora): The EU framework. Specifically the RTS on incident classification — this defines when you must notify and how fast.
- [Stripe Engineering Blog — Increment: Reliability](https://increment.com/reliability/): Real engineering writing on operational reliability from one of the most operationally mature payment companies.
 
 ---


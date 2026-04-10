# Technical due-diligence in fintech 
What to look for when evaluating fintech engineered systems. 

## Context
Technical due-diligence (TDD) is a structured evaluation of systems, practices, & technical health of an organisation. It happens in three-main context:

- pre-acquisition or investment - an Investor or acquirer evaluating the target.
- leadership transition - an incoming CTO, or a fractional leadership assessing what they are inheriting.
- consulting engagement - an external advisor evaluating client before or during partnership.

A fintech with structural technical debt, sometimes, carries risks not directly visible on a balance sheet; like regulatory exposure, operational fragility in payment-critical systems, or a compliance obligation deferred rather than resolved. 

---

## What TDD is not
Before we talk about actual framework, let's clarify what technical due-diligence is not about. 

- **Not a code-audit:** While reading code is a part of TDD, the intent is to not find the bad code, but to understand whether the prganisation can reliably build, operate and scale the system as business grows. A codebase with ugly code but with operational runbooks, maximum test coverage and a reliable team has more chances of survival than a highly-architectured systems with no runbooks and monthly downtimes.
- **Not a technology checklist:** The questions on architecture styles or tech-stack without context produces no real value. The important part is to understand user and business context to decide if monolithic is better than microservices.
- **Not adversarial:** The best TDDs are almost always collaborative. Team who feels evaluated rather than interrogated are more likely to give out accurate informations. The goal is to identify structural gaps in a financial system and work on improving them, and not catch or point fingers on the people.

---

## The TDD-Evaluation framework
TDD in fintech has seven dimensions, each with different weight depending on the context - pre-investment, leadership transition or consulting engagement. 

![alt text](https://github.com/kratika-jain-em/em-playbook/blob/main/leadership-strategies/engineering-excellence/fintech-technical-due-diligence.png "Financial-technical-due-diligence")

### Dimension 1 - System Reliability

The first question in any fintech TDD is: what happens when something goes wrong?
 
This is a more revealing question than "what does the system do when it works." Any system works in the happy path. What differentiates mature from immature engineering organisations is how they handle failure.

#### What to ask
 
- *Walk me through the last three production incidents. What was the root cause, how long did it take to detect, and what changed as a result?*
- *How do you know a payment has failed? How long after the failure do you know?*
- *What happens to funds during a PSP outage? Where do they sit and who monitors them?*
- *What is your settlement failure rate? What happens when a settlement file is rejected?*
 
#### What you are looking for
 
Teams with mature operational practices can answer these questions quickly and specifically. They have incident timelines, dashboards, monitoring etc. If they don't have live examples, they have comprehensive runbooks to follow. 

---

### Dimension 2 - Data Integrity

Financial data has a different standard than other data. It must be accurate to the penny, complete across all time periods, auditable to any point in history, and consistent across every system that holds a view of it.

#### What to ask
 
- How do we verify the ledger is correct?
- Can we reconstruct the balance of any account at any point in the past 7 years?
- What is the process when a reconciliation break is found, specifically, what data is corrected and how is the correction audited?
- Show me the schema for your core financial tables. Walk me through how a payment flows through them.
 
#### What you are looking for
 
The ledger schema question is one of the most revealing in a fintech TDD. A team that has thought carefully about financial data modelling will have a double-entry ledger, explicit pending and posted states, immutable transaction records, and a clear audit trail for every state change. 

---

### Dimension 3 — Compliance Engineering
 
Regulatory compliance in fintech is not a binary state, but a continuous practice. An organisation can be technically compliant today and non-compliant tomorrow because a regulation changed, a new product was launched without a compliance review, or a system change inadvertently reduced a control.
 
#### What to ask
 
- Which regulations apply to this business, and who in engineering owns the understanding of each?
- When was the last regulatory change that required an engineering response? How was it handled?
- What is your PCI DSS scope? When was the last assessment?
- Have you ever received a regulatory finding or action? What was it and what changed?
- What would trigger an FCA, DORA, or CFPB notification from an engineering perspective, and who makes that call?
- Walk me through how a new feature goes through a compliance review before launch.
 
#### What you are looking for
 
The best answer to "who in engineering owns regulatory understanding" is a specific name or role, not "the compliance team." 
 
The feature compliance review question is particularly revealing. Mature organisations have a defined process, with legal and compliance review at design stage, not post-launch. Immature organisations have a version of "compliance reviews it before it goes live" that in practice means a checkbox after engineering is already done.

---
 
### Dimension 4 — Security 
 
Security in fintech is not just about protecting user data, but also about protecting funds, preventing fraud at the infrastructure level, and maintaining the trust of payment schemes and regulators.
 
#### What to ask
 
- How are production secrets managed? Who has access to production credentials?
- When was the last penetration test? What were the findings and what was remediated?
- How are privileged access and production access controlled?
- What is the process when an employee with production access leaves?
- How is sensitive financial data, like PAN data, bank account numbers, sort codes etc, stored and accessed?
 
#### What you are looking for
 
The secrets management question is a fast signal. Teams with mature security practices use a secrets manager (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager) with rotation policies, audit logging, and no hardcoded credentials. Teams with immature practices have environment variables, shared accounts, and .env files that have been committed to version control at some point in their history.
 
The employee offboarding question is more revealing than it sounds. "We revoke their accounts" is not sufficient. The question is: what accounts, how quickly, & verified how? A production system with shared credentials and no access audit trail cannot answer this question well.

---
 
### Dimension 5 — Team & Organisational Health
 
Systems are built and maintained by people. The most important technical debt in many organisations is not just in the code, but also in the team structure, the knowledge distribution, and the organisational conditions that determine whether the engineering organisation can sustain what it has built.
 
#### What to ask
 
- Who are the two or three people whose departure would most damage the engineering organisation's ability to operate? What is the plan if they leave?
- How is knowledge documented and transferred? Can a new engineer get productive on the system without those key people?
- What is the on-call rotation? How many engineers are on it, and how frequent is each person's rotation?
- What is the voluntary attrition rate in engineering over the last 12 months?
- What does a typical sprint look like in terms of new features vs maintenance vs technical debt vs incidents?

#### What you are looking for
 
A payment system where two people hold the full operational knowledge is a business risk, not just an engineering risk. If those two people leave, the ability to safely operate and change the system goes with them.
 
The sprint composition question tells you whether the team is in a sustainable operating rhythm or whether they are in a permanent firefighting mode. A team spending more than 30% of sprint capacity on incidents and unplanned work is not in a state where it can safely accelerate delivery.

---
 
### Dimension 6 — Technical Debt Assessment
 
Technical debt in fintech can create regulatory risk, operational fragility, or security exposure, and not just development velocity. These categories need to be assessed separately from the standard "this makes us slower" debt.
 
#### Debt classification for fintech
 
| Category | Risk | Example |
|---|---|---|
| **Regulatory debt** | Compliance exposure | Audit logs that don't meet FCA retention requirements |
| **Operational debt** | Incident risk | No runbooks for a critical payment flow |
| **Security debt** | Data or fund exposure | Secrets without rotation, excessive production access |
| **Financial data debt** | Reconciliation risk | Mutable ledger records with no event history |
| **Velocity debt** | Delivery slowdown | Tightly coupled monolith making changes risky |
 
The first four categories should be treated as risk items, not engineering backlog items. They belong in a risk register, not a sprint backlog, because their remediation has a different urgency and a different stakeholder audience.
 
#### What to ask
 
- What are the three things in the system that you are most worried about?
- Is there anything that you would not feel comfortable changing right now? Why?
- What would happen if you needed to switch PSPs in 30 days?
- How long would it take to onboard a new bank integration?

#### What you are looking for

The "what are you most worried about" question is the most valuable in a TDD because engineers know. They will tell you if they trust that you are asking in good faith. The answers to this question have frequently been more revealing than any amount of architectural documentation.

---

### Dimension 7 — Delivery Velocity & Process

#### What to ask
 
- How long does it take from a feature being approved to it being in production?
- What is your deployment frequency for the payment service?
- How do you manage database migrations in production?
- What does the release process look like for a change to core payment logic?
- When was the last time a deployment caused a production incident?
 
#### What you are looking for
 
Deployment frequency is a leading indicator of organisational health, not just technical maturity. Teams that deploy frequently have necessarily solved many hard problems: fast test cycles, safe rollback, good observability, and a culture of small incremental changes. Teams that deploy monthly or quarterly usually have none of these things and are accumulating risk with every release.
 
The database migration question is specific to fintech as payment systems have strict schema change requirements, and the process for zero-downtime migrations is a meaningful signal of operational maturity.

---
 
## My recommendation
 
### Start with the worry question
Before any structured evaluation, ask the engineering leadership: "What are the three things in this system you are most worried about?" The answers orient the rest of the TDD toward what actually matters. If those answers do not surface in the formal evaluation, the formal evaluation is incomplete.
 
### Weight the fintech-specific dimensions heavily 
In a standard TDD, systems reliability and team health are typically the top-weighted dimensions. In fintech, data integrity and compliance engineering deserve equal weight. A business with a regulatory finding or a material reconciliation gap has a problem that velocity and architecture quality cannot compensate for.
 
### Ask for a demonstration, not a presentation 
"Show me the reconciliation dashboard" is more informative than "tell me about your reconciliation process." "Run a payment through the test environment and show me what happens in the ledger" is more informative than an architecture diagram. What people show you in a live/sandbox environment reveals what they are confident in.
 
### Distinguish between debt that slows and debt that risks 
The most important output of a fintech TDD is a clear categorisation of technical debt by risk type, not just by remediation effort. A board or investor needs to understand: what of what we found creates regulatory exposure? What creates operational fragility? What just makes us slower? These are different conversations with different stakeholders.
 
### Write the finding that engineering already knows 
The most useful TDD report for an engineering organisation is one that articulates clearly in business language, what the engineering team already knows but has not been able to communicate with sufficient urgency to the people who need to act on it. The evaluator's role is often translation as much as discovery.

---

## Further Reading
 
- [Martin Kleppmann — Designing Data-Intensive Applications](https://dataintensive.net/) — The theoretical foundation for understanding data integrity, consistency, and audit in distributed financial systems.
- [a16z — Technical Due Diligence](https://a16z.com/technical-due-diligence/) — A good general framework from the investor perspective. Apply the fintech-specific overlays from this piece on top.
- [FCA — Technology and Cyber Resilience Questionnaire](https://www.fca.org.uk/firms/technology-cyber-resilience) — What the FCA asks of regulated firms. Useful as a compliance dimension checklist,if a firm cannot answer these questions, neither can a TDD evaluator who did not ask them.
- [DORA — EBA Guidelines on ICT and Security Risk Management](https://www.eba.europa.eu/regulation-and-policy/operational-resilience/guidelines-ict-and-security-risk-management) — The EU standard for what good looks like in operational and security risk management. Directly applicable to the compliance and security dimensions of a fintech TDD.
- [Increment — Issue 19: Security](https://increment.com/security/) — Real engineering writing on security practices at scale. Several pieces directly applicable to the security posture dimension of fintech TDD.
 

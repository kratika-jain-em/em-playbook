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

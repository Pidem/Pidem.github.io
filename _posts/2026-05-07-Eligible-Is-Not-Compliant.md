---
layout: post
title: "Eligible Isn't Compliant"
subtitle: "Closing the gap between AWS services and validated AI in GxP environments"
date: 2026-05-07
---

# Eligible isn't Compliant

*Closing the gap between AWS services and validated AI in GxP environments*

---

Every life sciences customer I talk to arrives with the same question, phrased differently each time:

> "You keep saying Bedrock and AgentCore are *eligible*. Show me one customer that actually made it *compliant* in their GxP environment."

The frustration is fair. "HIPAA-eligible" and "GxP-compatible" are AWS statements about AWS Services. They don't say anything about whether the application a customer builds on top is validated, inspection-ready, or acceptable to a regulator. Eligibility is the starting line, not the finish line.

---

## The gap in one sentence

The shared responsibility model for GxP AI follows the more general cloud shared responsibility model: AWS provides capabilities and the customer owns the final decision on how to build.

| Capability domain | AWS provides | Customer owns |
|---|---|---|
| **Qualified infrastructure & identity** | ISO 27001/9001, SOC 1/2/3, FedRAMP, HIPAA eligibility; IAM, IAM Identity Center, AgentCore Identity | Supplier qualification documentation, role definitions, least-privilege policies, segregation of duties |
| **Observability** | AgentCore Observability (sessions, traces, spans), CloudTrail, CloudWatch | What to monitor, thresholds, alerting rules, log retention policy |
| **Encryption & data protection** | KMS, TLS primitives, Bedrock Guardrails PHI/PII redaction, Comprehend Medical | Sensitive data classification, key management policy, data residency decisions |
| **Immutability & versioning** | Bedrock immutable model versions, CloudFormation IaC, S3 Object Lock (WORM) | Pinning model versions, prompt and tool version control, change approval process |
| **Control flow & HITL** | Step Functions, Prompt Flows, AgentCore Policy, MCP elicitation support | HITL trigger design, confidence thresholds, escalation paths, approver workflows |
| **Evaluation & assurance** | AgentCore Evaluations, Bedrock model evaluations, Automated Reasoning, Audit Manager, Config conformance packs (21 CFR Part 11) | Validation strategy (CSA / IQ/OQ/PQ), acceptance criteria, intended use documentation, SOP integration |

AWS delivers the primitives. The customer defines the policy. The regulator inspects how those two combine in your application. You don't become compliant by procuring eligible services. You become compliant by *using them inside a validated application*.

---

## Why uniform validation doesn't work anymore

Traditional CSV treated every system the same: full IQ/OQ/PQ regardless of risk. That approach breaks on AI for multiple reasons: First, it's expensive. Full CSV on every prompt change is not survivable. Second, it's not flexible enough. A literature summarization tool used internally has nothing in common, risk-wise, with an AI agent that influences a regulatory submission.

Two forces are driving the change. FDA's [Computer Software Assurance guidance](https://www.fda.gov/media/161521/download) is the efficiency fix: match validation rigor to actual risk, not to a uniform template. The EU AI Act's August 2026 high-risk enforcement is the deadline. Conformity assessment, technical documentation, and ongoing monitoring are now legal obligations for high-risk AI, and most life sciences use cases (clinical decision support, pharmacovigilance, quality) land in that bucket.

CSA gives you the framework. The EU AI Act sets the clock.

The classic example: an AI agent that summarizes scientific literature.

- **Internal team meeting summaries** → low risk → minimal controls, basic audit trail
- **Input to research direction** → medium risk → structured HITL, drift monitoring, Guardrails
- **Cited in a regulatory submission** → high risk → full IQ/OQ/PQ, automated reasoning, red team, CAPA

Before any PoC: classify the workload. 

> Severity × probability × detectability 

What is the harm being done if something goes wrong? How likely is this scenario ? Can I detect any failures or hallucinations? These questions ultimately dictate everything downstream: validation cost, time to production, documentation burden, and whether the system will survive an inspection.

---

## What regulators actually expect

Regulators don't care about specific use cases. They care about a small set of properties the system must demonstrate, regardless of whether it's harmonizing clinical data or screening promotional material. Five expectations cover most of what matters, and each maps to a concrete pattern that you can build with features of eligibile AWS Services.

**1. Reproducibility: same input, same output.**
The probabilistic nature of LLMs can be monitored: You can pin model versions, lower model temperatures, version your prompts in source control, define your applications as Infrastructure as Code (Iac). You get a deterministic system made of stochastic parts. AgentCore evalutions can help you monitor and verify and track agentic behavior. *This is relevant for: clinical data harmonization (eDTM → ADaM) and anything that feeds regulatory submissions.*

**2. Traceability: every factual claim ties to a source.**
Retrieval-augmented generation with citation attribution via Bedrock Knowledge Bases. Automated Reasoning checks against the grounded context. If it can't cite it, it can't say it. Final authorship stays human. AI accelerates drafting and consistency. *This is relevant for: clinical study report authoring, MLR content review.*

**3. Explainability: the reasoning chain is inspectable, not only the output.**
AgentCore Observability captures sessions, traces, and spans: the full chain of tool calls, retrieved documents, and intermediate decisions that produced a given answer. For multi-step agents, this is the audit trail. In pharmacovigilance signal detection, the question "why did you flag this case?" needs a reconstructable answer, not only a probability score.

**4. Human accountability: a qualified person signs off on consequential decisions.**
Step Functions HITL gates, Prompt Flow approvals, AgentCore Policy enforcing action boundaries in code, not only in procedure. Confidence thresholds route low-certainty outputs to human review. The system can recommend; the human decides. *Where this bites: MLR pre-screening (committee retains authority), AE case adjudication, any AI output that influences a regulatory submission.*

**5. Continuous monitoring and change control: validation is where compliance begins, not where it ends.**
CloudWatch thresholds on model performance, drift detection via AgentCore Evaluations, AWS Config for configuration drift, Audit Manager for continuous evidence collection. This expectation is where most customers hit the "partner AI" problem: AI features auto-activating in validated Veeva, Box, ServiceNow, and Salesforce environments through monthly tenant updates. Unless change control detects and qualifies those activations *before* they go live, you're running an unvalidated change in a validated GxP system. Nearly every customer who has invested in GxP SaaS platforms carries this exposure today.

These five are the lens. If your architecture can demonstrate all five with AWS-native evidence, the use case specifics become details, not obstacles.

---

## Validation as an ongoing-journey

The unlock for GxP AI is treating the validation lifecycle as infrastructure, not paperwork.

- **Sprint 0:** risk classification, CSA-based validation strategy, IaC baseline (CloudFormation), AWS Config conformance pack enabled, CI/CD pipeline with compliance gates.
- **Dev sprints:** feature work with prompts versioned, models pinned, Guardrails configured, and evidence auto-collected via Audit Manager.
- **Validation sprint (not quarter):** IQ is automated CloudFormation deployment verification. OQ and PQ are automated test execution against pre-agreed acceptance criteria, including AgentCore Evaluations for AI-specific performance tests. Trace review happens in AgentCore Observability. Quality gate signs off.
- **Production:** continuous validation via CloudWatch thresholds, AgentCore Evaluations, drift detection, quarterly review via Audit Manager reports, change control triggered on any model or prompt change.

The regulated landing zone pattern (pre-qualified multi-account environments with change management, configuration, and security baked in) means each new AI workload starts on a validated foundation instead of building one. Customers using this approach report 30–40% reductions in qualification cycle times, and timelines from 12–16 weeks down to 4–8 weeks for adding a new service like Bedrock to an already-qualified estate.

Validation becomes reusable infrastructure, not paperwork redone each time.

---

## The bottom line

"Is this feasible in a GxP environment?" is the wrong question. Of course it's feasible, and multiple top-10 pharma have done it. The real questions are:

1. Have I classified the workload against actual risk, not assumed uniform validation?
2. Is my validation strategy expressed as infrastructure, so it can be re-run rather than re-invented?
3. Do I have the AI-specific controls (prompt versioning, tool dependency locking, drift detection, automated reasoning) that standard software validation doesn't cover?
4. Am I governing the partner AI features that are activating in my validated platforms whether I'm ready or not?

Eligibility gets you through the AWS part of the conversation. Compliance is what you build on top. The good news: the patterns are now concrete enough to be repeatable.

---

Additional resources: [A Guide to Building AI Agents in GxP Environments](https://aws.amazon.com/blogs/machine-learning/a-guide-to-building-ai-agents-in-gxp-environments/).

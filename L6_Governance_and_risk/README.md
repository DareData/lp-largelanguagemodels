# Lecture 6: Governance and Risk

**Block D: Operating LLM systems**

### Scope

How to keep an LLM application in production trustworthy, lawful and affordable. The lecture covers guardrails that keep answers grounded and route uncertain cases to people, where the system and its model calls are hosted and what the cloud does and does not guarantee for personal data, and the cost and latency budgets that keep the system viable at scale. It builds on the observability and release practices of lecture 5.

```mermaid
flowchart LR
    G["Hallucination guardrails<br/>(lesson 00)"] --> P["Data privacy and PII<br/>(lesson 01)"] --> C["Cost and latency budgeting<br/>(lesson 02)"]
```

### Learning Objectives

By the end of this lecture, you will be able to:

- **Design** guardrail layers: intent detection, grounding, structured outputs, citation validation and routing to clarification or human review.
- **Protect** customer data with EU hosting of every component, contractual, technical and jurisdictional provider guarantees, and the application controls no provider gives: authorization before retrieval, minimisation and access control.
- **Budget** latency and cost with percentiles, time to first token and cost per successful task, and enforce limits in code.

### Running example

A customer-support assistant that answers questions about products, orders, accounts and policies using the customer's own data. The practices apply to any LLM application that serves people from company or personal data.

### Lessons in this lecture

00. [Hallucination Guardrails](00-hallucination-guardrails.md)
01. [Data Privacy and PII](01-data-privacy-and-pii.md)
02. [Cost and Latency Budgeting](02-cost-and-latency-budgeting.md)

### Coming next

- [Lecture 7](../L7_AI_Chatbot_Use_Case/README.md): an applied AI chatbot: architecture, conversation storage, context management and security.

### Further reading

- NIST, [AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) (released January 2023) and its Generative AI profile, NIST-AI-600-1 (July 2024): a voluntary framework for managing AI risk across an organisation.

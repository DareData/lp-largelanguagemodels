# Lesson 01: Data Privacy and PII

### Where to host personal data, what to demand from a provider, and what you still have to do yourself

Many LLM applications handle personal data: customer accounts, orders, payments, contracts and support history. For most European companies the first privacy decision is **where the application, its data and its model calls are hosted**. Hosting everything in the EU keeps the data under EU rules and avoids most of the paperwork needed to send personal data abroad. It also leaves physical security, patching and certifications to a cloud provider that does them at scale. This lesson explains why the EU is the default, how to choose a provider, which guarantees to ask for, and which protections no provider can give you.

<p align="center">
  <img src="../images/l6-01-01.png" alt="Privacy meme" width="400">
</p>

### Learning Objectives

By the end of this lesson, you will be able to:

- **Explain** why EU hosting is the default for personal data, and why storing data in the EU does not by itself protect it from non-EU laws.
- **Check** that every component, including the model endpoint, logs and surrounding services, stays in the EU.
- **Assess** what a provider guarantees in its contract, in its technology and through the country of its owner.
- **Separate** what the provider guarantees from the controls the team still has to build.

---

### Lesson Content

#### Why the EU is the default

Under the GDPR, personal data can move freely within the EU and the wider European Economic Area. Sending it anywhere else is allowed, but only with an extra legal basis for each destination, called a [transfer mechanism](https://commission.europa.eu/law/law-topic/data-protection/international-dimension-data-protection_en). Examples are the EU-US Data Privacy Framework or standard contractual clauses. The European data-protection authorities also [recommend](https://edpb.europa.eu/system/files/2021-06/edpb_recommendations_202001vo.2.0_supplementarymeasurestransferstools_en.pdf) a written assessment of the risks in the destination country. These arrangements can also fall: the EU courts have already struck down two earlier EU-US agreements. If the data never leaves the EU, none of this is needed, and the team has one simple rule to check every component against.

| Reason | What EU hosting gives |
| :-- | :-- |
| Simpler GDPR compliance | No contractual clauses or risk assessments for data that stays in the EU |
| Protection from legal change | Nothing to redo if a transfer agreement is struck down again |
| Sovereignty | The data is governed by European law: the GDPR, the [Data Act](https://eur-lex.europa.eu/eli/reg/2023/2854/oj/eng) and the [Data Governance Act](https://eur-lex.europa.eu/eli/reg/2022/868/oj/eng) |
| Easier sales | Many enterprise, healthcare and public-sector buyers require EU hosting |
| Sector rules | Banks and insurers can more easily show the supplier control and exit plans that [DORA](https://eur-lex.europa.eu/eli/reg/2022/2554/oj/eng) requires |

#### Storing data in the EU is not the same as being protected by EU law

Two questions decide how safe EU data really is. **Where is the data stored?** This is called residency. **Which country's laws apply to the company that runs the servers?** This depends on who owns the provider.

Microsoft Azure shows why the difference matters. Its [EU Data Boundary](https://learn.microsoft.com/en-us/privacy/eudb/eu-data-boundary-learn) commits Microsoft to store and process customer data inside the EU and EFTA, apart from a few documented exceptions, and its model service offers an EU [data zone](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/deployment-types) so that model calls are also processed in the EU. That is strong residency. But Microsoft's parent company is American, so EU residency does not protect the data from US legal requests. Even when data is stored and processed in the EU, a US authority can order Microsoft to disclose it under laws such as the [CLOUD Act](https://www.law.cornell.edu/uscode/text/18/2713). The same applies to the EU regions of Amazon, Google and every other US-owned cloud.

European-owned providers, such as OVHcloud, Scaleway, IONOS or STACKIT, are based in the EU and have no US parent company, so US orders of this kind cannot reach the data through them. That is why public-sector and other sovereignty-sensitive buyers often prefer them. The downside is that they offer fewer services and fewer of the newest models, so many teams combine them with open-weight models they run themselves.

| Option | Where the data is stored | Whose laws can reach it | Trade-off |
| :-- | :-- | :-- | :-- |
| US-owned cloud in an EU region (for example Azure with the EU Data Boundary) | EU | EU and US | Most services and the newest models |
| Same, with encryption keys held by you, outside the provider | EU | EU and US, but the provider can only hand over encrypted data | You have to manage the keys |
| European-owned provider with no non-EU parent | EU | EU only | Fewer services, fewer or later models |
| Open-weight model on your own EU infrastructure | EU | Your own | You run the GPUs, serving and security |
| Global model endpoint | Anywhere | The provider's | Not EU hosting at all |

#### Where data leaves the EU

An application can run in an EU region and still send personal data out of it. The main server is rarely the problem. The leaks come from the small services and default settings around it, and AI services are the ones most often forgotten, even though they receive the customer's own words.

| Leak | Why it happens | Fix |
| :-- | :-- | :-- |
| Global model deployment | It is cheaper and gets new models first, so it is often the default | Use an EU data zone or a regional deployment, and block global deployments by policy |
| The model vendor's own API | It may process data outside the EU | Turn on the vendor's EU processing, or call the model through your EU cloud |
| Tracing, analytics and error monitoring | Prompts, IP addresses and user ids are sent to tools hosted elsewhere | Use an EU instance, host the tool yourself, or remove personal data before sending |
| Email, CDN, support and chat tools | Names, addresses and history are processed worldwide | Choose EU settings or EU alternatives |
| Backups and failover | Copies are replicated to a region outside the EU | Pick backup and failover regions explicitly |
| Support and operations staff | Staff looking at the data from outside the EU counts as a transfer | Limit remote access in the contract |

---

#### Provider guarantees to require

Under the GDPR the provider is a **processor**: it handles personal data on your behalf. [Article 28](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng) requires a written contract in which the provider agrees to use the data only as you instruct, keep it secure, use subcontractors only with your approval, and delete or return the data at the end. Get every answer below written into that contract, not just said in a sales meeting.

| Guarantee | Question to ask |
| :-- | :-- |
| Location of data and model calls | Where are data, prompts, backups and logs stored and processed? |
| No training on your data | Do the terms exclude prompts, outputs and fine-tuning data from training? |
| Retention and abuse monitoring | How long are prompts kept, who can read them, and can we opt out? |
| Subcontractors | Who are they, where do they work, and are we told before they change? |
| Owner's country | Where is the parent company, and can staff access the data from outside the EU? |
| Encryption and keys | Is data encrypted by default, and can we hold the keys for the model, the search index and the logs? |
| Private network and policies | Can the model API be reached only from our private network, and can we enforce regions and deployment types? |
| Breaches and audits | Will we be told about a breach without delay, and do ISO 27001 or SOC 2 reports cover the AI services? |
| Exit | How is our data returned and deleted when we change provider? |

Even the best guarantees only cover the infrastructure. Under the cloud's [shared responsibility model](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility), you always remain responsible for your data, accounts, access rights and configuration. One customer seeing another customer's data, or personal data written in plain text to a log, can happen just as easily in a correctly hosted, fully certified environment.

#### What stays your job

| Control | What to do |
| :-- | :-- |
| Check permissions before searching | Take the user's identity from the logged-in session, filter the search by what that user may see, and have every tool check ownership again; never let the model decide whose data to read |
| Send less data | Send the model only the fields it needs; mask, replace or summarise the rest |
| Clean the logs | Remove personal data from traces and logs before they are stored; detection tools such as Presidio help but miss some of it |
| Limit staff access | Single sign-on and roles for staff; access to unmasked data is rare, justified and logged |

The OWASP Top 10 for LLM applications lists [sensitive information disclosure](https://genai.owasp.org/llmrisk/llm022025-sensitive-information-disclosure/) as a top risk. It also warns that telling the model in the system prompt not to reveal data is not a protection, because prompt injection can get around it.

**Practices overview**

| Practice | Why |
| :-- | :-- | 
| Host every component that touches personal data in the EU | No transfer paperwork and no legal uncertainty |
| List every surrounding service and where it keeps data | Data usually leaves through small services, AI services most of all | 
| Ask both where the data is and who owns the provider | A US authority can order a US-owned provider to disclose EU-stored data | 
| Write location, training, retention, subcontractor and exit terms into the contract | The contract, not the console setting, is the guarantee |
| Check permissions before searching, and send the model only what it needs | The provider does not protect your application |

#### In practice

Decide first where to host and which provider to trust, based on how sensitive the data is. A US-owned cloud with EU residency suits most business data; public-sector, health and other sensitive data often calls for a European-owned provider or self-hosting. Write the region down as a requirement for every component and surrounding service, enforce it with cloud policies, and put the answers to the guarantee questions in the contract. Then do the work no provider does for you: permission checks in search, sending the model only what it needs, and clean logs. Involve the data-protection officer early, and a lawyer for sensitive data.

**Summary:**
- EU hosting is the default because it keeps personal data under EU rules and avoids the paperwork and uncertainty of sending it abroad.
- Storing data in the EU is not enough on its own: a US-owned provider such as Microsoft can still be ordered by a US authority to disclose it, while European-owned providers fall under EU law only.
- Data usually leaves the EU through global model deployments, monitoring and other tools, backups and support access, not through the main server.
- Provider guarantees belong in the contract; permission checks, sending less data, clean logs and staff access remain the team's job.

---

### References

- European Commission, [International dimension of data protection](https://commission.europa.eu/law/law-topic/data-protection/international-dimension-data-protection_en) (accessed October 2026).
- [Regulation (EU) 2016/679 (GDPR)](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng); [Data Act](https://eur-lex.europa.eu/eli/reg/2023/2854/oj/eng); [Data Governance Act](https://eur-lex.europa.eu/eli/reg/2022/868/oj/eng); [DORA](https://eur-lex.europa.eu/eli/reg/2022/2554/oj/eng).
- European Data Protection Board (2021), [Recommendations 01/2020 on measures that supplement transfer tools](https://edpb.europa.eu/system/files/2021-06/edpb_recommendations_202001vo.2.0_supplementarymeasurestransferstools_en.pdf).
- [18 U.S.C. 2713](https://www.law.cornell.edu/uscode/text/18/2713), Required preservation and disclosure of communications and records (CLOUD Act, 2018).
- Microsoft, [What is the EU Data Boundary?](https://learn.microsoft.com/en-us/privacy/eudb/eu-data-boundary-learn), [Deployment types in Microsoft Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/deployment-types) and [Shared responsibility in the cloud](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility) (accessed October 2026).
- OWASP (2025), [LLM02 Sensitive Information Disclosure](https://genai.owasp.org/llmrisk/llm022025-sensitive-information-disclosure/); Presidio, [documentation](https://presidio.dataprivacystack.org/).

### Further Reading

- [Lesson 00: Hallucination Guardrails](00-hallucination-guardrails.md) - the previous lesson.
- [Lesson 02: Cost and Latency Budgeting](02-cost-and-latency-budgeting.md) - the next lesson.
- [Lecture 2, Lesson 01: Tool Calling](../L2_Agentic_systems_and_tool_calling/01-tool-calling.md) - identity from the session and complete mediation.

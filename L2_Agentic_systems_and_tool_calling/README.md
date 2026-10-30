# Lecture 2: Agentic Systems and Tool Calling

**Block B: Agentic systems**

### Scope

How a language model becomes a system that acts, with practical scenarios in every lesson. The lecture covers what an agent is and how the agentic loop runs, tool-calling best practices (descriptions, toolset size, errors and enforcement in code), the architectures built from tool calls, from fixed pipelines and ReAct to an orchestrator with subagents, how frameworks orchestrate state, memory, handoffs to people and resumed work, and the Model Context Protocol. Observing and evaluating agents in production is covered in [lecture 5](../L5_LLMOps/README.md).

### Learning Objectives

By the end of this lecture, you will be able to:

- **Define** an agent, its building blocks and the level of autonomy a task needs.
- **Implement** the agentic loop correctly, stopping on the finish signal.
- **Design** tools a model chooses correctly, size toolsets, structure errors, and enforce critical rules in code.
- **Choose** between fixed pipelines, ReAct and an orchestrator with subagents, and design the orchestrator's decomposition, briefs and error handling.
- **Build** an agent as a graph with state, routing, a ReAct loop, memory and human approval.
- **Connect** an agent or an existing host to MCP servers, and apply their security duties.

### Running example

Each lesson illustrates its ideas with short scenarios from different businesses (a bicycle repair shop, a brewery, a bookshop, an accounting firm and others). The order-support assistant for a retail chain, with tools to search customers, look up orders and start a return, is built end to end in the companion notebook.

### Lessons in this lecture

00. [Introduction to Agentic Systems](00-introduction-to-agentic-systems.md)
01. [Tool Calling](01-tool-calling.md)
02. [Agent Architectures and ReAct](02-agent-architectures-and-react.md)
03. [Building Agents: Orchestration, State and Memory](03-building-agents-orchestration-and-memory.md)
04. [Introduction to MCP](04-introduction-to-mcp.md)

### Companion notebook

[code/order_support_agent.ipynb](code/order_support_agent.ipynb): the order-support assistant as a graph with a router, a ReAct branch, an approval interrupt and identity checks, adapted from the Hugging Face Agents Course email-sorting notebook. It runs on Azure OpenAI, or offline with a scripted stand-in for the model.

### Coming next

- [Lecture 3](../L3_Retrieval_augmented_generation/README.md): retrieval-augmented generation: data preparation, chunking, embedding models and retrieval.

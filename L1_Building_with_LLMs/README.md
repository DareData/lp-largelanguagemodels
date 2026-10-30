# Lecture 1: Building with LLMs

**Block A: LLM basics**

### Scope

From how an LLM turns a prompt into a reply, token by token, to choosing the model that serves it: architecture, tokens, context windows and the parameters that shape each reply; output your code can parse and trust; and how to choose between proprietary, open-weight and self-hosted models.

### Learning Objectives

By the end of this lecture, you will be able to:

- **Explain** how an LLM turns a prompt into a reply, token by token, and why that drives limits and cost.
- **Define** the context window and **choose** generation parameters such as temperature, max tokens and stop sequences.
- **Get** structured output your code can trust, using schemas, validation and bounded retries.
- **Choose** a model by quality, cost, latency, data constraints and licensing, and keep the choice reversible.

### Prompt engineering

Go over [Prompting best practices](Support_Material/Prompting%20best%20practices%20-%20Claude%20Platform%20Docs.pdf) in the support material as the reference for writing prompts throughout the course: clear and direct instructions, examples, structuring with tags, roles in the system prompt, step-by-step reasoning and controlling the format of the reply.

### Lessons in this lecture

00. [LLM Basics: Architecture, Tokens, Context Windows and Generation Parameters](00-llm-foundations.md)
01. [Structured Outputs](01-structured-outputs.md)
02. [Model Selection](02-model-selection.md)

### Coming next

- [Lecture 2](../L2_Agentic_systems_and_tool_calling/README.md): agentic systems and tool calling: agents, tools, architectures, frameworks and MCP.

### Support material

- [Attention Is All You Need](Support_Material/NIPS-2017-attention-is-all-you-need-Paper.pdf) (Vaswani et al., 2017): the transformer paper behind lesson 00.
- [Prompting best practices](Support_Material/Prompting%20best%20practices%20-%20Claude%20Platform%20Docs.pdf): prompt engineering guidance.

# Glossary

<!--
Editorial policy (maintainer notes):
- Define terms here once; other pages should link here rather than redefine.
- Add terms only when they appear in at least one curriculum page.
- Keep entries concise and implementation-neutral where possible.
- Prefer examples from LCLS/Ops workflows.
- Keep ### headings alphabetically ordered by term title.
- Core terms (site UI badge / filter): Agent, Harness, MCP (Model Context Protocol),
  Model, RAG (Retrieval-Augmented Generation), SKILL, Tooling.
-->

### Agent

- **Definition:** A system that observes context, decides next actions, uses tools, and checks outcomes toward a goal.
- **Why it matters:** It frames the practical loop learners build and evaluate.
- **LCLS example:** Reads run notes, queries a facility status tool, and proposes next operator checks.
- **Note:** Not just a model response; it is model plus harness plus tool use.

### Context Window

- **Definition:** The maximum amount of input and conversation history a model can consider in one request.
- **Why it matters:** It shapes how much state and retrieved content can be passed at once.
- **LCLS example:** A long incident thread may need summarization before adding fresh logs and instructions.

### Evaluation

- **Definition:** A repeatable method to measure whether agent behavior meets expected quality, safety, and reliability criteria.
- **Why it matters:** It turns subjective impressions into trackable performance signals.
- **LCLS example:** Test prompts verify that an agent cites the right source and avoids unsafe write actions.

### Guardrails

- **Definition:** Constraints and checks that limit unsafe behavior, such as permission boundaries, argument validation, and approval gates.
- **Why it matters:** They reduce operational risk in tool-using systems.
- **LCLS example:** A workflow requires explicit approval before any irreversible command is executed.

### Harness

- **Definition:** The orchestration layer that connects the model to tools, permissions, context handling, and execution flow.
- **Why it matters:** It defines how safely and reliably tool-using behavior runs.
- **LCLS example:** A coding harness runs shell commands with guardrails and logs execution results for review.
- **Note:** Harness logic should stay replaceable and not absorb domain-specific business logic.

### MCP (Model Context Protocol)

- **Definition:** A protocol for exposing tools and resources to models through a standardized interface.
- **Why it matters:** It reduces integration friction and makes tool access patterns more consistent.
- **LCLS example:** An MCP server exposes read-only facility status and searchable documentation resources to agents.
- **Note:** MCP standardizes tool access; it does not replace tool design or safety policy.

### Model

- **Definition:** The language model that performs reasoning and text generation (for example Claude, GPT, Gemini, or Llama).
- **Why it matters:** Model choice affects quality and cost, but most durable value comes from tooling and process.
- **LCLS example:** A general model interprets an operator request and drafts the right sequence of tool calls.
- **Note:** A model alone is not an agent workflow.

### RAG (Retrieval-Augmented Generation)

- **Definition:** A pattern where the system retrieves relevant documents or records at runtime and supplies them to the model before generation.
- **Why it matters:** It improves factual grounding without retraining models.
- **LCLS example:** Before answering a question, the agent retrieves recent operations notes and current runbook snippets.
- **Note:** RAG is retrieval plus prompting; it is not model fine-tuning.

### SKILL

- **Definition:** A structured instruction document that teaches an agent how to use a specific tool safely and effectively.
- **Why it matters:** It captures operational know-how in readable, version-controlled text.
- **LCLS example:** A SKILL explains how to query a service, interpret fields, and handle expected failure modes.
- **Note:** Like a man page for tool usage, not a model-weight customization.

### Tooling

- **Definition:** External capabilities the agent can invoke, such as CLI tools, APIs, indexes, scripts, and data pipelines.
- **Why it matters:** Tooling is the main long-term investment because humans and agents can both use it.
- **LCLS example:** A command-line tool that queries beamline metadata is useful to both operators and agents.

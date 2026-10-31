# Report: Building AI Agents with LangGraph

## 1. Chain vs. agentic workflow
A conventional LLM chain runs a fixed sequence of steps (prompt → model → parser), always in the same order. An agentic workflow lets the model decide at runtime: which branch to follow, whether to call a tool, whether to loop again or stop. LangGraph expresses this as a graph with a shared state, conditional edges and cycles.

## 2. Conceptual questions

### Exercise 1: Role of each component
1. LangChain
   Answer: A framework of building blocks for LLM applications: a common interface to model providers, prompts, message types, tools, retrievers and output parsers. Good for linear pipelines where the steps are fixed.
2. LangGraph
   Answer: A library on top of LangChain that models an application as a graph of nodes sharing a state, with conditional edges, loops and persistence (checkpointers). Suited to agents that decide what to do next, call tools repeatedly and keep memory.
3. An LLM
   Answer: The neural network that generates text from a prompt (here `gpt-4o-mini`). It is the reasoning part of the agent: it understands the question, classifies it, decides on tool calls and writes the answer.

### Exercise 2: Graph basics
1. What is a node?
   Answer: A Python function that receives the state and returns the keys it updates.
2. What is an edge?
   Answer: A link that says which node runs next. Normal edges are fixed; conditional edges choose the next node at runtime with a routing function.
3. What is `START`?
   Answer: The virtual entry point; its edge decides the first node to run.
4. What is `END`?
   Answer: The virtual exit point; reaching it stops the graph and returns the final state.
5. Why is the state passed between nodes?
   Answer: It is the only shared memory between nodes. Each node reads previous results and adds its own, which keeps nodes independent, makes the data flow explicit, and allows checkpointing and memory.

### Router evaluation
Where did the router fail (if anywhere)? What would make the routing more robust?

Answer: The router depends on a free-text label from the LLM and a substring check, so it can fail on ambiguous questions (e.g. a Python function that computes a derivative), on unexpected spellings ("Mathematics.", "AI", "math"), and the silent fallback to `ai` hides errors. More robust options: structured output with a `Literal`/enum type, validation of the label with logging of fallbacks, few-shot examples for borderline cases, `temperature=0`, and a larger labelled test set with per-category accuracy.

## 3. Design choices (capstone)
- **Classifier → specialized agents**: one node per category (COURSES, PROGRAMMING, MATHEMATICS, ADMINISTRATION, GENERAL), all built by the same `make_agent` factory with a different system prompt.
- **Tools**: a safe calculator based on `ast` (no `eval()`, so no arbitrary code execution) and a date/time tool. Every agent can call them; after the tool runs, control returns to the same agent (`route_after_tools`).
- **Memory**: `MemorySaver` checkpointer with a `thread_id`; different threads have separate memories.
- **Error handling**: empty input → fallback node; unknown label → GENERAL; API error in the classifier → fallback; API error in an agent → apology message; invalid tool calls → `ToolNode(handle_tool_errors=True)` returns the error to the agent instead of crashing.

Diagram: see [`diagrams/README.md`](diagrams/README.md).

## 4. Results
To fill in after the notebook has been run with outputs (routing accuracy, demo queries, memory test, tool trace).

Answer:

## 5. Limitations and possible improvements
- The classifier only looks at the last message, so a follow-up such as "and the second one?" may be misrouted.
- The router uses free text; structured output would make it more reliable.
- The administration and course agents have no access to real university data; a database or document retrieval tool would avoid generic answers.
- Each question costs at least two LLM calls (classifier + agent), which adds latency and cost.

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

Answer: In our run the router made no mistake (6/6 = 100%), but the test set is small and the questions are clear-cut. The router depends on a free-text label from the LLM and a substring check, so it can potentially fail on ambiguous questions (e.g. a Python function that computes a derivative), on unexpected spellings ("Mathematics.", "AI", "math"), and the silent fallback to `ai` hides errors. More robust options: structured output with a `Literal`/enum type, validation of the label with logging of fallbacks, few-shot examples for borderline cases, `temperature=0`, and a larger labelled test set with per-category accuracy.

## 3. Design choices (capstone)
- **Classifier → specialized agents**: one node per category (COURSES, PROGRAMMING, MATHEMATICS, ADMINISTRATION, GENERAL), all built by the same `make_agent` factory with a different system prompt.
- **Tools**: a safe calculator based on `ast` (no `eval()`, so no arbitrary code execution) and a date/time tool. Every agent can call them; after the tool runs, control returns to the same agent (`route_after_tools`).
- **Memory**: `MemorySaver` checkpointer with a `thread_id`; different threads have separate memories.
- **Error handling**: empty input → fallback node; unknown label → GENERAL; API error in the classifier → fallback; API error in an agent → apology message; invalid tool calls → `ToolNode(handle_tool_errors=True)` returns the error to the agent instead of crashing.

Diagram: see [`diagrams/README.md`](diagrams/README.md).

## 4. Results
Model: `gpt-4o-mini`, `temperature=0`, LangGraph 1.2.19, LangChain 1.4.7 (Google Colab).

| Experiment | Result |
|---|---|
| First graph (Part II) | `{'message': 'Hello LangGraph!', 'response': 'Agent received: Hello LangGraph!'}`, as expected |
| Router demo (Part VI) | 3/3 routed correctly (mathematics, programming, artificial_intelligence) |
| Router evaluation | 6/6 = 100% accuracy, 2 to 7.6 s per question |
| Tool agent (Part VIII) | "125 * 38" → calculator called → 4750; "capital of Morocco" → no tool, answers Rabat |
| Safe calculator | `25 * 4 + 10` → 110; `__import__('os')` → Invalid expression |
| Memory (Part IX) | Without checkpointer the name is forgotten; with `MemorySaver` it answers "Your name is Ahmed"; another thread does not know it |

Capstone demo:

| Query | Category | Tool | Result |
|---|---|---|---|
| What topics are usually covered in a machine learning course? | COURSES | none | Structured course outline |
| Write a Python function that computes the factorial of n. | PROGRAMMING | none | Python function with explanation |
| What is 125 * 38 + 17? | MATHEMATICS | calculator | 4767 (correct) |
| How do I request an enrollment certificate? | ADMINISTRATION | none | Generic steps, advises contacting the office |
| What is today's date? | GENERAL | current_datetime | Saturday, October 31, 2026 (correct) |
| (empty question) | EMPTY | none | Fallback message asking for a question |
| My name is Ahmed. / What is my name? | GENERAL | none | "Your name is Ahmed." (memory works) |
| Compute (15 + 27) * 3 using the calculator. | MATHEMATICS | calculator | 126 (correct, full trace shown in the notebook) |

All five categories, both tools, memory and the empty-input error case behave as designed.

Note: the PNG rendering of the tool agent and capstone graphs failed in Colab (the mermaid.ink service returned HTTP 400), so the notebook prints the Mermaid source instead; the rendered diagrams are in [`diagrams/README.md`](diagrams/README.md).

## 5. Limitations and possible improvements
- The classifier only looks at the last message, so a follow-up such as "and the second one?" may be misrouted.
- The router uses free text; structured output would make it more reliable.
- The administration and course agents have no access to real university data; a database or document retrieval tool would avoid generic answers.
- Each question costs at least two LLM calls (classifier + agent), which adds latency and cost.

# Report: Building AI Agents with LangGraph

## 1. Chain vs. agentic workflow
A normal LLM chain always runs the same steps in the same order (prompt, model, parser). In an agentic workflow the model takes decisions while it runs: which branch to take, whether to call a tool, whether to continue or stop. LangGraph lets us write this as a graph with a shared state, conditional edges and loops.

## 2. Conceptual questions

### Exercise 1: Role of each component
1. LangChain
   Answer: The toolbox. One interface for the different LLM providers, plus prompts, messages, tools and parsers. Good when the steps are always the same.
2. LangGraph
   Answer: Built on LangChain. The application becomes a graph of nodes that update a shared state, with edges that can depend on a condition or loop back. The program can then decide at runtime what to do next, call tools several times and keep a memory.
3. An LLM
   Answer: The model that understands the question and generates the text (here gpt-4o-mini). In our agents it classifies the question, decides if a tool is needed and writes the answer.

### Exercise 2: Graph basics
1. What is a node?
   Answer: A function that takes the state and returns the part it changes.
2. What is an edge?
   Answer: It says which node runs next. It can be fixed, or conditional when a function chooses the next node.
3. What is `START`?
   Answer: The entry point of the graph (not a real node); its edge gives the first node to run.
4. What is `END`?
   Answer: The exit point. When we reach it the graph stops and returns the final state.
5. Why is the state passed between nodes?
   Answer: The nodes do not call each other, so the state is the only way to share information. Each node reads what the others wrote and adds its own result. It also makes the run easy to inspect, and it is what the checkpointer saves for the memory.

### Router evaluation
Where did the router fail (if anywhere)? What would make the routing more robust?

Answer: It did not fail in our run (6/6 on the evaluation, 3/3 in the Part VI demo), but the questions are short and clearly in one category. The weak point is that the LLM returns free text, we only check if a word is inside it, and anything unknown silently goes to the `ai` branch. A question like "write a Python function that computes a derivative" could go either way. Improvements: structured output with a `Literal` of the three categories, checking the label and logging when the fallback is used, a few examples in the prompt for borderline cases, and a bigger labelled test set with accuracy per category.

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
| What is 125 * 38 + 17? | MATHEMATICS | calculator expected (only the final answer is printed) | 4767 (correct) |
| How do I request an enrollment certificate? | ADMINISTRATION | none | Generic steps, advises contacting the office |
| What is today's date? | GENERAL | current_datetime (the model cannot know the date otherwise) | Saturday, October 31, 2026 (correct) |
| (empty question) | EMPTY | none | Fallback message asking for a question |
| My name is Ahmed. / What is my name? | GENERAL | none | "Your name is Ahmed." (memory works) |
| Compute (15 + 27) * 3 using the calculator. | (not printed) | calculator | 126 (correct, full trace shown in the notebook) |

All five categories, both tools, memory and the empty-input error case behave as designed.

Note: the PNG rendering of the tool agent and capstone graphs failed in Colab (the mermaid.ink service returned HTTP 400), so the notebook prints the Mermaid source instead; the rendered diagrams are in [`diagrams/README.md`](diagrams/README.md).

## 5. Limitations and possible improvements
- The classifier only looks at the last message, so a follow-up such as "and the second one?" may be misrouted.
- The router uses free text; structured output would make it more reliable.
- The administration and course agents have no access to real university data; a database or document retrieval tool would avoid generic answers.
- Each question costs at least two LLM calls (classifier + agent), which adds latency and cost.

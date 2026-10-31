# Lab: Building AI Agents with LangGraph

PhD lab on agentic workflows with LangChain and LangGraph.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/yassirtl/LangGraph-Lab/blob/main/langgraph_lab.ipynb)

## Files
| Path | Description |
|---|---|
| `langgraph_lab.ipynb` | Notebook with code, outputs and answers |
| `langgraph_lab_Statement.ipynb` | Original lab statement |
| `report.md` | Short report answering the conceptual questions |
| `diagrams/` | Architecture diagrams (Mermaid, rendered on GitHub) |
| `screenshots/` | Screenshots showing execution |

## Contents
| Part | Topic |
|---|---|
| I | Environment setup |
| II | First graph: state, nodes, edges |
| III | LLM node |
| IV | Two-stage workflow (analyzer, answer generator) |
| V-VI | Classifier and conditional routing |
| VII-VIII | Tools (calculator, date/time) and tool-using agent |
| IX | Memory with `MemorySaver` |
| X | Capstone: Intelligent University Assistant |

## Capstone architecture
```
User question -> Classifier -+-> Courses agent --------+
                             +-> Programming agent ----+
                             +-> Mathematics agent ----+-> Tools (calculator, date/time) -> same agent -> Final response
                             +-> Administration agent -+
                             +-> General agent --------+
                             +-> Fallback (empty input / API error)
```

Design choices:
- The calculator uses `ast` instead of `eval()` so only arithmetic can run.
- An unknown classifier label falls back to `GENERAL`, an empty question goes to the fallback node.
- `ToolNode(handle_tool_errors=True)` returns tool errors to the agent instead of crashing the graph.
- After a tool call, control goes back to the agent that asked for it.

## How to run
### Google Colab
1. Open `langgraph_lab.ipynb` in Google Colab (CPU runtime is enough).
2. Add `OPENAI_API_KEY` in the Colab Secrets panel (key icon), or paste it when asked.
3. Run the cells in order.

### Locally with Jupyter
```bash
python3 -m venv .venv && source .venv/bin/activate
pip install jupyter pandas langgraph langchain langchain-openai langchain-community
export OPENAI_API_KEY=...
jupyter notebook langgraph_lab.ipynb
```

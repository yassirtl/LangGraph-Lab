# Architecture diagrams

Exported from the notebook with `graph.get_graph().draw_mermaid()`.

## Router (Parts V-VI)

```mermaid
---
config:
  flowchart:
    curve: linear
---
graph TD;
	__start__([<p>__start__</p>]):::first
	classifier(classifier)
	math(math)
	programming(programming)
	ai(ai)
	__end__([<p>__end__</p>]):::last
	__start__ --> classifier;
	classifier -.-> ai;
	classifier -.-> math;
	classifier -.-> programming;
	ai --> __end__;
	math --> __end__;
	programming --> __end__;
	classDef default fill:#f2f0ff,line-height:1.2
	classDef first fill-opacity:0
	classDef last fill:#bfb6fc
```

## Tool-using agent (Part VIII)

```mermaid
---
config:
  flowchart:
    curve: linear
---
graph TD;
	__start__([<p>__start__</p>]):::first
	agent(agent)
	tools(tools<hr/><small><em>__tools = calculator, current_datetime</em></small>)
	__end__([<p>__end__</p>]):::last
	__start__ --> agent;
	agent -.-> __end__;
	agent -.-> tools;
	tools --> agent;
	classDef default fill:#f2f0ff,line-height:1.2
	classDef first fill-opacity:0
	classDef last fill:#bfb6fc
```

## Capstone: Intelligent University Assistant (Part X)

```mermaid
---
config:
  flowchart:
    curve: linear
---
graph TD;
	__start__([<p>__start__</p>]):::first
	classifier(classifier)
	fallback(fallback)
	tools(tools<hr/><small><em>__tools = calculator, current_datetime</em></small>)
	courses_agent(courses_agent)
	programming_agent(programming_agent)
	mathematics_agent(mathematics_agent)
	administration_agent(administration_agent)
	general_agent(general_agent)
	__end__([<p>__end__</p>]):::last
	__start__ --> classifier;
	administration_agent -.-> __end__;
	administration_agent -.-> tools;
	classifier -.-> administration_agent;
	classifier -.-> courses_agent;
	classifier -.-> fallback;
	classifier -.-> general_agent;
	classifier -.-> mathematics_agent;
	classifier -.-> programming_agent;
	courses_agent -.-> __end__;
	courses_agent -.-> tools;
	general_agent -.-> __end__;
	general_agent -.-> tools;
	mathematics_agent -.-> __end__;
	mathematics_agent -.-> tools;
	programming_agent -.-> __end__;
	programming_agent -.-> tools;
	tools -.-> administration_agent;
	tools -.-> courses_agent;
	tools -.-> general_agent;
	tools -.-> mathematics_agent;
	tools -.-> programming_agent;
	fallback --> __end__;
	classDef default fill:#f2f0ff,line-height:1.2
	classDef first fill-opacity:0
	classDef last fill:#bfb6fc
```

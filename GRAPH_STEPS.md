# Detailed Graph Structure Explanation with Code Snippets

## Imports

The code uses several important libraries:

```python
import os
import pandas as pd
from io import StringIO
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END
from tavily import TavilyClient
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.pydantic_v1 import BaseModel
from langgraph.checkpoint.memory import MemorySaver
from typing import List
import streamlit as st
```

Key libraries:
- `langgraph`: For creating the state graph
- `langchain_openai`: For interacting with OpenAI's ChatGPT
- `tavily`: For web searches
- `streamlit`: For creating the user interface

## State Definition

The workflow state is defined using a Pydantic model:

```python
class AgentState(BaseModel):
    task: str
    competitors: List[str]
    csv_file: str
    financial_data: str = ""
    analysis: str = ""
    competitor_data: str = ""
    comparison: str = ""
    feedback: str = ""
    report: str = ""
    content: List[str]
    revision_number: int
    max_revisions: int
```

This state is passed between nodes and updated throughout the workflow.

## Graph Structure

The graph is created using `StateGraph`:

```python
flow = StateGraph(AgentState)
```

### Nodes

Each node in the graph is a function that takes and returns an `AgentState`. Here's an example of the `gather_financials_node`:

```python
def gather_financials_node(state: AgentState):
    csv_file = state.csv_file
    df = pd.read_csv(StringIO(csv_file))
    financial_data_to_string = df.to_string(index=False)
    combined_content = f"{state.task}\nHere is the financial data:\n{financial_data_to_string}"
    messages = [
        SystemMessage(content=GATHER_FINANCIALS_PROMPT),
        HumanMessage(content=combined_content),
    ]
    response = llm_model.invoke(messages)
    financial_data = response.content
    return AgentState(**{**state.dict(), "financial_data": financial_data})
```

Nodes are added to the graph like this:

```python
flow.add_node("gather_financials", gather_financials_node)
```

### Edges

Edges define the flow between nodes:

```python
flow.add_edge("gather_financials", "analyze_data")
flow.add_edge("analyze_data", "research_competitors")
# ... more edges ...
```

### Conditional Logic

The graph includes conditional logic to determine whether to continue iterations:

```python
def should_continue(state: AgentState):
    if state.revision_number > state.max_revisions:
        return END
    return "collect_feedback"

flow.add_conditional_edges(
    "compare_performance",
    should_continue,
    {END: END, "collect_feedback": "collect_feedback"},
)
```

## Graph Compilation and Visualization

The graph is compiled and can be visualized:

```python
graph = flow.compile(checkpointer=memory)
graph.get_graph().draw_mermaid_png(output_file_path="graph.png")
```

## Streamlit Integration

The graph is integrated with Streamlit for user interaction:

```python
def main():
    st.title("Financial Report Generator")
    # ... Streamlit UI code ...
    if st.button("Generate Report") and uploaded_file is not None:
        # ... Initialize state ...
        for s in graph.stream(initial_state, thread):
            st.write(s)
        # ... Display final report ...

if __name__ == "__main__":
    main()
```

## Workflow Steps

1. **Gather Financials**: Extracts data from uploaded CSV.
2. **Analyze Data**: Analyzes financial data using LLM.
3. **Research Competitors**: Uses Tavily API to gather competitor info.
4. **Compare Performance**: Compares company to competitors.
5. **Collect Feedback**: Generates feedback on the comparison.
6. **Research Critique**: Performs additional research based on feedback.
7. **Write Report**: Generates final financial report.

## Key Features

1. **LLM Integration**: Uses OpenAI's ChatGPT for analysis and text generation.
2. **Web Search**: Incorporates Tavily API for competitor research.
3. **Iterative Refinement**: Includes a feedback loop for improving analysis.
4. **State Management**: Uses `AgentState` to maintain context across nodes.
5. **Checkpointing**: Implements `MemorySaver` for potential workflow resumption.
6. **Visualization**: Can generate a visual representation of the graph.
7. **User Interface**: Streamlit integration for easy user interaction.

This graph structure creates a comprehensive, AI-driven financial analysis and report generation system, combining LLM capabilities with web-based research in an iterative process.

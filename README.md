# 🚲🗽Bike-sharing System in New York City
![](https://img.shields.io/badge/python-3.10%2B-blue?logo=Python)
![Gurobi Version](https://img.shields.io/badge/gurobi-11.0.0-blue?logo=gurobi&logoColor=red)

⚡ Optimization in Balance between Demand Capturing and Cost Deployment ⚡

> [!NOTE]
> This project is built upon previous research done by scholars at HKUST. Click [here](https://doi.org/10.1145/2820783.2820837.).

## Overview

Bike-sharing systems have emerged as a popular and sustainable mode of transportation in urban environments, offering an efficient means of travel for commuters and tourists alike. Understanding the dynamics of bike-sharing demand is crucial for optimizing resource allocation to ensure efficient service delivery and customer satisfaction. Leveraging the dataset and preprocessing techniques established by prior researchers, our study aims to develop a simplified yet robust model capable of capturing the nuances of demand variations and customer patterns. Furthermore, this study goes beyond mere prediction by integrating an optimization model to determine the optimal number of bikes to deploy. Focusing on the week of August 16th to August 22nd, we aim to devise a strategy for resource allocation that maximizes efficiency and minimizes operational costs. In essence, this project bridges academic research and practical
applications, offering a comprehensive framework for understanding and managing the transitional demands of bike-sharing stations in New York City. By leveraging insights from previous research and employing advanced analytical techniques, we aspire to contribute to the ongoing efforts to enhance the sustainability and accessibility of urban transportation systems.

### Key Techniques Used

- **Network Analysis**: Implement loops and conditionals in your apps.
- **K-NN**: Automatically save state after each step in the graph. Pause and resume the graph execution at any point to support error recovery, human-in-the-loop workflows, time travel, and more.
- **Regression**: Interrupt graph execution to approve or edit next action planned by the agent.
- **Gurobi Optimization**: Stream outputs as they are produced by each node (including token streaming).

### Step-by-step Breakdown

1. <details>
    <summary>Network Analysis</summary>

    - we used the package `networkx` as the primary tool for network analysis. **NOTE:** we need make sure the model knows that it has these tools available to call. We can do this by converting the LangChain tools into the format for OpenAI tool calling using the `.bind_tools()` method.
    - we define the tools we want to use - a web search tool in our case. It is really easy to create your own tools - see documentation here on how to do that [here](https://python.langchain.com/docs/modules/agents/tools/custom_tools).
   </details>

2. <details>
    <summary>Initialize graph with state.</summary>

    - we initialize graph (`StateGraph`) by passing state schema (in our case `MessagesState`)
    - `MessagesState` is a prebuilt state schema that has one attribute -- a list of LangChain `Message` objects, as well as logic for merging the updates from each node into the state
   </details>

3. <details>
    <summary>Define graph nodes.</summary>

    There are two main nodes we need:

      - The `agent` node: responsible for deciding what (if any) actions to take.
      - The `tools` node that invokes tools: if the agent decides to take an action, this node will then execute that action.
   </details>

4. <details>
    <summary>Define entry point and graph edges.</summary>

      First, we need to set the entry point for graph execution - `agent` node.

      Then we define one normal and one conditional edge. Conditional edge means that the destination depends on the contents of the graph's state (`MessageState`). In our case, the destination is not known until the agent (LLM) decides.

      - Conditional edge: after the agent is called, we should either:
        - a. Run tools if the agent said to take an action, OR
        - b. Finish (respond to the user) if the agent did not ask to run tools
      - Normal edge: after the tools are invoked, the graph should always return to the agent to decide what to do next
   </details>

5. <details>
    <summary>Compile the graph.</summary>

    - When we compile the graph, we turn it into a LangChain [Runnable](https://python.langchain.com/v0.2/docs/concepts/#runnable-interface), which automatically enables calling `.invoke()`, `.stream()` and `.batch()` with your inputs
    - We can also optionally pass checkpointer object for persisting state between graph runs, and enabling memory, human-in-the-loop workflows, time travel and more. In our case we use `MemorySaver` - a simple in-memory checkpointer
    </details>

6. <details>
   <summary>Execute the graph.</summary>

    1. LangGraph adds the input message to the internal state, then passes the state to the entrypoint node, `"agent"`.
    2. The `"agent"` node executes, invoking the chat model.
    3. The chat model returns an `AIMessage`. LangGraph adds this to the state.
    4. Graph cycles the following steps until there are no more `tool_calls` on `AIMessage`:

        - If `AIMessage` has `tool_calls`, `"tools"` node executes
        - The `"agent"` node executes again and returns `AIMessage`

    5. Execution progresses to the special `END` value and outputs the final state.
    And as a result, we get a list of all our chat messages as output.
   </details>




# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: Agent State](chapter_01.md)

- **Example 1** (python): backend/src/agent/state.py
- **Example 2** (python): Simplified example of initial state setup
- **Example 3** (python): Example of a node updating the search_query
- **Example 4** (python): backend/src/agent/state.py
- **Example 5** (python): class OverallState(TypedDict):

## [Chapter 2: Agent Workflow Nodes](chapter_02.md)

- **Example 1** (python): From backend/src/agent/graph.py
- **Example 2** (python): From backend/src/agent/graph.py
- **Example 3** (python): From backend/src/agent/graph.py
- **Example 4** (python): From backend/src/agent/graph.py
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 3: Agent Tools and Schemas](chapter_03.md)

- **Example 1** (python): from pydantic import BaseModel, Field
- **Example 2** (python): This dictionary represents what an LLM might generate
- **Example 3** (python): from pydantic import BaseModel, Field
- **Example 4** (python): LLM's reflection on current research
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 4: Agent Prompts](chapter_04.md)

- **Example 1** (python): backend/src/agent/prompts.py
- **Example 2** (python): Example: Constructing a system prompt
- **Example 3** (python): Example: Incorporating tool descriptions into a prompt
- **Example 4** (mermaid): graph TD

## [Chapter 5: Agent Graph Orchestration](chapter_05.md)

- **Example 1** (python): from langgraph.graph import StateGraph
- **Example 2** (python): from backend.src.agent.graph import generate_qu...
- **Example 3** (python): from langgraph.channels import Send
- **Example 4** (python): from backend.src.agent.graph import evaluate_re...
- **Example 5** (python): Set the entry point of the graph
- **Example 6** (python): The 'continue_to_web_research' node sends tasks,
- **Example 7** (python): def route_research(state: OverallState) -> str:
- **Example 8** (python): From backend/src/agent/graph.py
- **Example 9** (mermaid): sequenceDiagram

## [Chapter 6: Agent Configuration](chapter_06.md)

- **Example 1** (python): backend/src/agent/configuration.py
- **Example 2** (python): from backend.src.agent.configuration import Con...
- **Example 3** (python): Example: How an agent component might use the configuration
- **Example 4** (python): backend/src/agent/configuration.py
- **Example 5** (text): This diagram illustrates how the `Configuration...

## [Chapter 7: Full-stack Application Structure](chapter_07.md)

- **Example 1** (python): From backend/src/agent/app.py
- **Example 2** (python): From backend/src/agent/app.py (continued)
- **Example 3** (mermaid): sequenceDiagram


## How to Use Code Examples

1. Each code example is designed to be self-contained
2. Copy the code and run it in your development environment
3. Modify the examples to understand how they work
4. Refer back to the chapter context for detailed explanations

## Code Conventions

- All examples follow the project's coding standards
- Comments are included to explain complex logic
- Error handling is included where appropriate
- Examples progress from simple to complex within each chapter

# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: Communication: Sharing Data Between Nodes](chapter_01.md)

- **Example 1** (python): shared_store.py (Conceptual Representation)
- **Example 2** (python): Inside PromptGeneratorNode's execution logic
- **Example 3** (python): Inside LLMCallNode's execution logic
- **Example 4** (python): Inside ResponseParserNode's execution logic
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 2: Nodes: The Smallest Building Blocks](chapter_02.md)

- **Example 1** (python): pocket_flow/core/node.py (Conceptual Base Class)
- **Example 2** (python): my_app/nodes.py
- **Example 3** (python): main.py (Simulated Node execution)
- **Example 4** (text): --- Running FeatureExtractionNode ---
- **Example 5** (mermaid): sequenceDiagram
- **Example 6** (mermaid): graph TD

## [Chapter 3: Flows: Orchestrating Nodes with Actions](chapter_03.md)

- **Example 1** (python): from pocket_flow import Node, SharedStore, Action
- **Example 2** (python): from pocket_flow import Flow
- **Example 3** (python): from pocket_flow import SharedStore
- **Example 4** (text): *Explanation*: `SummarizeTextNode` takes `input...
- **Example 5** (text): *Explanation*: Both `KeywordExtractNode` and `O...
- **Example 6** (text): *Explanation*:
- **Example 7** (text): *Expected Output for Short Text*: You'll see `[...

## [Chapter 4: What is Pocket Flow?](chapter_04.md)

- **Example 1** (python): Conceptual Node definitions
- **Example 2** (python): from pocket_flow import Flow, Action, SharedSto...
- **Example 3** (python): initial_store = SharedStore(data={"user_query":...
- **Example 4** (mermaid): sequenceDiagram

## [Chapter 5: Utility Functions: Connecting to the Real World](chapter_05.md)

- **Example 1** (python): File: utilities/web_search.py
- **Example 2** (python): File: nodes/search_node.py
- **Example 3** (python): File: my_app_flow.py
- **Example 4** (mermaid): sequenceDiagram

## [Chapter 6: Batch Processing: Efficiently Handling Large Inputs](chapter_06.md)

- **Example 1** (python): from pocket_flow import Node, SharedStore, Flow
- **Example 2** (python): from pocket_flow import Node, SharedStore, Flow...
- **Example 3** (python): from pocket_flow import Node, SharedStore, Flow...
- **Example 4** (mermaid): sequenceDiagram

## [Chapter 7: Agents & Workflows: Advanced LLM Application Patterns](chapter_07.md)

- **Example 1** (python): Assuming a basic LLMUtility is available
- **Example 2** (python): Simple placeholder nodes for demonstration
- **Example 3** (python): class SummarizeNode(Node):
- **Example 4** (mermaid): sequenceDiagram
- **Example 5** (mermaid): sequenceDiagram


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

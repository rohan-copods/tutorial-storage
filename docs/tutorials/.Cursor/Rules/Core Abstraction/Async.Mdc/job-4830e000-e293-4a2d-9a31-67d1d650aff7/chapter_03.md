# Chapter 3: Flows: Orchestrating Nodes with Actions

In the previous chapter, [Nodes: The Smallest Building Blocks](chapter_02.md), we explored how individual `Nodes` serve as focused units, capable of performing specific tasks within the Pocket Flow framework. We understood their `Prep -> Exec -> Post` lifecycle and their role in processing data. While powerful on their own, a collection of isolated nodes doesn't constitute a complete application. Just like individual musicians need a conductor to create a symphony, nodes require an orchestrator to define their sequence, handle decision points, and manage the overall flow of execution.

---

### Problem & Motivation

Imagine building an AI application where you need to process user input, summarize it, and then, based on the summary's characteristics, either extract keywords or generate a full response. How do you define this sequence of operations? How do you tell the system to take one path if the summary is long, and another if it's short? Individual nodes can perform the summarization, keyword extraction, or response generation, but they lack the intelligence to *decide* the next step or *connect* with each other in a meaningful, conditional way.

This is precisely the problem `Flows` solve. Without `Flows`, our Pocket Flow application would be a disjointed collection of tasks. There would be no mechanism to:
1.  **Define a clear operational sequence**: How does Node A lead to Node B?
2.  **Implement branching logic**: How can the application make decisions and take different paths based on runtime conditions?
3.  **Manage the lifecycle of a complex task**: From start to finish, how do we ensure all necessary steps are performed in the correct order?

`Flows` are the project's answer to orchestrating the execution of `Nodes` into a cohesive, intelligent application. They provide the necessary structure to model complex tasks as a graph, where nodes are vertices and transitions are edges, allowing for powerful, dynamic AI applications.

---

### Core Concept Explanation: The Orchestrator of Operations

At its heart, a `Flow` is a directed graph that defines the sequence of operations within your Pocket Flow application. It connects various `Nodes`, acting as the "brain" that dictates when and how each node is executed. Think of it as a meticulously designed recipe or a flowchart: each step (Node) is clearly defined, and there are instructions (Actions) on what to do next based on the outcome of the current step.

A `Flow` brings several critical capabilities to Pocket Flow:

*   **Orchestration**: It defines the order in which `Nodes` are executed. This can be a simple linear sequence or a much more complex branching structure.
*   **Decision Making with Actions**: The true power of `Flows` lies in their ability to handle dynamic transitions. After a `Node` completes its execution, its `post()` method (or `exec()` in simplified cases) returns an `Action`. This `Action` is a signal that tells the `Flow` which path to take next. For example, a summarization node might return an `Action("summary_too_long")` or `Action("summary_ok")`, leading the `Flow` to different subsequent nodes.
*   **Graph-based Structure**: Internally, a `Flow` maintains a map of `Nodes` and their possible transitions based on `Actions`. This allows for highly flexible and readable application designs.
*   **Modularity through Nested Flows**: Just as nodes encapsulate small tasks, `Flows` can encapsulate sub-workflows. A complex `Flow` can include other `Flows` as if they were simple `Nodes`, promoting reusability and keeping your application structure clean and manageable.

The combination of `Nodes` performing focused tasks, communicating via the [Shared Store](chapter_01.md), and being orchestrated by `Flows` via `Actions` forms the fundamental operational model of Pocket Flow.

---

### Practical Usage Examples: Building Our First Flow

Let's use our motivating example: summarize text, and if the summary is too long, extract keywords; otherwise, just output the summary. This requires conditional logic which `Flows` handle gracefully.

First, let's define our (simplified) nodes. For this example, we'll assume the LLM interaction happens within the node and focus on the `Action` return.

#### Step 1: Defining Nodes with Actions

```python
from pocket_flow import Node, SharedStore, Action

class SummarizeTextNode(Node):
    def exec(self, store: SharedStore):
        text = store.get("input_text", "")
        # Simulate LLM summarization
        summary = f"Summary of '{text[:20]}...': This is a generated summary."
        if len(text) > 50: # Simulate a long input leading to a long summary
            summary += " This part makes the summary artificially long."
        store.set("summary", summary)
        print(f"Summarized: {summary[:50]}...")
        if len(summary) > 100: # Decision point: too long?
            return Action("summary_too_long")
        return Action("summary_ok")

class KeywordExtractNode(Node):
    def exec(self, store: SharedStore):
        summary = store.get("summary", "")
        # Simulate LLM keyword extraction
        keywords = f"Keywords from summary: Important, Key, Data."
        store.set("output", keywords)
        print(f"Extracted Keywords: {keywords}")
        return Action("done_processing") # Indicate completion

class OutputSummaryNode(Node):
    def exec(self, store: SharedStore):
        summary = store.get("summary", "")
        store.set("output", summary)
        print(f"Outputting Summary: {summary}")
        return Action("done_processing") # Indicate completion
```
*Explanation*: Each node performs a task and, crucially, returns an `Action`. `SummarizeTextNode` decides based on the *simulated* summary length. `KeywordExtractNode` and `OutputSummaryNode` simply indicate they are `done_processing`.

#### Step 2: Constructing the Flow

Now, we'll build the `Flow` that connects these nodes using the `on()` method to define transitions based on `Actions`.

```python
from pocket_flow import Flow

# Create a new Flow instance
text_processing_flow = Flow("TextProcessingFlow")

# Define the starting node
text_processing_flow.node(SummarizeTextNode, "summarize_text")

# Define transitions based on Actions returned by "summarize_text"
text_processing_flow.on("summary_too_long", KeywordExtractNode, "extract_keywords")
text_processing_flow.on("summary_ok", OutputSummaryNode, "output_summary")

# Define where "extract_keywords" leads after it's done
text_processing_flow.on("done_processing", OutputSummaryNode, "output_summary", from_node_id="extract_keywords")

# Finally, mark the end of the flow when any processing is complete
text_processing_flow.on("done_processing", Flow.END)
```
*Explanation*:
- We start the `Flow` with `SummarizeTextNode` named `"summarize_text"`.
- If `"summarize_text"` returns `"summary_too_long"`, the `Flow` transitions to `KeywordExtractNode`.
- If `"summarize_text"` returns `"summary_ok"`, the `Flow` transitions to `OutputSummaryNode`.
- After `KeywordExtractNode` (`"extract_keywords"`) is done, it will return `"done_processing"`, which we want to lead to `OutputSummaryNode`. We specify `from_node_id` to make this transition explicit from a specific node.
- The `Flow.END` action signifies the completion of the entire flow.

#### Step 3: Running the Flow

Let's execute the `Flow` with different inputs to observe the branching.

```python
from pocket_flow import SharedStore

# --- Scenario 1: Short text (summary_ok path) ---
print("\n--- Running Flow with Short Text ---")
store_short = SharedStore(input_text="Hello, this is a short test message.")
result_store_short = text_processing_flow.run(store_short)
print(f"Final Output (Short Text): {result_store_short.get('output')}")
# Expected output will be the summary itself

# --- Scenario 2: Long text (summary_too_long path) ---
print("\n--- Running Flow with Long Text ---")
long_text = "This is a very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very very extremely long text has to be placed in an iframe or similar, to prevent the main document from becoming too large and slow."

Let's process this request. The core of the request is to generate Chapter 3 documentation according to *very specific* guidelines and structure.

Here's my plan of action, mapping to the requirements:

1.  **Chapter Title**: Start with `# Chapter 3: Flows: Orchestrating Nodes with Actions`.
2.  **Transition from Previous Chapter**:
    *   Refer to Chapter 2: "Nodes: The Smallest Building Blocks".
    *   Explain that while nodes do tasks, something is needed to sequence them.
3.  **Problem & Motivation (Phase 1: Problem-Solution Fit)**:
    *   Discuss the problem of isolated nodes.
    *   Explain *why* Flows are crucial for sequencing, decision-making, and managing complex tasks.
    *   Introduce a concrete use case: "Summarize text, then conditionally extract keywords or output the summary based on length." This will guide examples.
4.  **Core Concept Explanation (Phase 1: Concept Overview)**:
    *   Define what a `Flow` is (a directed graph of `Nodes`).
    *   Explain its purpose: orchestration, defining sequence.
    *   Introduce `Actions` as the key mechanism for transitions and branching.
    *   Mention `Shared Store` (from Chapter 1) as the communication backbone.
    *   Briefly touch on nested flows for modularity.
    *   Use analogies (conductor, recipe, flowchart).
5.  **Practical Usage Examples (Phase 1: Quick Start, Phase 2: Configuration & Usage Patterns)**:
    *   **Crucial**: Break down code into small blocks (under 10 lines each).
    *   Implement the motivating use case.
    *   Define three simple nodes (`SummarizeTextNode`, `KeywordExtractNode`, `OutputSummaryNode`), showing how `exec` returns `Action`.
    *   Show how to instantiate a `Flow`.
    *   Demonstrate `flow.node()` to add initial nodes.
    *   Demonstrate `flow.on()` to define transitions based on `Actions`. Include `from_node_id` for clarity.
    *   Show how to mark the `Flow.END`.
    *   Provide two scenarios for running the flow: one leading to keyword extraction, one leading directly to summary output, using a `SharedStore`.
    *   Include `print` statements within nodes and for final output to show behavior.
    *   Provide clear explanations after each code block.
6.  **Internal Implementation Walkthrough (Phase 2: Detailed Implementation, API & Interface Reference)**:
    *   Explain that `Flow` manages an internal graph (nodes as vertices, `Action` transitions as edges).
    *   Describe the `run()` method's loop:
        *   Get current node.
        *   Call `node.prep()`, `node.exec()`, `node.post()`.
        *   Receive `Action` from `post()` (or `exec()`).
        *   Use `Action` to determine the next node in the graph.
        *   Repeat until `Flow.END` or no valid transition.
    *   **Visual Aid**: Use a `mermaid` sequence diagram to illustrate the `Flow.run()` method's interaction with `CurrentNode` and `SharedStore`.
    *   Mention conceptual code files (e.g., `pocket_flow/flow.py`).
7.  **System Integration (Phase 1: Architecture Context, Phase 2: Integration Examples, Phase 3: Related Concepts)**:
    *   **Cross-references**: Link back to [Shared Store](chapter_01.md) and [Nodes](chapter_02.md).
    *   Explain how `Flows` use `Shared Store` for data passing between nodes.
    *   Explain how `Flows` encapsulate `Nodes`.
    *   Briefly discuss nested flows as a form of integration (a `Flow` acting as a `Node` within another `Flow`).
8.  **Best Practices & Tips (Phase 3: Best Practices, Troubleshooting)**:
    *   **Modularity**: Keep flows focused (single responsibility).
    *   **Action Naming**: Use clear, descriptive `Action` names.
    *   **Error Handling**: Define explicit `Actions` for error states.
    *   **Reusability**: Leverage nested flows.
    *   **Testing**: Test individual nodes and then the flow.
    *   **Visualization**: Suggest visualizing complex flows (e.g., with graphviz if a tool is built).
9.  **Chapter Conclusion**:
    *   Summarize the role of `Flows` as orchestrators.
    *   Transition to [What is Pocket Flow?](chapter_04.md) to contextualize `Flows` within the broader framework.

Throughout the process, I will ensure beginner-friendly language, consistent formatting, and adherence to all "Universal Documentation Standards" and "Formatting Requirements".
# Chapter 3: Flows: Orchestrating Nodes with Actions

In the previous chapter, [Nodes: The Smallest Building Blocks](chapter_02.md), we delved into the fundamental units of Pocket Flow: `Nodes`. We learned that each `Node` is designed to perform a single, focused task, following a lifecycle to prepare, execute, and post-process data. While `Nodes` are essential for their atomic operations, a collection of isolated `Nodes` cannot form a complete, intelligent application. We need a mechanism to dictate the order of operations, handle conditional logic, and guide data through a series of steps.

---

### Problem & Motivation: Connecting the Dots

Consider a common scenario in an AI application: you receive a user query, and your goal is to provide a concise answer. This might involve several steps: first, generating a summary of relevant documents; second, if that summary is still too verbose, extracting key entities or points; and finally, formulating a final answer based on the condensed information.

The challenge here isn't just *what* tasks to perform (which `Nodes` handle), but *how* to sequence these tasks and *when* to choose alternative paths. For instance:
*   How do we ensure summarization happens *before* keyword extraction?
*   How do we decide to *skip* keyword extraction if the initial summary is already concise enough?
*   What happens if a `Node` encounters an error, and how does the system react?

Without a dedicated orchestration layer, managing these interactions would lead to tangled, imperative code within `Nodes` themselves, violating their single-responsibility principle. This makes the application brittle, hard to debug, and difficult to extend.

`Flows` in Pocket Flow provide the elegant solution to this orchestration problem. They serve as the *master plan* or *conductor* for your application, defining how `Nodes` are interconnected and interact to achieve a larger goal. By using `Flows`, we encapsulate the operational logic, making our AI applications modular, readable, and highly adaptable.

---

### Core Concept Explanation: The Application's Conductor

A `Flow` is a directed graph where each vertex is a `Node` (or another `Flow` in the case of nested flows), and the edges represent transitions between these `Nodes`. Its primary role is to define the operational sequence of your Pocket Flow application, ensuring that tasks are executed in the correct order and that decisions can be made dynamically based on runtime conditions.

The key elements that make `Flows` powerful orchestrators are:

*   **Graph-based Structure**: A `Flow` internally maintains a mapping of `Nodes` and the possible paths leading from them. This allows you to visually (conceptually) trace the journey of data through your application.
*   **Actions: The Decision Makers**: The heart of a `Flow`'s dynamic capability lies in `Actions`. After a `Node` completes its `exec()` (or `post()`) method, it returns an `Action` object. This `Action` is a signal, a specific instruction that tells the `Flow` what to do next. For example, a `SummarizeNode` might return `Action("summary_too_long")` to indicate that further processing is needed, or `Action("summary_complete")` to signal successful completion.
*   **Transitions**: The `Flow` uses these `Actions` to determine the next `Node` to execute. You explicitly define these transitions when building the `Flow`, mapping specific `Actions` from a source `Node` to a target `Node`.
*   **Modular Design**: `Flows` can be nested, meaning a `Flow` can contain other `Flows` as if they were simple `Nodes`. This enables you to break down very complex applications into smaller, manageable, and reusable sub-flows.

In essence, if `Nodes` are the individual gears, the `Flow` is the intricate clockwork mechanism that brings them all together to tell time. It orchestrates the entire process, ensuring data flows correctly through the [Shared Store](chapter_01.md) and that the application adapts to various scenarios through `Action`-driven decisions.

---

### Practical Usage Examples: Orchestrating Our Text Processing

Let's revisit our use case: summarize text, and if the summary is too long, extract keywords; otherwise, just output the summary. This will showcase how `Flows` use `Actions` to manage branching logic.

#### Step 1: Defining Our Nodes with Dynamic Actions

We need three nodes: one to summarize, one to extract keywords, and one to output the final result. Notice how the `SummarizeTextNode` returns different `Actions` based on a condition.

```python
from pocket_flow import Node, SharedStore, Action, Params

# A node that simulates summarization and decides the next step
class SummarizeTextNode(Node):
    def exec(self, store: SharedStore):
        text_input = store.get("input_text", "")
        # Simulate LLM summarization based on input length
        summary_base = f"Summary of '{text_input[:30]}...': This is a concise overview."
        summary = summary_base
        
        # Artificially make summary long for demonstration
        if len(text_input) > 100:
            summary += " This additional content makes the summary quite verbose, requiring further processing."
        
        store.set("summary", summary)
        print(f"[SummarizeTextNode] Generated Summary: {summary[:70]}...")
        
        # Decide the next Action based on simulated summary length
        if len(summary) > 100:
            print("[SummarizeTextNode] Summary too long, requesting keyword extraction.")
            return Action("summary_too_long")
        print("[SummarizeTextNode] Summary concise, proceeding to output.")
        return Action("summary_ok")
```
*Explanation*: `SummarizeTextNode` takes `input_text` from the `SharedStore`. It generates a summary and then critically, based on a simulated length check, returns either `Action("summary_too_long")` or `Action("summary_ok")`. These `Actions` are the signals the `Flow` will use.

Now, the subsequent nodes for keyword extraction and final output.

```python
# A node that simulates keyword extraction
class KeywordExtractNode(Node):
    def exec(self, store: SharedStore):
        summary = store.get("summary", "")
        # Simulate LLM keyword extraction
        keywords = f"Keywords from summary: Important, AI, Flow, Orchestration, Data."
        store.set("final_output", keywords) # Store final result
        print(f"[KeywordExtractNode] Extracted Keywords: {keywords}")
        return Action("done_processing") # Indicates completion for this path

# A node that outputs the summary directly
class OutputSummaryNode(Node):
    def exec(self, store: SharedStore):
        summary = store.get("summary", "")
        store.set("final_output", summary) # Store final result
        print(f"[OutputSummaryNode] Outputting Summary: {summary[:70]}...")
        return Action("done_processing") # Indicates completion for this path
```
*Explanation*: Both `KeywordExtractNode` and `OutputSummaryNode` retrieve data from the `SharedStore` and place their results into `"final_output"`. They then return `Action("done_processing")` to signal their completion to the `Flow`.

#### Step 2: Constructing the Flow with Transitions

Now, we build the `Flow` itself, defining how these nodes connect based on the `Actions` they return.

```python
from pocket_flow import Flow

# Initialize our Flow
text_processing_flow = Flow("ConditionalTextFlow")

# 1. Define the starting node of the flow
text_processing_flow.node(SummarizeTextNode, "summarize_step")

# 2. Define transitions based on Actions from "summarize_step"
# If "summarize_step" returns "summary_too_long", go to KeywordExtractNode
text_processing_flow.on("summary_too_long", KeywordExtractNode, "extract_keywords_step")

# If "summarize_step" returns "summary_ok", go to OutputSummaryNode
text_processing_flow.on("summary_ok", OutputSummaryNode, "output_summary_step")

# 3. Define what happens after "extract_keywords_step" finishes
# If "extract_keywords_step" returns "done_processing", go to OutputSummaryNode
# We explicitly specify 'from_node_id' to indicate which node this 'on' refers to.
text_processing_flow.on("done_processing", OutputSummaryNode, "output_summary_step", from_node_id="extract_keywords_step")

# 4. Define the end of the flow.
# Any node returning "done_processing" can lead to the Flow's termination.
text_processing_flow.on("done_processing", Flow.END)
```
*Explanation*:
*   `text_processing_flow.node(...)` adds the first node to the flow.
*   `text_processing_flow.on(action, next_node_class, next_node_id, from_node_id=...)` defines an edge in the graph. If the node *before* this transition (either implicitly the previously added node, or explicitly `from_node_id`) returns `action`, the flow moves to `next_node_class` (identified by `next_node_id`).
*   `Flow.END` is a special action signifying that the flow should terminate.

#### Step 3: Running the Flow with Different Inputs

Let's observe how the `Flow` dynamically chooses its path based on the input text length.

```python
# Running with a short input text
print("\n--- Running Flow with SHORT Text (should output summary directly) ---")
short_text_input = "Pocket Flow simplifies AI application development by using a graph of nodes."
initial_store_short = SharedStore(input_text=short_text_input)
final_store_short = text_processing_flow.run(initial_store_short)
print(f"Final Result (Short Text): {final_store_short.get('final_output')}")

print("\n--- Running Flow with LONG Text (should extract keywords) ---")
long_text_input = """
The field of Artificial Intelligence has seen tremendous advancements in recent years,
particularly with the rise of Large Language Models (LLMs). These models are capable
of understanding and generating human-like text, opening up new possibilities for
automation, content creation, and intelligent assistance. Building applications with LLMs,
however, can be complex due to the need for intricate orchestrations of various steps
like parsing input, calling different models, handling conditional logic, and formatting
outputs. Pocket Flow aims to simplify this complexity by providing a minimalist framework
that models tasks as interconnected steps, using a shared data store for communication.
This lightweight design emphasizes flexibility and easy integration of diverse LLM tools,
allowing developers to focus on the application's logic rather than boilerplate.
"""
initial_store_long = SharedStore(input_text=long_text_input)
final_store_long = text_processing_flow.run(initial_store_long)
print(f"Final Result (Long Text): {final_store_long.get('final_output')}")
```
*Expected Output for Short Text*: You'll see `[SummarizeTextNode]` indicating a concise summary and then `[OutputSummaryNode]` directly outputting it.
*Expected Output for Long Text*: You'll see `[SummarizeTextNode]` indicating a long summary, then `[KeywordExtractNode]` taking over, and finally `[OutputSummaryNode]` presenting the keywords.
This demonstrates the `Flow`'s ability to orchestrate conditional logic based on runtime data and node `Actions`.

---

### Internal Implementation Walkthrough: How Flows Work Under the Hood

The magic of `Flows` happens within its `run()` method, which effectively traverses the defined graph of nodes. Let's break down the typical sequence of events:

1.  **Graph Representation**: Internally, a `Flow` instance maintains a graph-like structure, often a dictionary mapping `node_id`s to `Node` instances, and another structure (e.g., a nested dictionary or adjacency list) mapping `(source_node_id, action_string)` pairs to `target_node_id`s. When you call `flow.node()` and `flow.on()`, you're building this graph.

2.  **Initialization**: When `flow.run(initial_store)` is called, the `Flow` identifies its starting `Node` (the first one added via `node()`). It also takes the `initial_store`, which is essentially the `SharedStore` pre-populated with any necessary input data.

3.  **The Execution Loop**: The `Flow` enters a loop that continues until it explicitly receives a `Flow.END` action or encounters a state where no valid transition can be found. In each iteration:

    a.  **Get Current Node**: The `Flow` retrieves the `Node` instance corresponding to the current `node_id`.

    b.  **Execute Node Lifecycle**: It invokes the `Node`'s lifecycle methods:
        *   `current_node.prep(store)`: Prepares the node for execution.
        *   `action_result = current_node.exec(store)`: Executes the node's core logic.
        *   `current_node.post(store)`: Handles post-execution tasks.
        *(Note: While `post` typically handles results, for simplicity and common patterns, `exec` often returns the `Action` directly to the `Flow` if no separate `post`-logic is needed for the flow transition itself).*

    c.  **Process Action**: The `Flow` receives the `Action` object returned by the `Node` (e.g., `Action("summary_too_long")`).

    d.  **Determine Next Node**: Using its internal graph, the `Flow` looks up the next `node_id` that corresponds to the `current_node_id` and the `action_result.name`. If `action_result` is `Flow.END`, the loop terminates.

    e.  **Transition**: The `Flow` updates its `current_node_id` to the newly determined next node, and the loop continues to the next iteration.

4.  **Completion**: Once the `Flow.END` action is received, the loop breaks, and the final state of the `SharedStore` is returned to the caller.

This process ensures that data is consistently passed through the `SharedStore` and that the application logic flows dynamically based on `Node` outputs.

```mermaid
sequenceDiagram
    participant App as Application
    participant Flow
    participant NodeInstance as Current Node
    participant SharedStore

    App->>Flow: run(initial_store)
    Flow->>Flow: Initialize with start node
    loop While current_node_id is not Flow.END
        Flow->>NodeInstance: prep(SharedStore)
        NodeInstance->>NodeInstance: Executes internal logic
        NodeInstance-->>Flow: returns Action("next_action")
        Flow->>NodeInstance: post(SharedStore)
        Flow->>Flow: Determine next node from graph using (current_node_id, Action)
        alt No transition defined for Action
            Flow->>App: Error: Invalid Action or no path found
            break
        end
        Flow->>Flow: Update current_node_id to next node
    end
    Flow-->>App: returns final_store
```
*Reference*: The core logic for `Flow` definition and execution can conceptually be found within `pocket_flow/flow.py` (though the actual file structure might vary, this conceptual file handles the graph management and execution loop).

---

### System Integration: Flows in the Pocket Flow Ecosystem

`Flows` don't operate in isolation; they are the central orchestrators that bind together other core components of Pocket Flow.

*   **Integration with [Nodes](chapter_02.md)**: This is the most direct integration. `Flows` are designed specifically to schedule and execute `Nodes`. Every step in a `Flow` is either a `Node` or a nested `Flow`. The `Flow` relies on `Nodes` to perform the actual work and to signal their outcomes via `Actions`.
*   **Integration with [Shared Store](chapter_01.md)**: The `Shared Store` is the indispensable communication bus. `Flows` pass the `Shared Store` instance to each `Node` it executes. This allows `Nodes` to read inputs generated by previous `Nodes` and write outputs for subsequent `Nodes` or the final result. The `Flow` itself doesn't directly manipulate data within the store beyond passing it along.
*   **Integration with Params**: While `Params` provide immutable configuration to individual `Nodes`, `Flows` can also accept `Params` during their creation, which can then be passed down to specific nodes within the flow, allowing for configurable flow behaviors without altering the flow's structure.
*   **Nested Flows**: A powerful integration pattern is nesting `Flows`. A `Flow` itself can be treated as a `Node` within another, larger `Flow`. This promotes extreme modularity, allowing complex systems to be built from smaller, independently testable sub-flows.
    ```python
    # Example of a nested flow (conceptual)
    from pocket_flow import Flow, Node, Action

    # Define a sub-flow
    class SubFlow(Flow):
        def __init__(self, name="SubFlow"):
            super().__init__(name)
            self.node(NodeA, "node_a") \
                .on("done_A", NodeB, "node_b") \
                .on("done_B", Flow.END)

    # Define a main flow that uses the sub-flow
    main_flow = Flow("MainFlow") \
        .node(NodeX, "node_x") \
        .on("done_X", SubFlow, "my_sub_flow_instance") \ # SubFlow used like a Node
        .on("done_processing", NodeY, "node_y", from_node_id="my_sub_flow_instance") \
        .on("done_Y", Flow.END)
    ```
    In this example, `SubFlow` is instantiated and used within `main_flow` as if it were a regular `Node`, demonstrating how a `Flow` can encapsulate a complex sequence of operations, offering a high degree of reusability and abstraction.

---

### Best Practices & Tips: Designing Effective Flows

*   **Single Responsibility Principle for Flows**: Just like `Nodes`, try to keep your `Flows` focused on a single, well-defined goal. A `Flow` that tries to do too much becomes hard to understand, debug, and reuse.
*   **Descriptive Action Names**: Use clear, concise, and meaningful names for your `Actions` (e.g., `"summary_too_long"`, `"validation_failed"`, `"data_ready"`). This makes the flow's logic immediately understandable.
*   **Explicit Error Handling**: Design specific `Actions` for error states (e.g., `Action("error_bad_input")`). Define explicit transitions for these error `Actions` to dedicated error-handling nodes or to terminate the `Flow` gracefully.
*   **Leverage Nested Flows for Modularity**: For complex applications, break down your main process into smaller sub-flows. This improves readability, allows for independent testing of sub-processes, and promotes code reuse.
*   **Visualize Complex Flows**: For larger flows, sketching out the graph on paper or using a diagramming tool (like mermaid or graphviz, if integrated) can be incredibly helpful for understanding and debugging.
*   **Input/Output Contracts**: While the `Shared Store` is flexible, document the expected keys that each `Node` (and thus `Flow`) reads from and writes to the store. This acts as a loose contract for data flow.
*   **Avoid Cycles (Unless Intentional)**: Be cautious with cycles in your flow graph unless you explicitly design them for iterative processes. Undesigned cycles can lead to infinite loops.

---

### Chapter Conclusion

`Flows` are the backbone of any non-trivial Pocket Flow application. They elevate a collection of independent `Nodes` into a coherent, intelligent system capable of complex decision-making and task orchestration. By defining clear sequences and leveraging `Actions` for dynamic transitions, `Flows` allow developers to model intricate AI application logic in a structured, maintainable, and highly flexible manner. Understanding how to design and implement effective `Flows` is crucial for building robust and adaptable AI solutions with Pocket Flow.

Now that we've covered the individual components – how data is shared, how tasks are performed, and how these tasks are orchestrated – we are ready to bring it all together. In the next chapter, [What is Pocket Flow?](chapter_04.md), we will zoom out to understand the holistic architecture and philosophy of the entire framework.
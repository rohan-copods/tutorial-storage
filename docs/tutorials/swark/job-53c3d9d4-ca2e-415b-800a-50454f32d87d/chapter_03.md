# Chapter 3: LLM Prompt Construction

Previous: [Workspace Data Ingestion](chapter_02.md)

### Architectural Intent & Motivation

The LLM Prompt Construction module serves as a critical translation layer within the `swark` pipeline. Its primary responsibility is to transform raw source code and contextual metadata into a structured, optimized prompt suitable for submission to a generative AI model.

This abstraction satisfies the architectural requirement of decoupling the data gathering phase (`Workspace Data Ingestion`) from the AI interaction phase (`AI Model Interaction`). Language models require precisely formatted input that includes not only the user's code but also explicit instructions, output constraints, and examples (few-shot prompting). This module encapsulates the logic for assembling this complex query, ensuring that all requests to the AI are consistent, effective, and designed to yield parsable responses.

### Contextual Use Case

Consider the primary workflow initiated by the `swark: Analyze Architecture` command.

1.  The `Workspace Data Ingestion` module finishes reading the contents of selected source files (`.ts`, `.tsx`, etc.).
2.  The `Command Orchestration` component receives this collection of file contents.
3.  The orchestrator then invokes the `PromptBuilder` from this module.
4.  The `PromptBuilder` takes the raw code strings and wraps them within a predefined template. This template includes a "system prompt" that instructs the AI to act as a software architect and a "formatting prompt" that demands the output be a specific JSON structure compatible with `swark`'s `Domain Model`.
5.  The final output is a single, complete string ready to be sent as the payload in an API call to the AI model. Without this step, the AI would lack the necessary context and constraints to generate a useful architectural analysis.

### Concept Decomposition

*   **Prompt Template:** A static string structure that defines the sections of the final prompt. It contains placeholders for dynamic content like source code.
*   **System Instructions:** A part of the template that defines the AI's role, persona, and high-level goals. For example: "You are an expert software architect. Your task is to analyze the provided code and identify its core components and their relationships."
*   **Formatting Constraints:** A critical instruction set, embedded in the template, that dictates the exact schema of the expected output (e.g., JSON). This is essential for deterministic parsing in the `Response Formatting` stage.
*   **User Context:** The dynamic portion of the prompt, primarily consisting of the source code ingested from the user's workspace.
*   **PromptBuilder:** The core entity of this module. It implements the logic for populating the Prompt Template with the User Context and system-specific instructions.

### Reference Implementation

The canonical usage pattern involves instantiating a `PromptBuilder`, populating it with context from the ingested files, and then calling a terminal method to generate the final string.

```typescript
// Assumes 'ingestedFiles' is an array of { path: string, content: string }
// provided by the Workspace Data Ingestion module.

import { PromptBuilder } from './promptBuilder';

// 1. Instantiate the builder within the Command Orchestrator.
const promptBuilder = new PromptBuilder();

// 2. Iteratively add the relevant source code context.
for (const file of ingestedFiles) {
    promptBuilder.addSourceCode(file.path, file.content);
}

// 3. Finalize the prompt for the AI model.
const finalPrompt: string = promptBuilder.build();
```

This snippet demonstrates how the orchestrator uses the builder to abstract away the details of prompt formatting.

### Architectural Mechanics (White-Box Analysis)

#### Design Pattern Identification

*   **Builder Pattern:** The `PromptBuilder` entity is a direct implementation of the Builder pattern. This pattern is employed to separate the construction of a complex object (the final prompt string) from its representation.
    *   **Justification:** The prompt's structure is non-trivial, involving multiple sections (system instructions, code blocks, formatting rules). The Builder pattern allows the `Command Orchestration` component to add these pieces step-by-step without needing to know the intricate details of final string assembly, token counting, or special character escaping.

#### Control Flow/State

The `PromptBuilder` maintains an internal, mutable state, likely an array of strings or a structured object representing the different parts of the prompt.

1.  **Instantiation:** The builder is initialized with a base template.
2.  **Accumulation:** Methods like `addSourceCode` append data to the builder's internal state. This is a state-mutating operation.
3.  **Finalization:** The `build()` method is a terminal operation. It consumes the accumulated internal state, performs final formatting (e.g., joining strings, wrapping code in markdown blocks), and returns an immutable `string`. The internal state of the builder may be cleared after this operation to allow for reuse.

### Architectural Visualization

An architectural diagram is omitted for this module. The `PromptBuilder` follows a linear, sequential construction process that is more clearly represented by its code implementation than by a visual diagram. Generating a sequence or state diagram for this simple accumulation and finalization logic would be noise and violate the principle of diagrammatic necessity.

### System Topology & Integration

#### Dependency Graph

*   **Upstream (Dependents):**
    *   `Command Orchestration` (Chapter 9): This is the sole consumer of the `PromptBuilder`. It directs the prompt construction process.
*   **Downstream (Dependencies):**
    *   This module has no direct runtime dependencies on other `swark` modules. However, it has a critical data dependency on the output of `Workspace Data Ingestion` (Chapter 2).

#### Data Propagation

*   **Input Boundary:** The module's primary entry point (e.g., `PromptBuilder.addSourceCode`) accepts data representing file context, typically in the form of `{ path: string, content: string }`.
*   **Output Boundary:** The module's terminal exit point (`PromptBuilder.build`) produces a single, primitive `string`. This string is then passed directly to the `AI Model Interaction` module (Chapter 4) as its primary payload.

### Engineering Standards

#### Performance Implications

*   **Complexity:** The operational complexity is approximately O(C), where C is the total number of characters across all ingested source files. The primary cost is associated with string concatenation and storage. Implementations should use an array of string parts and a final `join('')` operation rather than repeated `+` concatenation to avoid quadratic performance degradation in engines without string optimization.
*   **Token Limits:** A significant constraint is the context window (token limit) of the target LLM. The `PromptBuilder` may eventually need to incorporate logic to truncate or summarize input if the total size of the source code exceeds the model's limit.

#### Anti-Patterns

*   **Prompt Injection via Orchestrator:** The `Command Orchestration` module should not manually construct or manipulate parts of the prompt string itself. All prompt-related logic must be encapsulated within the `PromptBuilder`. Bypassing the builder breaks the abstraction and leads to inconsistent AI queries.
*   **Context Stuffing:** Providing the builder with irrelevant context (e.g., `node_modules`, build artifacts, large binary files masquerading as text) is a significant misuse case. While filtering is the responsibility of the upstream `Workspace Data Ingestion` module, the `PromptBuilder` is where the cost of this error is realized in the form of expensive, slow, and low-quality AI responses.

### Conclusion

The LLM Prompt Construction module is the architectural lynchpin that enables effective communication with the generative AI. By abstracting the complexities of prompt engineering behind a well-defined `PromptBuilder`, it provides the `swark` system with a robust and maintainable mechanism for translating user code into actionable AI queries. This encapsulation ensures that as AI models and prompting techniques evolve, changes can be isolated to this single module without impacting the broader application workflow.

Next: [AI Model Interaction](chapter_04.md)
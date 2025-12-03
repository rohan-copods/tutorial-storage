# Chapter 7: Result Persistence

-   **Reference**: [Chapter 6: Mermaid Diagram Generation](chapter_06.md)

### Architectural Intent & Motivation

The Result Persistence module serves as the terminal stage of the `swark` data processing pipeline. Its primary responsibility is to materialize the in-memory analysis artifacts—the structured domain model and the generated Mermaid diagram—by writing them to the file system.

This abstraction satisfies a core business requirement: providing users with tangible, shareable, and version-controllable outputs from a successful analysis run. By decoupling the file I/O logic from the core analysis and generation phases, the system maintains a clean separation of concerns, allowing other components to remain agnostic about the underlying storage mechanism.

### Contextual Use Case

A developer executes the `swark: Analyze Architecture` command on a file named `PaymentService.ts`. The pipeline successfully ingests the code, queries the AI, parses the response into the domain model, and generates a Mermaid diagram string. The `Command Orchestration` component then invokes the Result Persistence module. This module takes the final data and creates two new files in the same directory as the source: `PaymentService.analysis.json` and `PaymentService.diagram.md`. The developer can now review these files, commit them to Git, or use them in external documentation.

### Concept Decomposition

The persistence logic is broken down into the following key concepts:

*   **Output Writer:** The central function, `writeOutput`, which encapsulates the entire persistence operation.
*   **Payloads:** The data designated for persistence. This consists of two distinct artifacts:
    *   **Analysis JSON:** The raw, serialized `Analysis` domain model (defined in Chapter 1), providing a machine-readable representation of the code structure.
    *   **Diagram Markdown:** The Mermaid syntax string generated in the previous chapter (Chapter 6), ready for rendering.
*   **Path Derivation:** The logic for constructing the absolute output file paths. This process uses the original source file's path as a base to ensure the generated artifacts are co-located with the code they describe.

### Reference Implementation

The canonical usage involves passing a configuration object to the `writeOutput` function. This function is asynchronous, reflecting the nature of file system I/O.

```typescript
// Invocation from the Command Orchestrator
import { writeOutput } from './resultPersistence';

// Data artifacts from previous pipeline stages
const sourcePath = '/workspace/src/api/PaymentService.ts';
const analysisJson = '{"components": [{"name": "PaymentHandler"}]}';
const diagramMarkdown = 'graph TD\n    PaymentHandler-->StripeClient';

// Execute the persistence operation
await writeOutput({
    sourcePath: sourcePath,
    analysisJson: analysisJson,
    diagramMarkdown: diagramMarkdown
});
```

This snippet demonstrates how the orchestrator provides the necessary payloads and source context to the persistence module, which then handles the file creation side effects.

### Architectural Mechanics (White-Box Analysis)

#### Design Pattern Identification

The Result Persistence module implements the **Facade Pattern**. It provides a single, simplified method (`writeOutput`) that conceals the more complex, low-level interactions with the underlying Node.js file system APIs (`fs/promises`, `path`). Consumers of this module do not need to manage path resolution, file handle creation, or sequential write operations; they simply call the high-level function with the required data.

#### Control Flow/State

The control flow is linear and asynchronous:
1.  The `writeOutput` function is invoked with the source path and data payloads.
2.  The `path` module is used to derive the full output paths for the `.json` and `.md` files from the `sourcePath` argument.
3.  Two asynchronous file write operations are initiated using the Node.js `fs/promises` API. These writes can occur in parallel to optimize I/O.
4.  The function returns a `Promise<void>` that resolves when both write operations have successfully completed or rejects if either one fails.
5.  This module is stateless; its behavior depends solely on the inputs provided for each call.

### Architectural Visualization

A diagram is not applicable for this module. The control flow is a simple, linear sequence of operations (derive path, write file A, write file B). As per the documentation protocol, generating a diagram for such a process would introduce unnecessary complexity rather than clarify the architecture.

### System Topology & Integration

#### Dependency Graph

*   **Upstream (Dependents):**
    *   `Command Orchestration` (Chapter 9): This is the exclusive consumer of the Result Persistence module. It invokes `writeOutput` as the final step in a successful analysis pipeline.
*   **Downstream (Dependencies):**
    *   **Node.js `fs/promises` API:** Used for all asynchronous file writing operations.
    *   **Node.js `path` API:** Used for parsing the source path and constructing the destination paths for the output files.
    *   **Domain Model & Types** (Chapter 1): The structure of the analysis data (which becomes `analysisJson`) is defined by the core system types.

#### Data Propagation

*   **Ingress Data:** The module's public function, `writeOutput`, accepts a single object containing:
    *   `sourcePath: string`: The absolute path to the file that was analyzed.
    *   `analysisJson: string`: A stringified JSON object representing the `Analysis` domain model.
    *   `diagramMarkdown: string`: A string containing the complete Mermaid diagram syntax.
*   **Egress Data:** This module is a "sink" and produces no return value (`Promise<void>`). Its output is a **side effect**: the creation or overwriting of files on the local file system.

### Engineering Standards

#### Performance Implications

The performance of this module is bound by disk I/O speed. While the logic itself has a constant time complexity (`O(1)`), the actual execution time is dependent on the size of the JSON and Markdown payloads being written to disk. The use of asynchronous APIs ensures that the VS Code extension host is not blocked during these I/O operations.

#### Anti-Patterns

*   **Ignoring the Returned Promise:** Failing to `await` the `writeOutput` call is a critical misuse. Subsequent operations, such as displaying a "Save Complete" notification or opening the file in the webview, could execute before the files are actually present on disk, leading to race conditions and errors.
*   **Passing Incomplete Paths:** Providing a relative or malformed `sourcePath` can cause the module to write files to an unintended or invalid location. The `sourcePath` is expected to be an absolute path resolved by the `Workspace Data Ingestion` module.
*   **Neglecting Error Handling:** The caller (`Command Orchestration`) is responsible for wrapping the `writeOutput` call in a `try...catch` block. Failure to do so means that file system errors (e.g., permission denied, disk full) will result in an unhandled promise rejection, potentially crashing the extension's command execution context.

### Conclusion

The Result Persistence module provides a crucial, well-defined boundary for handling file system side effects. By acting as a simple facade, it finalizes the analysis workflow, translating abstract in-memory data structures into durable, user-accessible files. This function is essential for making the output of `swark` both portable and integrated with standard developer workflows like version control.

-   **Forward Link**: [Chapter 8: Webview-based Visualization](chapter_08.md)
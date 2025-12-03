# Chapter 1: Core Domain Models

- Initialization point.

## Architectural Intent & Motivation

The Core Domain Models are the canonical data structures that represent the fundamental entities within the `swark` system. Their primary responsibility is to establish a strict, type-safe contract for data interchange between the distinct layers of the processing pipeline. These models act as the system's *lingua franca*, ensuring that the Codebase Ingestion service, the LLM Prompt Engineering layer, and the Output Visualization component all share a consistent and unambiguous understanding of the software artifacts being analyzed.

This abstraction directly satisfies the architectural requirement for a decoupled, layered pipeline. By defining a shared, immutable data schema, the models eliminate implicit dependencies and prevent data-shape mismatches between stages, enforcing the system's unidirectional data flow.

## Contextual Use Case

Within the `swark` workflow, the `Codebase Ingestion & Analysis` service (Chapter 2) is the first point of contact with the target repository. When it parses a file and identifies a significant software artifact, such as a Python class, it does not pass a raw dictionary or a generic tuple to the next stage.

Instead, it is mandatory for the ingestion service to instantiate a `Component` model. It populates the model's fields (`name`, `type`, `dependencies`, `code_snippet`). This structured object is then passed to the `LLM Prompt Engineering` service. The prompt engineering layer can then reliably access `component.dependencies` to construct a query about its relationships, confident in the existence and type of the data without needing any knowledge of the initial parsing logic.

## Concept Decomposition

The domain is deconstructed into a primary logical unit, the `Component`, which serves as the atomic unit of architectural analysis.

### Key Domain Terminology

*   **Component:** The fundamental unit of analysis in `swark`. It represents a discrete, identifiable software artifact such as a class, function, module, or microservice. Each `Component` is the node in the final dependency graph.
*   **Dependency:** A directed relationship from one `Component` to another. This signifies that the source component requires or references the target component to fulfill its function.

## Reference Implementation

The following snippet demonstrates the canonical instantiation of a `Component` model by an upstream service like the ingestion layer. The model is a simple data container, devoid of business logic.

*Constraint: The following code is a representative, language-agnostic example synthesized to align with the architectural synopsis, as no direct source was provided. It reflects a common Data Transfer Object (DTO) pattern.*

```python
# Canonical instantiation within the Ingestion service
# Presumed structure from swark.core.models

# A database utility class is identified in the target codebase.
db_client_component = Component(
    name="DatabaseClient",
    type="CLASS",
    dependencies=["ConnectionPool", "os.environ"],
    code_snippet="class DatabaseClient:\n  def __init__(self): ..."
)

# A high-level service class is identified.
api_service_component = Component(
    name="ApiService",
    type="CLASS",
    dependencies=["DatabaseClient", "Flask.request"],
    code_snippet="class ApiService:\n  def get_user(self, user_id): ..."
)
```

## Architectural Mechanics (White-Box Analysis)

### Design Pattern Identification

*   **Data Transfer Object (DTO):** The `Component` model is a pure DTO. Its sole purpose is to aggregate and transport data between the distinct layers of the `swark` pipeline. It contains no business logic, ensuring a clean separation between data and behavior.
*   **Value Object:** Instances of the domain models are treated as Value Objects. Their identity is defined by the data they hold, not by a unique instance identifier. They are intended to be immutable once created, supporting the functional, unidirectional data flow of the architecture.

### Control Flow/State

The Core Domain Models are passive and contain no control flow. Their state is defined entirely at the moment of instantiation by the `Codebase Ingestion & Analysis` service. State is not mutated as the models pass through the pipeline; subsequent layers may create new, transformed collections of models, but they do not modify existing instances in place. This immutability is critical for preventing side effects and ensuring predictable behavior.

## Architectural Visualization (Mermaid)

A visual diagram is not applicable for this abstraction. The Core Domain Models are declarative data structures, not procedural components. They lack state transitions, asynchronous interactions, or a compositional hierarchy, making a diagram redundant. Their structure is most clearly and accurately conveyed through their type definitions.

## System Topology & Integration

### Dependency Graph

*   **Upstream (Dependents):** Nearly every other module in `swark` depends on the Core Domain Models. This includes:
    *   `Codebase Ingestion & Analysis` (Chapter 2): Creates and populates these models.
    *   `LLM Prompt Engineering` (Chapter 3): Reads data from these models to build prompts.
    *   `AI Model Interaction Service` (Chapter 4): May receive and pass through these models.
    *   `Output Visualization` (Chapter 5): Consumes the final collection of models and their relationships to render a graph.
    *   `Command & Control Flow` (Chapter 6): Orchestrates the flow of these models between stages.

*   **Downstream (Dependencies):** The Core Domain Models have no dependencies on other `swark` modules. They are foundational and should only depend on language-native types (e.g., strings, lists).

### Data Propagation

*   **Data Entry:** `Component` objects are instantiated and enter the `swark` pipeline exclusively from the `Codebase Ingestion & Analysis` service. This is the sole source of truth for these models.
*   **Data Flow:** A collection of `Component` objects (e.g., a `list[Component]`) is the standard payload passed sequentially between each major service in the pipeline.

## Engineering Standards

### Performance Implications

Performance impact is minimal. The memory and CPU footprint of instantiating these models is negligible. The overall resource consumption scales linearly, O(n), with the number of `Component` artifacts identified in the target codebase. As these are simple data carriers, they do not introduce any algorithmic complexity.

### Anti-Patterns

*   **Embedding Business Logic:** Adding methods to a model that perform I/O, make API calls, or contain complex logic is a severe anti-pattern. This would violate the DTO pattern and couple the system's data contract to a specific implementation.
*   **In-Place Mutation:** A downstream service (e.g., Prompt Engineering) must not modify a `Component` instance it receives. Modifying a model in-place breaks the immutability assumption and violates the unidirectional data flow, leading to unpredictable behavior in subsequent pipeline stages.
*   **Schema Circumvention:** Services must not pass raw dictionaries or other non-standard structures between layers. Adherence to the Core Domain Models is mandatory to maintain the integrity of the system's contract.

## Conclusion

The Core Domain Models are the architectural bedrock of `swark`. By providing a simple, stable, and universal contract for representing software artifacts, they enable the system's loosely coupled, pipeline-based design. This ensures that each component can evolve independently while maintaining seamless interoperability, resulting in a more robust and maintainable system.

- Forward Link: Codebase Ingestion & Analysis
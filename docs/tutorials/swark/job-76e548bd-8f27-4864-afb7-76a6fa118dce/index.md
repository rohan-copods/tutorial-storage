# swark Tutorial

Welcome to the comprehensive tutorial for swark. This tutorial is automatically generated from the codebase to help you understand the core concepts and implementation patterns.

## Project Overview

swark is a VS Code extension that automates architectural diagram generation by analyzing source code through an external LLM. The system is designed with a layered, pipeline architecture where a central orchestrator executes a sequence of components for context gathering, AI-driven analysis, data parsing, and final visualization.


## System Architecture

```mermaid
graph TB
    title["swark Architecture"]

    subgraph Interface_Layer["Interface Layer"]
        LLMPromptingInteraction["LLM Prompting & Interaction"]
        VSCodeExtensionLifecycleActivation["VS Code Extension Lifecycle & Activation"]
    end

    subgraph Business_Logic["Business Logic"]
        WorkspaceReaderContextBuilder["Workspace Reader & Context Builder"]
        CoreCommandOrchestrator["Core Command Orchestrator"]
    end

    subgraph Data_Layer["Data Layer"]
        DomainPrimitivesTypes["Domain Primitives & Types"]
        AnalysisOutputParserFormatter["Analysis Output Parser & Formatter"]
        ResultVisualizationPersistence["Result Visualization & Persistence"]
    end

```

## Component Relationships

```mermaid
flowchart TD
    DomainPrimitivesTypes["Domain Primitives & Types<br/>Defines the core data structures like 'ComponentModel' that serve as the system's vocabulary.
This foundational layer establishes the data contracts used by all other compo..."]
    WorkspaceReaderContextBuilder["Workspace Reader & Context Builder<br/>Gathers context by reading and processing files from the user's workspace.
It identifies relevant source code, counts language statistics, and prepares the
'ArchitecturalFi..."]
    LLMPromptingInteraction["LLM Prompting & Interaction<br/>Encapsulates all communication with the external Large Language Model API.
This module builds the specific prompt from workspace context and manages the
request/response cy..."]
    AnalysisOutputParserFormatter["Analysis Output Parser & Formatter<br/>Transforms the raw, unstructured LLM string response into the system's structured 'ComponentModel'.
It is also responsible for serializing this structured data into the fin..."]
    ResultVisualizationPersistence["Result Visualization & Persistence<br/>Manages the presentation and storage of the final architectural diagram.
This component writes the generated Mermaid diagram to a markdown file and displays
it to the user ..."]
    CoreCommandOrchestrator["Core Command Orchestrator<br/>Orchestrates the primary end-to-end user workflow for generating an architecture diagram.
It sequences the calls to the reader, LLM interactor, formatter, and viewer to exe..."]
    VSCodeExtensionLifecycleActivation["VS Code Extension Lifecycle & Activation<br/>Serves as the main entry point that integrates the application logic with the VS Code API.
It is responsible for registering the 'createArchitecture' command and managing t..."]

    %% Relationships
    VSCodeExtensionLifecycleActivation -->|"triggers command execution of"| CoreCommandOrchestrator
    CoreCommandOrchestrator -->|"invokes to read workspace context"| WorkspaceReaderContextBuilder
    CoreCommandOrchestrator -->|"invokes to query LLM for analysis"| LLMPromptingInteraction
    CoreCommandOrchestrator -->|"invokes to parse LLM response"| AnalysisOutputParserFormatter
    CoreCommandOrchestrator -->|"invokes to persist and display diagram"| ResultVisualizationPersistence
    WorkspaceReaderContextBuilder -->|"provides file context to"| LLMPromptingInteraction
    LLMPromptingInteraction -->|"provides raw analysis string to"| AnalysisOutputParserFormatter
    AnalysisOutputParserFormatter -->|"provides Mermaid diagram syntax to"| ResultVisualizationPersistence
    WorkspaceReaderContextBuilder -->|"instantiates data contracts from"| DomainPrimitivesTypes
    AnalysisOutputParserFormatter -->|"constructs structured 'ComponentModel' from"| DomainPrimitivesTypes
```

## Table of Contents

1. [Chapter 1: Domain Primitives & Types](chapter_01.md) - Comprehensive documentation for Domain Primitives & Types following structured methodology...
2. [Chapter 2: Workspace Reader & Context Builder](chapter_02.md) - Comprehensive documentation for Workspace Reader & Context Builder following structured methodology...
3. [Chapter 3: LLM Prompting & Interaction](chapter_03.md) - Comprehensive documentation for LLM Prompting & Interaction following structured methodology...
4. [Chapter 4: Analysis Output Parser & Formatter](chapter_04.md) - Comprehensive documentation for Analysis Output Parser & Formatter following structured methodology...
5. [Chapter 5: Result Visualization & Persistence](chapter_05.md) - Comprehensive documentation for Result Visualization & Persistence following structured methodology...
6. [Chapter 6: Core Command Orchestrator](chapter_06.md) - Comprehensive documentation for Core Command Orchestrator following structured methodology...
7. [Chapter 7: VS Code Extension Lifecycle & Activation](chapter_07.md) - Comprehensive documentation for VS Code Extension Lifecycle & Activation following structured method...

## How to Use This Tutorial

1. **Start with Chapter 1** to understand the foundational concepts
2. **Follow the sequence** - each chapter builds upon previous concepts
3. **Practice with code examples** - every chapter includes practical examples
4. **Refer to diagrams** - use architecture diagrams for visual understanding
5. **Cross-reference concepts** - chapters link to related topics

## Tutorial Features

- **Progressive Learning**: Concepts are introduced in logical order
- **Code Examples**: Every chapter includes practical, executable code
- **Visual Diagrams**: Mermaid diagrams illustrate complex relationships
- **Cross-References**: Easy navigation between related concepts
- **Beginner-Friendly**: Written for newcomers to the codebase

## Contributing

This tutorial is auto-generated from the codebase. To improve it:
1. Update the source code documentation
2. Add more detailed comments to key functions
3. Regenerate the tutorial using the documentation system

---

*Generated using AI-powered codebase analysis*

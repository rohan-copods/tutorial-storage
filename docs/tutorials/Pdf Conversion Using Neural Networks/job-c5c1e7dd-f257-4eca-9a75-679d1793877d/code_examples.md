# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: The Challenge of PDF Document Structure](chapter_01.md)

- **Example 1** (markdown): Section 1.1: Introduction
- **Example 2** (python): conceptual_pdf_parser.py
- **Example 3** (text): --- Naive Extraction Output ---
- **Example 4** (mermaid): sequenceDiagram

## [Chapter 2: The Goal: Document Structure Recovery](chapter_02.md)

- **Example 1** (python): Conceptual input from low-level PDF data (simplified representation)
- **Example 2** (python): The *goal* of the structure recovery process,
- **Example 3** (mermaid): graph TD

## [Chapter 3: Using Neural Networks for PDF Conversion](chapter_03.md)

- **Example 1** (python): This is a conceptual example, not executable code yet.
- **Example 2** (mermaid): graph TD
- **Example 3** (mermaid): graph LR

## [Chapter 4: Direct Processing of Low-Level PDF Data](chapter_04.md)

- **Example 1** (python): from PyPDF2 import PdfReader
- **Example 2** (python): Conceptual example: imagine this is part of a decompressed PDF content stream
- **Example 3** (python): This is a sample of the structured data that would be output
- **Example 4** (python): src/pdf_parser/low_level_reader.py (Conceptual)
- **Example 5** (python): src/pdf_parser/stream_processor.py (Conceptual)
- **Example 6** (python): src/pdf_parser/operator_extractor.py (Conceptual)
- **Example 7** (mermaid): sequenceDiagram
- **Example 8** (mermaid): graph TD

## [Chapter 5: Designing Input Features for Models](chapter_05.md)

- **Example 1** (python): class TextElement:
- **Example 2** (python): def extract_bare_features(element):
- **Example 3** (python): def extract_extended_features(current_element, next_element=None):
- **Example 4** (python): import numpy as np
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 6: Recurrent Neural Networks (RNNs) and Seq2Seq Architecture](chapter_06.md)

- **Example 1** (python): import tensorflow as tf
- **Example 2** (python): Decoder Input: A sequence for label generation (initially padding, then predicted labels)
- **Example 3** (mermaid): sequenceDiagram
- **Example 4** (mermaid): graph TD

## [Chapter 7: Measuring Model Performance and Accuracy](chapter_07.md)

- **Example 1** (python): from sklearn.metrics import f1_score
- **Example 2** (python): def calculate_iou(boxA, boxB):
- **Example 3** (mermaid): sequenceDiagram
- **Example 4** (python): evaluate_script.py (simplified)


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

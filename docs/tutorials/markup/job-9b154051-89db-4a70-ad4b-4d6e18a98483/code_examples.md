# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: The Main Rendering Interface](chapter_01.md)

- **Example 1** (markdown): Welcome!
- **Example 2** (ruby): require 'markup'
- **Example 3** (html): <h1>Welcome!</h1>
- **Example 4** (asciidoc): = Project Setup
- **Example 5** (ruby): require 'markup'
- **Example 6** (html): <div class="sect1">
- **Example 7** (ruby): require 'markup'
- **Example 8** (html): <h2>Hello, World!</h2>
- **Example 9** (mermaid): sequenceDiagram

## [Chapter 2: The Command-Line Tool](chapter_02.md)

- **Example 1** (markdown): Project Guide
- **Example 2** (bash): markup guide.md > guide.html
- **Example 3** (html): <h1>Project Guide</h1>
- **Example 4** (bash): markup guide.md -o guide.html
- **Example 5** (bash): echo "**Hello, World!**" | markup
- **Example 6** (html): <p><strong>Hello, World!</strong></p>
- **Example 7** (mermaid): sequenceDiagram

## [Chapter 3: Markup Language Registry](chapter_03.md)

- **Example 1** (ruby): lib/markup.rb
- **Example 2** (ruby): Check for Markdown support
- **Example 3** (text): Markup::Renderers::MarkdownRenderer
- **Example 4** (text): *   **Explanation**: This class defines a `rend...
- **Example 5** (text): *   **Explanation**: The `register` method adds...
- **Example 6** (text): *   **Expected Output**:

## [Chapter 4: Rendering Strategies: Commands vs. Gems](chapter_04.md)

- **Example 1** (ruby): Renders Markdown using the 'redcarpet' gem.
- **Example 2** (ruby): Renders AsciiDoc using the 'asciidoctor' command-line tool.
- **Example 3** (mermaid): sequenceDiagram

## [Chapter 5: Specific Language Renderers](chapter_05.md)

- **Example 1** (ruby): lib/markup/markdown.rb
- **Example 2** (ruby): lib/markup/asciidoc.rb
- **Example 3** (ruby): config/registry.rb
- **Example 4** (mermaid): sequenceDiagram

## [Chapter 6: The Generic Renderer Contract](chapter_06.md)

- **Example 1** (ruby): lib/markup/renderer/preformatted_text_renderer.rb
- **Example 2** (ruby): class PreformattedTextRenderer < Markup::Renderer::Base
- **Example 3** (ruby): class PreformattedTextRenderer < Markup::Renderer::Base
- **Example 4** (ruby): lib/markup/renderer/base.rb
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

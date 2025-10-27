# Chapter 7: HTML AST (HAST)

In the previous chapter, we delved into the intricacies of the [Markdown AST (MDAST)](chapter_06.md), understanding how `react-markdown` parses raw markdown into a structured, hierarchical representation. The MDAST provides a powerful way to understand and manipulate the *structure and semantics* of markdown content. However, while MDAST is excellent for markdown itself, it's not directly what browsers or React components expect for rendering. This is where the HTML Abstract Syntax Tree, or HAST, enters the stage.

---

### Problem & Motivation

The fundamental challenge after parsing markdown into an MDAST is bridging the gap between a markdown-specific data structure and a web browser's native understanding of HTML. An MDAST, with nodes like `paragraph`, `heading`, `link`, and `strong`, clearly represents markdown features. But a browser doesn't know what a `paragraph` node is; it understands `<p>` tags, `<a>` tags, `<strong>` tags, and `<h1>` to `<h6>` tags. Directly converting MDAST to React elements without an intermediate step would mean writing complex logic to map every possible MDAST node type to its corresponding HTML element, including handling attributes and nested structures, which can quickly become unwieldy and error-prone.

This is precisely the problem HAST solves. It acts as an **intermediate, HTML-centric representation** of the content. By transforming the MDAST into a HAST, `react-markdown` gains a standard, predictable data structure that closely mirrors the actual HTML DOM. This standardization simplifies the subsequent rendering phase significantly, as the system no longer needs to interpret markdown semantics but can instead focus on directly translating HTML semantics into React components or native DOM elements.

Consider a markdown link: `[Visit Google](https://google.com)`. In MDAST, this is a `link` node with a `url` property and a `children` array containing a `text` node. While semantically correct for markdown, to render this as HTML, we need an `<a>` tag with an `href` attribute and a text content. HAST provides this exact mapping, transforming the markdown `link` node into an HTML `element` node with `tagName: 'a'`, `properties: { href: 'https://google.com' }`, and a `children` array containing a `text` node with `value: 'Visit Google'`. This clear, HTML-aligned structure is crucial for the final rendering step.

---

### Core Concept Explanation

HAST stands for **HTML Abstract Syntax Tree**. It is a tree-like data structure that represents HTML documents in a programmatic way, much like MDAST represents markdown. The key difference is that HAST nodes directly correspond to HTML elements, attributes, and text nodes, making it an ideal format for processing and ultimately rendering web content.

In the `unified` ecosystem (which `react-markdown` leverages), HAST is the standard format for representing HTML. Each node in a HAST has a `type` (e.g., `'element'`, `'text'`, `'doctype'`, `'comment'`) and, for `'element'` nodes, a `tagName` (e.g., `'p'`, `'a'`, `'div'`), `properties` (HTML attributes like `href`, `class`, `id`), and `children` (nested HAST nodes). Text content is represented by `'text'` nodes with a `value` property.

The significance of HAST lies in its **universality and closeness to the DOM**. Once content is converted to HAST, it can be easily serialized into an HTML string, converted into browser DOM nodes, or, in the case of `react-markdown`, transformed into React elements. This abstraction separates the concerns of markdown parsing from HTML rendering. The transformation from MDAST to HAST is primarily handled by a `unified` plugin called `remark-rehype`, which acts as the crucial bridge between the markdown world and the HTML world.

By having a HAST, `react-markdown` can:
1.  **Standardize HTML representation**: All markdown features are mapped to standard HTML elements.
2.  **Facilitate custom rendering**: Users can easily map specific HTML `tagName`s in the HAST to their own React components via [Custom Component Mapping](chapter_04.md).
3.  **Enable HTML manipulation**: Plugins can inspect, add, remove, or modify HAST nodes before rendering, allowing for powerful transformations like syntax highlighting, image resizing, or adding custom attributes.
4.  **Simplify sanitization**: Dangerous HTML elements or attributes can be identified and removed directly from the HAST, enhancing security.

---

### Practical Usage Examples

Let's illustrate how a simple markdown snippet is transformed into its HAST representation. We won't show the full `react-markdown` usage here, but rather focus on the intermediate HAST structure that `react-markdown` eventually receives and renders.

**Motivating Use Case: A Paragraph with a Bold Word and a Link**

Consider the markdown input:
```markdown
This is a **bold** word and a [link](https://example.com).
```

1.  **Input Markdown:**
    ```markdown
    This is a **bold** word and a [link](https://example.com).
    ```
    This is the raw string `react-markdown` receives.

2.  **Conceptual MDAST (from [Markdown AST (MDAST)](chapter_06.md)):**
    (Simplified for brevity)
    ```javascript
    {
      type: 'root',
      children: [
        {
          type: 'paragraph',
          children: [
            { type: 'text', value: 'This is a ' },
            { type: 'strong', children: [{ type: 'text', value: 'bold' }] },
            { type: 'text', value: ' word and a ' },
            {
              type: 'link',
              url: 'https://example.com',
              children: [{ type: 'text', value: 'link' }]
            },
            { type: 'text', value: '.' }
          ]
        }
      ]
    }
    ```
    *Explanation*: The MDAST clearly shows a `paragraph` containing `text` nodes, a `strong` node (for `**bold**`), and a `link` node (for `[link](...)`). Notice the markdown-specific node types.

3.  **Corresponding HAST:**
    ```javascript
    {
      type: 'root',
      children: [
        {
          type: 'element',
          tagName: 'p',
          properties: {},
          children: [
            { type: 'text', value: 'This is a ' },
            {
              type: 'element',
              tagName: 'strong',
              properties: {},
              children: [{ type: 'text', value: 'bold' }]
            },
            { type: 'text', value: ' word and a ' },
            {
              type: 'element',
              tagName: 'a',
              properties: { href: 'https://example.com' },
              children: [{ type: 'text', value: 'link' }]
            },
            { type: 'text', value: '.' }
          ]
        }
      ]
    }
    ```
    *Explanation*: The MDAST `paragraph` node has been transformed into a HAST `element` node with `tagName: 'p'`. The `strong` node became `tagName: 'strong'`. Crucially, the `link` node became an `element` node with `tagName: 'a'` and its `url` property was correctly mapped to the `href` property in the `properties` object. This HAST structure is now ready to be rendered directly into React elements.

This transformation is a critical step, converting the abstract markdown structure into an abstract HTML structure, making it incredibly straightforward for `react-markdown` to map these HAST elements to actual React components during the rendering phase.

---

### Internal Implementation Walkthrough

The transition from MDAST to HAST is a core part of the `unified` processing pipeline used by `react-markdown`. This transformation typically happens after all markdown-specific plugins have processed the MDAST, but *before* any HTML-specific processing or rendering.

The primary tool for this conversion is a `unified` plugin called `remark-rehype`.

Here's a step-by-step conceptual walkthrough:

1.  **Markdown Input**: The raw markdown string is provided to `react-markdown`.
2.  **Parsing to MDAST**: The `markdown` parser (e.g., `micromark` or `remark-parse`) transforms the raw string into an MDAST. This is covered in [Markdown AST (MDAST)](chapter_06.md).
    ```mermaid
    graph TD
        A[Markdown String] --> B(remark-parse)
        B --> C[MDAST]
    ```
3.  **MDAST Transformations**: Various `remark` plugins (e.g., `remark-gfm` for GitHub Flavored Markdown) might operate on the MDAST, modifying it to add features or normalize structures.
    ```mermaid
    graph TD
        C[MDAST] --> D(remark-gfm / other remark plugins)
        D --> E[Processed MDAST]
    ```
4.  **MDAST to HAST Conversion**: This is the pivotal step where `remark-rehype` comes into play. It traverses the processed MDAST, systematically mapping each MDAST node type to its corresponding HAST `element` or `text` node.
    *   An MDAST `paragraph` node becomes a HAST `element` with `tagName: 'p'`.
    *   An MDAST `heading` node with `depth: 1` becomes a HAST `element` with `tagName: 'h1'`.
    *   An MDAST `link` node has its `url` property mapped to the `href` property of a HAST `element` with `tagName: 'a'`.
    *   Text nodes remain `text` nodes, simply carrying their `value`.
    *   For HTML content embedded directly in markdown (e.g., `<div>hello</div>`), `remark-rehype` also handles converting these raw HTML nodes into appropriate HAST `element` nodes, which might then be further processed or sanitized.
    ```mermaid
    graph TD
        E[Processed MDAST] --> F(remark-rehype)
        F --> G[HAST]
    ```
    The `remark-rehype` plugin itself is highly configurable, allowing for custom mappings or handling of unknown markdown nodes.

5.  **HAST Transformations**: Once in HAST, `rehype` plugins can then operate on the HTML structure. This is where HTML-specific processing happens, such as code highlighting, image lazy-loading, or sanitization to remove potentially dangerous HTML. `react-markdown` uses a `rehype` plugin for its `skipHtml` and `unwrap` options.
    ```mermaid
    graph TD
        G[HAST] --> H(rehype-sanitize / other rehype plugins)
        H --> I[Processed HAST]
    ```
    This processed HAST is then passed to `react-markdown`'s internal rendering mechanism.

The core idea is a clear separation of concerns: `remark` deals with markdown, `rehype` deals with HTML, and `remark-rehype` bridges the two. This modularity is a hallmark of the `unified` ecosystem and allows `react-markdown` to be incredibly flexible and extensible.

---

### System Integration

HAST is a critical intermediary in the `react-markdown`'s data flow, serving as the standardized representation right before content is turned into interactive React elements. It sits between the complete markdown processing pipeline and the final rendering stage.

Let's look at its integration points:

1.  **Input from [Unified Processor](chapter_05.md)**: The `unified` processor orchestrates the entire pipeline. After it's done with all `remark` plugins (which process MDAST) and the `remark-rehype` plugin (which converts MDAST to HAST), it then feeds the resulting HAST to any `rehype` plugins. The *final* output of the `unified` processor, as far as `react-markdown`'s rendering component is concerned, is a HAST.

    ```mermaid
    graph TD
        A[Markdown Source] --> B(remark-parse)
        B --> C[MDAST]
        C -- "remark plugins" --> D[Processed MDAST]
        D -- "remark-rehype" --> E[HAST]
        E -- "rehype plugins" --> F[Processed HAST]
    ```

2.  **Output to [Node Renderer](chapter_09.md)**: The `react-markdown` component itself receives this final `Processed HAST`. It then recursively traverses this HAST. For each HAST `element` node, it determines which React component should render it. This mapping is where [Custom Component Mapping](chapter_04.md) plays a crucial role. For example, a HAST node with `tagName: 'h1'` will be rendered by `react-markdown`'s default `<h1>` component, or by a custom component if provided. HAST `text` nodes are rendered directly as text strings.

    ```mermaid
    graph TD
        F[Processed HAST] --> G(ReactMarkdown's Renderer)
        G -- "maps tagName to React Component" --> H[React Elements]
        H --> I[Rendered UI]
    ```

3.  **Interaction with [Plugins](chapter_08.md)**: Plugins can operate at various stages. While `remark` plugins manipulate MDAST, `rehype` plugins specifically operate on HAST. This means you can write plugins that directly modify the HTML structure, add classes, strip attributes, or even completely replace elements based on their `tagName` or `properties`, *before* they are converted to React components. This provides immense power and flexibility for custom HTML transformations.

In essence, HAST is the central data format that `react-markdown` understands for transforming processed content into its final visual representation, making it the linchpin between parsing and rendering.

---

### Best Practices & Tips

Working with HAST effectively can unlock powerful customization options for your `react-markdown` implementation.

*   **Understand HAST Node Structure**: Familiarize yourself with the basic HAST node types: `root`, `element`, `text`, `doctype`, `comment`. Especially important are `element` nodes (which have `tagName`, `properties`, and `children`) and `text` nodes (which have `value`). This knowledge is crucial when writing `rehype` plugins or debugging rendering issues.
*   **Use `rehype` Plugins for HTML Transformations**: If you need to modify the resulting HTML structure *before* it's rendered to React components (e.g., adding `rel="nofollow"` to all external links, syntax highlighting code blocks, lazy-loading images), `rehype` plugins are the way to go. They work directly on the HAST.
*   **Sanitization is Key**: Markdown can often contain raw HTML (e.g., `<div>...</div>`). If `react-markdown` is configured to allow raw HTML (`skipHtml={false}`), this HTML will be converted into HAST nodes. Without proper sanitization, this could lead to XSS vulnerabilities. `react-markdown` uses `rehype-sanitize` by default (or can be configured with specific options) to remove dangerous elements and attributes from the HAST. Always ensure your HAST is sanitized, especially when dealing with untrusted input.
*   **Differentiate MDAST vs. HAST Plugins**: If your transformation is purely markdown-semantic (e.g., converting specific markdown syntax like `==highlight==` into a node), then a `remark` plugin operating on MDAST is appropriate. If you're transforming HTML attributes or elements (e.g., adding `target="_blank"` to all external links, converting `<table>` elements into `<div>`s for responsiveness), then a `rehype` plugin operating on HAST is the correct choice.
*   **Debugging HAST**: When developing `rehype` plugins or trying to understand `react-markdown`'s rendering, it can be helpful to inspect the HAST directly. You can achieve this by logging the output of the `unified` processor before `react-markdown` renders it, or by building a small `unified` pipeline specifically to generate and log the HAST.

    ```javascript
    import { unified } from 'unified';
    import remarkParse from 'remark-parse';
    import remarkRehype from 'remark-rehype';
    import rehypeStringify from 'rehype-stringify'; // To convert HAST back to HTML string
    import rehypeSanitize from 'rehype-sanitize'; // For safety

    async function getHast(markdown) {
      const processor = unified()
        .use(remarkParse) // Parse markdown to MDAST
        .use(remarkRehype) // Convert MDAST to HAST
        .use(rehypeSanitize) // Sanitize the HAST
        // Add any custom rehype plugins here if needed
        // .use(myRehypePlugin)
        .use(rehypeStringify); // To get HTML string, or omit for raw HAST

      const file = await processor.process(markdown);
      console.log('Final HAST (as HTML string):', String(file));
      // To get the raw HAST object before stringification,
      // you'd typically need to inspect the 'file' object before rehypeStringify.
      // For this example, we show the string output.
    }

    getHast('Hello, **world**!');
    // Expected output will be an HTML string: <p>Hello, <strong>world</strong>!</p>
    ```
    This snippet shows how to manually run a `unified` pipeline to see the resulting HTML, which is a direct serialization of the HAST. You can insert console logs within custom `rehype` plugins to inspect the HAST object directly.

---

### Chapter Conclusion

The HTML AST (HAST) is the unsung hero that bridges the gap between markdown processing and HTML rendering within `react-markdown`. By providing a standardized, HTML-centric representation of your content, HAST streamlines the transformation of parsed markdown into interactive React elements. It’s the format that `react-markdown`'s rendering logic truly understands, enabling efficient mapping to components and robust customization through `rehype` plugins. Understanding HAST is key to unlocking the full power and flexibility of `react-markdown`, allowing you to manipulate the final output HTML structure with precision and control.

Now that we have a clear understanding of HAST and its role in preparing content for rendering, we'll dive into the world of `unified` [Plugins](chapter_08.md). These powerful extensions allow us to customize both the MDAST and HAST, injecting custom logic and transformations into any stage of the markdown processing pipeline.
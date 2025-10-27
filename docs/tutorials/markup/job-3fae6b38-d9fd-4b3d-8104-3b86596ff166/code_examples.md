# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: HTTP Request and Response](chapter_01.md)

- **Example 1** (http): GET /articles/getting-started HTTP/1.1
- **Example 2** (python): In a Django view function (e.g., in views.py)
- **Example 3** (http): HTTP/1.1 200 OK
- **Example 4** (mermaid): sequenceDiagram

## [Chapter 2: Django Project Structure](chapter_02.md)

- **Example 1** (bash): Create the 'markup' project in the current directory
- **Example 2** (text): markup/
- **Example 3** (python): From markup/manage.py
- **Example 4** (bash): Run the development server
- **Example 5** (bash): Create a new app named 'core'
- **Example 6** (text): markup/
- **Example 7** (mermaid): sequenceDiagram

## [Chapter 3: Configuration Settings](chapter_03.md)

- **Example 1** (text): markup/
- **Example 2** (python): markup/markup/settings.py
- **Example 3** (python): markup/markup/settings.py
- **Example 4** (python): markup/markup/settings.py - Example for PostgreSQL in production
- **Example 5** (python): markup/markup/settings.py
- **Example 6** (bash): Example of setting the environment variable (usually done by wsgi.py)

## [Chapter 4: URL Dispatcher](chapter_04.md)

- **Example 1** (bash): python manage.py startapp myapp
- **Example 2** (python): markup/settings.py (excerpt)
- **Example 3** (python): myapp/views.py
- **Example 4** (python): myapp/urls.py
- **Example 5** (python): markup/urls.py
- **Example 6** (mermaid): sequenceDiagram

## [Chapter 5: Views (Request Handlers)](chapter_05.md)

- **Example 1** (python): my_django_project/my_django_project/views.py
- **Example 2** (python): my_django_project/my_django_project/views.py
- **Example 3** (python): ... inside handle_post function, after `if request.method == 'POST':`
- **Example 4** (python): ... inside handle_post, after all POST-specific logic ...
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 6: WSGI Server Interface](chapter_06.md)

- **Example 1** (python): markup/markup/wsgi.py
- **Example 2** (bash): In your project's root directory (where manage.py is)
- **Example 3** (python): Simplified conceptual view of what a WSGI server does:
- **Example 4** (mermaid): sequenceDiagram
- **Example 5** (mermaid): graph TD


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

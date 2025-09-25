# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: Project Management Command](chapter_01.md)

- **Example 1** (bash): cd my_django_project/ # Replace with your proje...
- **Example 2** (text): manage.py  my_django_project/  # ... other file...
- **Example 3** (bash): python manage.py runserver
- **Example 4** (text): Watching for file changes with StatReloader
- **Example 5** (bash): python manage.py help
- **Example 6** (bash): python manage.py help runserver
- **Example 7** (python): my_django_project/manage.py
- **Example 8** (mermaid): sequenceDiagram

## [Chapter 2: The Django Project](chapter_02.md)

- **Example 1** (bash): Ensure you are in your project's root directory (e.g., 'markup_project_root/')
- **Example 2** (text): .
- **Example 3** (bash): Make sure you are in the directory containing manage.py
- **Example 4** (text): Watching for file changes with StatReloader
- **Example 5** (mermaid): graph TD
- **Example 6** (python): !/usr/bin/env python
- **Example 7** (python): markup/settings.py (excerpt)
- **Example 8** (python): markup/urls.py
- **Example 9** (python): markup/wsgi.py
- **Example 10** (python): markup/asgi.py

## [Chapter 3: Project Settings](chapter_03.md)

- **Example 1** (python): markup/markup/settings.py
- **Example 2** (python): markup/markup/settings.py
- **Example 3** (python): markup/markup/settings.py
- **Example 4** (python): An excerpt from manage.py (simplified)

## [Chapter 4: URL Routing](chapter_04.md)

- **Example 1** (bash): python manage.py startapp core
- **Example 2** (python): markup/core/urls.py
- **Example 3** (python): markup/core/views.py
- **Example 4** (python): markup/markup/urls.py
- **Example 5** (bash): python manage.py runserver
- **Example 6** (mermaid): sequenceDiagram

## [Chapter 5: Views (Request Handlers)](chapter_05.md)

- **Example 1** (python): my_django_project/my_django_project/views.py
- **Example 2** (python): my_django_project/my_django_project/urls.py
- **Example 3** (python): my_django_project/my_django_project/views.py
- **Example 4** (python): my_django_project/my_django_project/urls.py
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 6: Web Server Gateway Interface (WSGI)](chapter_06.md)

- **Example 1** (python): markup/markup/wsgi.py
- **Example 2** (bash): gunicorn markup.wsgi:application --bind 0.0.0.0...
- **Example 3** (python): Simplified conceptual call within Gunicorn


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

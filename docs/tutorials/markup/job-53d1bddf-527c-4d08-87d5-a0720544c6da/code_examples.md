# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: The Django Project](chapter_01.md)

- **Example 1** (bash): pip install Django
- **Example 2** (bash): django-admin startproject markup .
- **Example 3** (text): .
- **Example 4** (bash): python manage.py runserver
- **Example 5** (text): Watching for file changes with StatReloader
- **Example 6** (mermaid): graph TD

## [Chapter 2: Project Management Script](chapter_02.md)

- **Example 1** (bash): python manage.py runserver
- **Example 2** (bash): python manage.py startapp my_first_app
- **Example 3** (bash): python manage.py help
- **Example 4** (python): def main():
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 3: Project Settings](chapter_03.md)

- **Example 1** (python): markup/settings.py
- **Example 2** (python): markup/settings.py
- **Example 3** (python): markup/settings.py
- **Example 4** (python): markup/settings.py
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 4: View Functions](chapter_04.md)

- **Example 1** (python): my_django_project/my_django_project/views.py
- **Example 2** (python): my_django_project/my_django_project/views.py
- **Example 3** (python): Continuing inside the 'if request.method == 'POST':' block
- **Example 4** (python): Continuing inside the 'if request.method == 'POST':' block
- **Example 5** (python): Continuing inside the 'if request.method == 'POST':' block
- **Example 6** (mermaid): sequenceDiagram

## [Chapter 5: URL Dispatcher (Routing)](chapter_05.md)

- **Example 1** (bash): python manage.py startapp blog
- **Example 2** (python): markup/blog/views.py
- **Example 3** (python): markup/blog/urls.py
- **Example 4** (python): markup/markup/urls.py (Project-level urls.py)
- **Example 5** (mermaid): sequenceDiagram

## [Chapter 6: WSGI Application Gateway](chapter_06.md)

- **Example 1** (python): markup/markup/wsgi.py
- **Example 2** (bash): gunicorn markup.wsgi:application --bind 0.0.0.0...
- **Example 3** (mermaid): sequenceDiagram


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

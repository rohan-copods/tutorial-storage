# Code Examples Index

This index provides a quick reference to all code examples throughout the tutorial.

## [Chapter 1: The 'lodash' Object](chapter_01.md)

- **Example 1** (javascript): // app.js
- **Example 2** (javascript): // Example 2: Filtering active users
- **Example 3** (javascript): // Example 3: Sorting active users by name
- **Example 4** (javascript): // Example 4: Formatting full names
- **Example 5** (mermaid): sequenceDiagram
- **Example 6** (javascript): // Simplified example of lodash/src/head.js
- **Example 7** (javascript): // Simplified example of lodash/index.js
- **Example 8** (javascript): // Example of integration in a typical module

## [Chapter 2: Utility Functions](chapter_02.md)

- **Example 1** (javascript): import _ from 'lodash';
- **Example 2** (javascript): import _ from 'lodash';
- **Example 3** (javascript): import _ from 'lodash';
- **Example 4** (javascript): // A simplified conceptual implementation of Lo...
- **Example 5** (mermaid): graph TD
- **Example 6** (mermaid): graph LR

## [Chapter 3: Collection Operations](chapter_03.md)

- **Example 1** (javascript): const users = [
- **Example 2** (javascript): // Logs each user's name
- **Example 3** (javascript): const activeUsers = _.filter(users, { active: t...
- **Example 4** (javascript): const userNames = _.map(activeUsers, 'name');
- **Example 5** (javascript): const usersByDepartment = _.groupBy(users, 'dep...
- **Example 6** (javascript): const totalActiveUsers = _.reduce(users, (sum, ...
- **Example 7** (mermaid): sequenceDiagram

## [Chapter 4: Property Access and Manipulation](chapter_04.md)

- **Example 1** (javascript): const user1 = {
- **Example 2** (javascript): // Accessing primary email for user2 (missing e...
- **Example 3** (javascript): // Accessing primary email for user3 (missing c...
- **Example 4** (javascript): const newUser = { id: 4 };
- **Example 5** (javascript): const product = {
- **Example 6** (javascript): const settings = {
- **Example 7** (mermaid): sequenceDiagram

## [Chapter 5: Chaining Operations](chapter_05.md)

- **Example 1** (javascript): // Problematic non-chained example
- **Example 2** (javascript): const users = [
- **Example 3** (javascript): // Chained example using _() and .value()
- **Example 4** (javascript): // Explicit chaining with _.chain()
- **Example 5** (javascript): // Conceptual:
- **Example 6** (text): While the above describes the eager evaluation ...

## [Chapter 6: Functional Programming (FP) Variant](chapter_06.md)

- **Example 1** (javascript): // Import the FP variant of Lodash
- **Example 2** (javascript): // Example of currying: _fp.filter takes the pr...
- **Example 3** (javascript): // Example of currying and data-last with _fp.m...
- **Example 4** (javascript): // Example of _fp.sortBy (or _fp.sort for simpl...
- **Example 5** (javascript): // Composing functions with _fp.flow
- **Example 6** (mermaid): sequenceDiagram
- **Example 7** (javascript): import _fp from 'lodash/fp';


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

# Next.js API Routes: Building Backend Endpoints within Your Frontend Project

Next.js API Routes offer a powerful abstraction that allows developers to create server-side endpoints directly within their Next.js application. Essentially, they transform a Next.js project into a full-stack framework by providing a way to handle backend logic without the need for a separate server. This means you can build a robust API for your frontend to consume, all residing within the same codebase and deployed as a single unit.

The primary purpose of API Routes is to bridge the gap between your client-side user interface and server-side operations. This includes tasks such as fetching data from a database, processing form submissions, securely interacting with external APIs (like payment gateways or authentication services), or performing any computation that requires server-side execution. By co-locating your frontend and backend code, API Routes streamline development, simplify deployment, and maintain a consistent project structure.

Behind the scenes, each file within the 'pages/api' directory (for the Pages Router) or 'app/api' directory (for the App Router) automatically becomes an API endpoint. When a request is made to a specific API Route URL, Next.js executes the corresponding file as a serverless function. This function receives standard Node.js 'req' (request) and 'res' (response) objects (or standard Web API 'Request' and 'Response' objects in the App Router), allowing you to read request data and send back appropriate responses, typically in JSON format.


## Architecture

```mermaid
flowchart TD
    NextjsAPIRoutes["Next.js API Routes"]
    classDef highlight fill:#e1f5fe,stroke:#01579b,stroke-width:3px
    class NextjsAPIRoutes highlight
    MultiTenantRouting["Multi-Tenant Routing"]
    DynamicDocumentation["Dynamic Documentation Pages"]
    DocumentationContent["Documentation Content Source"]
    MDXCustomComponents["MDX Custom Components"]
    DocumentationNavigat["Documentation Navigation & Layout"]
    DocumentationVersion["Documentation Versioning"]
    StaticFileServingOpe["Static File Serving (OpenAPI Specs)"]
    DocumentationSearch["Documentation Search"]

    %% Relationships
    MDXCustomComponents -->|"consumes data from"| NextjsAPIRoutes
    NextjsAPIRoutes -->|"provides data to"| MDXCustomComponents
    NextjsAPIRoutes -->|"powers the backend for"| DocumentationSearch
    DocumentationSearch -->|"leverages backend via"| NextjsAPIRoutes
```

## Code Examples

### Fetching Project Details with a GET Request

This example demonstrates a basic Next.js API Route designed to handle 'GET' requests.
It's located at 'pages/api/projects/[id].ts', meaning it can be accessed via URLs
like '/api/projects/1' or '/api/projects/2'. The route retrieves an 'id'
from the request query parameters, simulates fetching project data from a source (like a database),
and responds with the project details as JSON. If the project is not found, it sends a 404 status.
It also ensures that only 'GET' requests are allowed for this endpoint.


```typescript
// pages/api/projects/[id].ts (Pages Router example)
import type { NextApiRequest, NextApiResponse } from 'next';

type Project = {
  id: string;
  name: string;
  description: string;
};

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<Project | { message: string }>
) {
  const { id } = req.query; // Extract 'id' from the URL query

  if (req.method === 'GET') {
    // In a real application, you would fetch this from a database
    const projects: Project[] = [
      { id: '1', name: 'Next.js App', description: 'A server-side rendered application.' },
      { id: '2', name: 'React Component Library', description: 'Reusable UI components.' },
    ];

    const project = projects.find(p => p.id === id);

    if (project) {
      res.status(200).json(project); // Respond with found project
    } else {
      res.status(404).json({ message: 'Project not found' }); // Respond with 404 if not found
    }
  } else {
    // If the method is not GET, send 'Method Not Allowed'
    res.setHeader('Allow', ['GET']);
    res.status(405).end('Method Not Allowed');
  }
}

```

### Creating a New User with a POST Request

This code snippet illustrates how to create an API Route that handles 'POST' requests.
Located at 'pages/api/users.ts', it is designed to receive data in the request body
(e.g., 'name' and 'email') to create a new user. The route checks for required fields,
simulates saving the new user (e.g., to a database), and responds with the newly created
user data and a 201 'Created' status. It also includes basic validation and method restriction.


```typescript
// pages/api/users.ts (Pages Router example)
import type { NextApiRequest, NextApiResponse } from 'next';

type User = {
  id: string;
  name: string;
  email: string;
};

type Data = {
  user?: User;
  message: string;
};

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<Data>
) {
  if (req.method === 'POST') {
    const { name, email } = req.body; // Access the request body

    // Basic validation for required fields
    if (!name || !email) {
      return res.status(400).json({ message: 'Name and email are required.' });
    }

    // In a real application, you would save this to a database
    const newUser: User = {
      id: String(Date.now()), // Simple ID generation
      name,
      email,
    };

    console.log('New user created:', newUser);

    // Respond with 201 Created status and the new user
    res.status(201).json({ user: newUser, message: 'User created successfully.' });
  } else {
    // If the method is not POST, send 'Method Not Allowed'
    res.setHeader('Allow', ['POST']);
    res.status(405).end('Method Not Allowed');
  }
}

```

### Searching an External API with Error Handling

This example showcases an API Route that performs a more complex task:
searching an external API. It handles a 'GET' request, validates the 'query'
parameter, and then attempts to 'fetch' data from a third-party service.
Crucially, it demonstrates robust error handling, including 'try...catch' blocks
for network requests, checking the 'ok' status of the external API response,
and returning appropriate HTTP status codes (e.g., 400 for bad input, 500 for
internal server errors, or reflecting the external API's status) along with informative
error messages to the client.


```typescript
// pages/api/search.ts (Pages Router example)
import type { NextApiRequest, NextApiResponse } from 'next';

type SearchResult = {
  title: string;
  url: string;
  description: string;
};

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<SearchResult[] | { message: string }>
) {
  if (req.method !== 'GET') {
    res.setHeader('Allow', ['GET']);
    return res.status(405).end('Method Not Allowed');
  }

  const { query } = req.query;

  if (!query || typeof query !== 'string') {
    return res.status(400).json({ message: 'A search query is required.' });
  }

  try {
    // Simulate an external API call
    const externalApiResponse = await fetch(`https://api.example.com/search?q=${encodeURIComponent(query)}', {
      headers: {
        'Authorization': 'Bearer YOUR_EXTERNAL_API_KEY', // Use environment variables for real keys
      }
    });

    if (!externalApiResponse.ok) {
      // Handle non-2xx responses from the external API
      const errorData = await externalApiResponse.json();
      console.error('External API error:', errorData);
      return res.status(externalApiResponse.status).json({ message: 'Failed to fetch search results: ${errorData.message || 'Unknown error'}' });
    }

    const data = await externalApiResponse.json();

    // Transform data if necessary before sending to client
    const searchResults: SearchResult[] = data.results.map((item: any) => ({
      title: item.title,
      url: item.link,
      description: item.snippet,
    }));

    res.status(200).json(searchResults);

  } catch (error: any) {
    // Catch network errors or other unexpected issues
    console.error('API Route error:', error);
    res.status(500).json({ message: 'Internal server error: ${error.message}' });
  }
}

```

## Implementation

Next.js API Routes are implemented as files within specific directories. For the Pages Router, these files reside in 'pages/api/'. Each file exports a default asynchronous function that accepts 'NextApiRequest' and 'NextApiResponse' objects. The filename or directory structure determines the API endpoint's URL (e.g., 'pages/api/users.ts' becomes '/api/users', and 'pages/api/users/[id].ts' becomes '/api/users/:id'). In the App Router, API Routes are defined in 'app/api/' directories, and you export functions named after HTTP methods (e.g., 'GET', 'POST') which receive standard Web API 'Request' and return standard 'Response' objects.

Best practices include method-based routing (checking 'req.method' or using method-specific exports in App Router), robust input validation and sanitization, and comprehensive error handling with appropriate HTTP status codes (e.g., 400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Internal Server Error). For security, always use environment variables for sensitive data like API keys, and avoid exposing them directly in client-side code. API Routes are serverless functions, meaning they are stateless and scale automatically, making them efficient for on-demand execution. Consider using libraries for database interaction (e.g., Prisma, Drizzle ORM) and authentication flows within these routes to build secure and scalable backend logic.


## Related Concepts

- Serverless Functions

- HTTP Methods (GET, POST, PUT, DELETE)

- RESTful APIs

- JSON

- Database Integration

- Authentication & Authorization

- Environment Variables

- Next.js Data Fetching (client-side and server-side)

- Next.js Middleware

- Frontend Routing

- Node.js

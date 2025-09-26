# Chapter 3: Global State Management (Contexts)

In the previous chapter, [Backend Data Services](chapter_02.md), we established a robust layer for interacting with Firebase Firestore, providing an abstract way to fetch, add, update, and delete our core data models. These services are crucial for persistence, but they don't inherently address how this data is shared and managed across various parts of our React application's user interface. As we build out a complex HR application, we'll encounter scenarios where the same pieces of information need to be accessible by many different components, potentially deeply nested within the component tree.

---

### Problem & Motivation

Imagine our 'hr-app' where a user logs in. The `Navbar` component needs to display the logged-in user's name, the `Dashboard` component needs their role to conditionally render features, and a `Profile` page needs all their details. Without a centralized state mechanism, we would have to fetch the user data in our main `App` component and then "prop drill" it down through multiple layers of components to reach the ones that need it. This quickly becomes cumbersome, error-prone, and makes components unnecessarily aware of data they don't directly use, leading to rigid and hard-to-maintain code.

The specific problem React Contexts solve in our 'hr-app' is avoiding *prop drilling* for globally relevant data like the current authenticated user, the list of employees, or system-wide settings. For instance, if an `EmployeeList` component fetches all employees, and then a `DeleteEmployeeButton` inside an `EmployeeCard` (which is inside `EmployeeListItem` inside `EmployeeList`) needs to trigger a re-fetch of the employee list or directly remove an employee from the global state, passing the `setEmployees` function down through multiple layers is impractical. Contexts allow us to make this employee data and the functions to modify it directly available where needed.

---

### Core Concept Explanation

React's Context API provides a way to pass data through the component tree without having to pass props down manually at every level. It's designed to share "global" data – such as the current authenticated user, theme settings, or cached data – with a tree of React components. In essence, Context acts like a designated "data channel" that components can subscribe to, gaining access to the shared information without explicit prop passing.

At its heart, the Context API consists of three main parts:
1.  **`React.createContext`**: This function creates a Context object. It takes an optional default value, which is used when a component consumes the context without a matching Provider above it in the tree.
2.  **`Context.Provider`**: This is a React component that "provides" the value to its children components. Any component inside the Provider's tree, no matter how deep, can access the value passed to the Provider's `value` prop.
3.  **`useContext` Hook**: This hook, used within a functional component, allows it to "consume" the value provided by the nearest `Context.Provider` above it in the component tree. It returns the current context value.

For our 'hr-app', we'll use Contexts to manage the state of the currently logged-in user and the list of all employees. This approach keeps our UI components lean, as they only need to declare their need for specific global data rather than managing it themselves or receiving it via tedious prop chains.

---

### Practical Usage Examples

Let's illustrate how to set up an `AuthContext` to manage the currently logged-in user. We'll start by creating the context and a provider component.

First, creating the context itself:

```javascript
// src/contexts/AuthContext.js
import React, { createContext, useState, useEffect } from 'react';

const AuthContext = createContext(null); // Default value is null
export default AuthContext;
```
*Explanation*: We import `createContext` from React and call it to create our `AuthContext` object. We initialize it with `null`, which will be the value if no `Provider` is found.

Next, we'll create an `AuthProvider` component to manage and provide the authentication state:

```javascript
// src/contexts/AuthContext.js (continued)
// ... (imports and AuthContext definition)

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null); // State for current user
  const [loading, setLoading] = useState(true); // Loading state for auth check

  // useEffect for initial auth check (e.g., from session storage)
  useEffect(() => {
    // Simulate fetching user from session/localStorage
    const storedUser = JSON.parse(localStorage.getItem('hr-app-user'));
    if (storedUser) {
      setUser(storedUser);
    }
    setLoading(false);
  }, []);

  const login = (userData) => { /* ... login logic ... */ setUser(userData); };
  const logout = () => { /* ... logout logic ... */ setUser(null); };

  const contextValue = { user, loading, login, logout };

  return (
    <AuthContext.Provider value={contextValue}>
      {!loading && children} {/* Render children only after loading */}
    </AuthContext.Provider>
  );
};
```
*Explanation*: The `AuthProvider` is a regular React component that wraps its `children` with `AuthContext.Provider`. It uses `useState` to manage the `user` and `loading` states. The `useEffect` simulates an initial check for a logged-in user. The `login` and `logout` functions will be defined more fully later. The `value` prop of `AuthContext.Provider` is an object containing `user`, `loading`, `login`, and `logout`, making them available to any component inside this provider.

Now, to make this context available throughout our application, we need to wrap our root `App` component with `AuthProvider`:

```javascript
// src/App.js
import React from 'react';
import { AuthProvider } from './contexts/AuthContext';
import AppRouter from './router/AppRouter'; // Assuming our router is here

function App() {
  return (
    <AuthProvider>
      <AppRouter /> {/* All components inside AppRouter can access AuthContext */}
    </AuthProvider>
  );
}

export default App;
```
*Explanation*: By placing `<AuthProvider>` around `<AppRouter>`, every component rendered by our router (and thus, every page and component in our application) will have access to the `AuthContext`'s value.

Finally, consuming the context in a component, for example, a `Navbar` to display the user's name:

```javascript
// src/components/Navbar.js
import React, { useContext } from 'react';
import AuthContext from '../contexts/AuthContext';

function Navbar() {
  const { user, logout } = useContext(AuthContext);

  if (!user) return null; // Don't show if no user is logged in

  return (
    <nav>
      <span>Welcome, {user.name}!</span>
      <button onClick={logout}>Logout</button>
    </nav>
  );
}

export default Navbar;
```
*Explanation*: The `Navbar` component uses the `useContext` hook, passing `AuthContext` to it. This immediately gives us access to the `user` object and the `logout` function provided by the `AuthProvider` higher up in the tree. We can then display the user's name and provide a logout button without any props being passed to `Navbar`.

---

### Internal Implementation Walkthrough

Let's dive deeper into how `AuthProvider` works and consider a more complete example that integrates with our [Backend Data Services](chapter_02.md).

Our `AuthContext.js` file will house both the context object and the `AuthProvider` component. The `AuthProvider` will typically manage its own state using React's `useState` or `useReducer` hooks, along with `useEffect` for side effects like initial data loading or interactions with external services.

1.  **State Management within `AuthProvider`**:
    The `AuthProvider` will hold the `user` state. It also needs to provide functions like `login`, `register`, and `logout` that interact with our `authService` (from Chapter 2) and then update this internal `user` state.

    ```javascript
    // src/contexts/AuthContext.js
    import React, { createContext, useState, useEffect, useCallback } from 'react';
    import { authService } from '../services/authService'; // From Chapter 2

    const AuthContext = createContext(null);

    export const AuthProvider = ({ children }) => {
      const [user, setUser] = useState(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);

      // Effect to check for authenticated user on initial load
      useEffect(() => {
        const checkAuthStatus = async () => {
          try {
            setLoading(true);
            const currentUser = await authService.getCurrentUser(); // Checks session/token
            if (currentUser) {
              setUser(currentUser);
            }
          } catch (err) {
            console.error("Auth check failed:", err);
            setUser(null);
          } finally {
            setLoading(false);
          }
        };
        checkAuthStatus();
      }, []); // Run once on mount

      // Login function that uses backend service
      const login = useCallback(async (email, password) => {
        setLoading(true);
        setError(null);
        try {
          const userData = await authService.login(email, password);
          setUser(userData);
          return true;
        } catch (err) {
          setError(err.message);
          return false;
        } finally {
          setLoading(false);
        }
      }, []);

      // Logout function
      const logout = useCallback(async () => {
        setLoading(true);
        setError(null);
        try {
          await authService.logout();
          setUser(null);
          return true;
        } catch (err) {
          setError(err.message);
          return false;
        } finally {
          setLoading(false);
        }
      }, []);

      const contextValue = {
        user,
        loading,
        error,
        login,
        logout,
        // register: async (details) => { /* ... */ } // Could add register
      };

      return (
        <AuthContext.Provider value={contextValue}>
          {children}
        </AuthContext.Provider>
      );
    };

    export const useAuth = () => { // Custom hook for convenience
      const context = useContext(AuthContext);
      if (context === undefined) {
        throw new Error('useAuth must be used within an AuthProvider');
      }
      return context;
    };
    ```

    *Explanation*:
    *   We import `authService` to interact with our backend.
    *   `useState` hooks manage the current `user`, `loading` status, and any `error` messages.
    *   `useEffect` runs once on component mount to check if a user is already authenticated (e.g., from a persistent session or token).
    *   `login` and `logout` are `useCallback` wrapped functions. They call the respective `authService` methods, update the internal `user` state, and handle loading/error states. `useCallback` helps optimize performance by preventing unnecessary re-creations of these functions.
    *   The `contextValue` object bundles `user`, `loading`, `error`, `login`, and `logout` to be passed to the `AuthContext.Provider`.
    *   A custom `useAuth` hook is exported for simpler consumption of the context (e.g., `const { user, login } = useAuth();`).

2.  **Interaction Flow**:

    ```mermaid
    sequenceDiagram
        participant App as App.js
        participant AuthProvider as AuthProvider (Context)
        participant authService as Backend Data Services
        participant UserComponent as User Component (e.g., Navbar)

        App->>AuthProvider: Renders AuthProvider
        AuthProvider->>authService: useEffect: getCurrentUser()
        authService-->>AuthProvider: Returns currentUser (or null)
        AuthProvider->>AuthProvider: Updates 'user', 'loading' state
        AuthProvider->>UserComponent: Provides context value {user, login, logout}

        UserComponent->>AuthProvider: useContext(AuthContext) (via useAuth hook)
        UserComponent->>AuthProvider: Calls login(email, pass)
        AuthProvider->>authService: Calls authService.login(email, pass)
        authService-->>AuthProvider: Returns user data
        AuthProvider->>AuthProvider: Updates 'user' state
        AuthProvider->>UserComponent: Context value updates (user changes)

        UserComponent->>AuthProvider: Calls logout()
        AuthProvider->>authService: Calls authService.logout()
        authService-->>AuthProvider: Confirms logout
        AuthProvider->>AuthProvider: Updates 'user' to null
        AuthProvider->>UserComponent: Context value updates (user is null)
    ```
    *Explanation*: The `App` component renders `AuthProvider`. `AuthProvider` then uses `useEffect` to check the current authentication status with `authService`. Once loaded, it provides the `user` object and functions (`login`, `logout`) to any `UserComponent` that consumes `AuthContext`. When `login` or `logout` is called from a `UserComponent`, `AuthProvider` interacts with `authService`, updates its internal state, and automatically triggers re-renders in all consuming components.

---

### API & Interface Reference

The primary interface for interacting with our `AuthContext` is through the custom `useAuth` hook.

#### `useAuth()` hook

**Purpose**: Provides convenient access to the authentication state and actions globally.

**Returns**: An object with the following properties:
*   `user`: (`Object | null`) The currently authenticated user object (e.g., `{ id: '123', name: 'John Doe', email: 'john@hr.com', role: 'admin' }`) or `null` if no user is logged in.
*   `loading`: (`boolean`) Indicates if authentication-related operations (like initial check, login, logout) are currently in progress.
*   `error`: (`string | null`) Any error message resulting from authentication operations, or `null` if no error.
*   `login`: (`Function`) An asynchronous function `(email, password) => Promise<boolean>` to attempt user login. Returns `true` on success, `false` on failure.
*   `logout`: (`Function`) An asynchronous function `() => Promise<boolean>` to log out the current user. Returns `true` on success, `false` on failure.

**Example Usage**:
```javascript
import { useAuth } from '../contexts/AuthContext';

function UserProfileDisplay() {
  const { user, loading, error, logout } = useAuth();

  if (loading) return <p>Loading user data...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!user) return <p>Please log in.</p>;

  return (
    <div>
      <h3>Hello, {user.name}</h3>
      <p>Role: {user.role}</p>
      <button onClick={logout}>Sign Out</button>
    </div>
  );
}
```
*Explanation*: This component easily accesses the `user` object and `logout` function without receiving them as props, making it reusable anywhere in the application.

---

### Configuration & Usage Patterns

To manage multiple types of global state, you can nest multiple Providers. For our 'hr-app', we'll likely need an `AuthContext` and an `EmployeeContext` (for managing the list of all employees and related actions).

```javascript
// src/contexts/EmployeeContext.js
import React, { createContext, useState, useEffect, useCallback } from 'react';
import { employeeApiService } from '../services/employeeApiService'; // From Chapter 2

const EmployeeContext = createContext(null);

export const EmployeeProvider = ({ children }) => {
  const [employees, setEmployees] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchEmployees = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const data = await employeeApiService.getAllEmployees();
      setEmployees(data);
    } catch (err) {
      setError(err.message);
      console.error("Error fetching employees:", err);
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchEmployees(); // Fetch employees on initial mount
  }, [fetchEmployees]);

  const addEmployee = useCallback(async (newEmployee) => {
    try {
      const added = await employeeApiService.addEmployee(newEmployee);
      setEmployees(prev => [...prev, added]); // Optimistic update
      return true;
    } catch (err) {
      setError(err.message);
      return false;
    }
  }, []);

  const updateEmployee = useCallback(async (id, updates) => {
    try {
      const updated = await employeeApiService.updateEmployee(id, updates);
      setEmployees(prev => prev.map(emp => emp.id === id ? updated : emp));
      return true;
    } catch (err) {
      setError(err.message);
      return false;
    }
  }, []);

  // Other functions: deleteEmployee, getEmployeeById etc.

  const contextValue = {
    employees,
    loading,
    error,
    fetchEmployees,
    addEmployee,
    updateEmployee,
  };

  return (
    <EmployeeContext.Provider value={contextValue}>
      {children}
    </EmployeeContext.Provider>
  );
};

export const useEmployees = () => { // Custom hook
  const context = useContext(EmployeeContext);
  if (context === undefined) {
    throw new Error('useEmployees must be used within an EmployeeProvider');
  }
  return context;
};
```
*Explanation*: Similar to `AuthContext`, this sets up an `EmployeeContext` with a `Provider` that manages the `employees` array, `loading` status, and provides functions to interact with `employeeApiService`. A `useEmployees` hook simplifies consumption.

Then, we would wrap our `App` with both providers:

```javascript
// src/App.js
import React from 'react';
import { AuthProvider } from './contexts/AuthContext';
import { EmployeeProvider } from './contexts/EmployeeContext';
import AppRouter from './router/AppRouter';

function App() {
  return (
    <AuthProvider>
      <EmployeeProvider> {/* Nested Provider */}
        <AppRouter />
      </EmployeeProvider>
    </AuthProvider>
  );
}

export default App;
```
*Explanation*: `EmployeeProvider` is nested inside `AuthProvider`. This means that any component within `AppRouter` can access *both* `AuthContext` and `EmployeeContext`. The order generally doesn't matter unless one context strictly depends on another (e.g., `EmployeeContext` might need the authenticated user's ID to fetch specific data, in which case `AuthProvider` should wrap `EmployeeProvider`).

---

### System Integration

Global state management contexts form a critical bridge between our [Backend Data Services](chapter_02.md) and our [User Interface Components](chapter_04.md).

*   **With Backend Data Services**: As shown in `AuthProvider` and `EmployeeProvider`, contexts are the primary consumers of our backend services. They encapsulate the logic for calling API endpoints (e.g., `authService.login`, `employeeApiService.getAllEmployees`), handling loading/error states, and updating the application's global state. This separation of concerns means UI components don't directly talk to the backend; they interact with the context's provided functions, which then manage the data flow.

    ```mermaid
    graph TD
        subgraph UI Components
            A[User Profile]
            B[Employee List]
            C[Add Employee Form]
        end

        subgraph Global State Management
            D[useAuth Hook]
            E[AuthContext.Provider]
            F[useEmployees Hook]
            G[EmployeeContext.Provider]
        end

        subgraph Backend Data Services
            H[Auth Service]
            I[Employee API Service]
        end

        A --> D: Consumes user, logout
        B --> F: Consumes employees, fetchEmployees
        C --> F: Consumes addEmployee

        D -.-> E: Accesses AuthContext
        F -.-> G: Accesses EmployeeContext

        E --> H: Calls login/logout
        G --> I: Calls getAllEmployees/addEmployee

        H -->> E: Returns auth status/user
        I -->> G: Returns employee data/status
    ```
    *Explanation*: UI components use custom hooks (`useAuth`, `useEmployees`) to access state and functions from the respective Context Providers. These Providers, in turn, leverage the `Auth Service` and `Employee API Service` to perform backend operations, then update their internal state which automatically reflects in the consuming UI components.

*   **With User Interface Components**: Contexts make data available to components, regardless of their depth. This drastically simplifies component APIs. Instead of `props.user` and `props.onLogout`, a component simply calls `useAuth()`. This makes components more reusable and less coupled to their parent's data fetching logic. The next chapter, [User Interface Components](chapter_04.md), will extensively demonstrate how components leverage these contexts.

---

### Best Practices & Tips

*   **Separate Concerns**: Design contexts to manage specific, related pieces of state. Avoid creating a single "mega-context" that holds all global state, as this can lead to performance issues (components re-rendering for irrelevant state changes) and make the context harder to maintain.
*   **Use `useCallback` and `useMemo` for Provider Values**: If the `value` object passed to `Context.Provider` is not memoized, all consuming components will re-render every time the parent `Provider` re-renders, even if the actual data hasn't changed. Wrap functions with `useCallback` and the `value` object with `useMemo` to prevent unnecessary re-renders.
*   **Create Custom Hooks for Consumption**: Instead of `const { user } = useContext(AuthContext);`, create a custom hook like `export const useAuth = () => useContext(AuthContext);`. This provides a cleaner API, centralizes error handling for missing providers, and can be extended with additional derived logic.
*   **Context for Infrequently Changing Data**: React Context is excellent for data that changes infrequently (like user authentication status) or for state that needs to be available globally without performance-critical updates. For very high-frequency updates or complex state logic, `useReducer` within a context or dedicated state management libraries might be considered, though for 'hr-app', `useState` and `useReducer` within contexts are sufficient.
*   **Avoid Over-Contextualizing**: Not every piece of state needs to be global. If data is only used by a few closely related components, passing props might still be simpler and more performant.

---

### Chapter Conclusion

This chapter introduced React's Context API as our primary mechanism for global state management in the 'hr-app'. We've explored how Contexts solve the problem of prop drilling, enabling us to efficiently share data like user authentication status and employee lists across our application. We walked through the implementation of `AuthContext` and `EmployeeContext`, showcasing how they integrate with our [Backend Data Services](chapter_02.md) and provide a clean interface for our UI components. By following best practices, we ensure our global state management is performant, maintainable, and scalable.

With our global state infrastructure in place, our application is now ready to build out the visual elements. The next chapter, [User Interface Components](chapter_04.md), will focus on designing and implementing the individual building blocks of our application's UI, which will extensively consume the global state provided by these contexts.
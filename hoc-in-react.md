# Higher-Order Components (HOC) in React

A **Higher-Order Component (HOC)** is an advanced React pattern for reusing component logic. A HOC is a **function** that takes a component as an argument and returns a **new component** with additional props, behavior, or data — without modifying the original component.

```
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

## Table of Contents

- [Why HOCs?](#why-hocs)
- [Anatomy of a HOC](#anatomy-of-a-hoc)
- [Example 1: withLoading](#example-1-withloading)
- [Example 2: withAuth](#example-2-withauth)
- [Example 3: withLogger](#example-3-withlogger)
- [Example 4: Composing Multiple HOCs](#example-4-composing-multiple-hocs)
- [Passing Props Through](#passing-props-through)
- [Naming and Display Name](#naming-and-display-name)
- [HOC vs Custom Hooks vs Render Props](#hoc-vs-custom-hooks-vs-render-props)

## Why HOCs?

Before Hooks existed, HOCs (along with Render Props) were the main way to share logic like data fetching, authentication checks, or subscriptions across multiple components. HOCs let you:

- **Reuse cross-cutting logic** (auth checks, logging, data fetching) across many components.
- **Keep components focused** by pulling shared behavior into a separate, composable function.
- **Enhance components without editing their source** — useful for third-party or shared components.

You'll still encounter HOCs in real codebases and libraries (e.g., `connect()` from Redux, `withRouter()` from older React Router versions), so understanding the pattern is valuable even though custom Hooks are now preferred for most new code.

## Anatomy of a HOC

A HOC is just a function with this shape:

```jsx
function withSomething(WrappedComponent) {
  return function EnhancedComponent(props) {
    // add extra logic, state, or props here
    return <WrappedComponent {...props} />;
  };
}
```

Key characteristics:

1. It's a **plain function**, not a component itself.
2. It **takes a component** as input.
3. It **returns a new component** that renders the original, usually with extra props.
4. The original component is left **unmodified**.

## Example 1: withLoading

Adds a loading state wrapper around any component.

```jsx
function withLoading(WrappedComponent) {
  return function WithLoading({ isLoading, ...props }) {
    if (isLoading) {
      return <p>Loading...</p>;
    }
    return <WrappedComponent {...props} />;
  };
}

// Usage
function UserList({ users }) {
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);

// In a parent component:
<UserListWithLoading isLoading={loading} users={users} />;
```

## Example 2: withAuth

Redirects or blocks rendering if the user isn't authenticated — commonly used for protected routes/pages.

```jsx
import { Navigate } from "react-router-dom";

function withAuth(WrappedComponent) {
  return function WithAuth(props) {
    const isAuthenticated = Boolean(localStorage.getItem("authToken"));

    if (!isAuthenticated) {
      return <Navigate to="/login" replace />;
    }

    return <WrappedComponent {...props} />;
  };
}

// Usage
function Dashboard() {
  return <h1>Welcome to your Dashboard</h1>;
}

const ProtectedDashboard = withAuth(Dashboard);
```

## Example 3: withLogger

Logs when a component mounts and unmounts — useful for debugging or analytics.

```jsx
import { useEffect } from "react";

function withLogger(WrappedComponent) {
  return function WithLogger(props) {
    useEffect(() => {
      console.log(`${WrappedComponent.name} mounted`);
      return () => console.log(`${WrappedComponent.name} unmounted`);
    }, []);

    return <WrappedComponent {...props} />;
  };
}

// Usage
function Profile({ name }) {
  return <p>Profile: {name}</p>;
}

const ProfileWithLogger = withLogger(Profile);
```

## Example 4: Composing Multiple HOCs

HOCs can be layered together to combine behaviors.

```jsx
const EnhancedProfile = withAuth(withLoading(withLogger(Profile)));
```

To avoid deeply nested, hard-to-read wrapping, a small `compose` utility (similar to Redux's `compose`) is commonly used:

```jsx
function compose(...fns) {
  return (component) => fns.reduceRight((acc, fn) => fn(acc), component);
}

const EnhancedProfile = compose(withAuth, withLoading, withLogger)(Profile);
```

## Passing Props Through

A well-behaved HOC should always forward unrelated props to the wrapped component, so it stays flexible and composable:

```jsx
function withExtraProp(WrappedComponent) {
  return function Enhanced(props) {
    return <WrappedComponent extra="some value" {...props} />;
  };
}
```

Placing `{...props}` **after** any props you add ensures the caller can still override them if needed.

## Naming and Display Name

Since a HOC wraps a component in an anonymous function, React DevTools shows a generic name unless you set `displayName` explicitly:

```jsx
function withLoading(WrappedComponent) {
  function WithLoading(props) {
    // ...
    return <WrappedComponent {...props} />;
  }

  WithLoading.displayName = `WithLoading(${WrappedComponent.displayName || WrappedComponent.name || "Component"})`;

  return WithLoading;
}
```

This makes debugging in DevTools much easier — you'll see `WithLoading(UserList)` instead of an unnamed component.

## HOC vs Custom Hooks vs Render Props

| Feature           | HOC                                          | Custom Hook                                      | Render Props                                        |
| ----------------- | -------------------------------------------- | ------------------------------------------------ | --------------------------------------------------- |
| Shape             | Function that returns a new component        | Function that returns values/state               | Component with a function-as-child or a render prop |
| Adds to tree      | Yes — wraps component in an extra layer      | No — logic only, no extra component              | Yes — an extra wrapping component                   |
| Composability     | Can get messy with many nested HOCs          | Easy — just call multiple hooks in one component | Can lead to "wrapper hell" similar to HOCs          |
| Modern preference | Legacy pattern, still used in some libraries | ✅ Preferred for new code                        | Largely replaced by hooks                           |

```jsx
// The same "loading" logic as a custom hook instead of a HOC:
function useLoading(initial = false) {
  const [isLoading, setIsLoading] = useState(initial);
  return { isLoading, setIsLoading };
}

function UserList({ users }) {
  const { isLoading } = useLoading(true);
  if (isLoading) return <p>Loading...</p>;
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

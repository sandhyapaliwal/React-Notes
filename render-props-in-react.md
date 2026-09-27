# Render Props in React

**Render Props** is a pattern for sharing code between React components using a prop whose value is a function. The component with the render prop calls this function instead of implementing its own rendering logic — letting the consumer decide what to render with the data or behavior provided.

> Note: Render Props is a **pattern**, not a React API — it predates Hooks and was the standard way to share stateful logic before custom Hooks became common.

## Table of Contents

- [Why Render Props?](#why-render-props)
- [Basic Concept](#basic-concept)
- [Example 1: Mouse Tracker](#example-1-mouse-tracker)
- [Example 2: Toggle Component](#example-2-toggle-component)
- [Example 3: Data Fetching with Render Props](#example-3-data-fetching-with-render-props)
- [Using children as a Function](#using-children-as-a-function)
- [Render Props vs Custom Hooks](#render-props-vs-custom-hooks)
- [Render Props vs Higher-Order Components (HOCs)](#render-props-vs-higher-order-components-hocs)

## Why Render Props?

Before Hooks existed, sharing stateful logic (mouse position, window size, auth state, data fetching) between components was awkward. Two patterns emerged to solve this:

- **Higher-Order Components (HOCs)** — functions that wrap a component and inject props.
- **Render Props** — components that accept a function prop and call it with internal state/data.

Render Props solved the "wrapper hell" and naming-collision issues of HOCs by making the data flow explicit — you can see exactly what's being passed down, right where it's used, instead of it being injected invisibly.

## Basic Concept

A component using the render props pattern takes a **function as a prop** and calls that function during render, instead of hard-coding what to display.

```jsx
function DataProvider({ render }) {
  const data = { message: "Hello from DataProvider" };
  return render(data);
}

function App() {
  return <DataProvider render={(data) => <h1>{data.message}</h1>} />;
}
```

Here, `DataProvider` doesn't know or care what gets rendered — it just hands its internal `data` to whatever function the consumer passed in via the `render` prop.

## Example 1: Mouse Tracker

A classic render props example — a component that tracks mouse position and lets any consumer decide how to display it.

```jsx
import { useState, useEffect } from "react";

function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, []);

  return render(position);
}

function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <p>
          Mouse position: {x}, {y}
        </p>
      )}
    />
  );
}
```

The same `MouseTracker` could be reused elsewhere with a completely different UI — a dot that follows the cursor, a tooltip, a debug overlay — just by passing a different `render` function.

## Example 2: Toggle Component

A reusable toggle that any component can use, without dictating what the toggle controls or how it looks.

```jsx
import { useState, useCallback } from "react";

function Toggle({ render }) {
  const [on, setOn] = useState(false);
  const toggle = useCallback(() => setOn((prev) => !prev), []);

  return render({ on, toggle });
}

function App() {
  return (
    <Toggle
      render={({ on, toggle }) => (
        <div>
          <button onClick={toggle}>{on ? "Hide" : "Show"} Details</button>
          {on && <p>Here are the extra details!</p>}
        </div>
      )}
    />
  );
}
```

## Example 3: Data Fetching with Render Props

Encapsulating loading/error/data state, letting the consumer control the presentation entirely.

```jsx
import { useState, useEffect } from "react";

function DataFetcher({ url, render }) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    let isCancelled = false;

    async function fetchData() {
      setState({ data: null, loading: true, error: null });
      try {
        const res = await fetch(url);
        const json = await res.json();
        if (!isCancelled) setState({ data: json, loading: false, error: null });
      } catch (err) {
        if (!isCancelled)
          setState({ data: null, loading: false, error: err.message });
      }
    }

    fetchData();
    return () => {
      isCancelled = true;
    };
  }, [url]);

  return render(state);
}

function UserProfile() {
  return (
    <DataFetcher
      url="/api/user/42"
      render={({ data, loading, error }) => {
        if (loading) return <p>Loading...</p>;
        if (error) return <p>Error: {error}</p>;
        return <h1>{data.name}</h1>;
      }}
    />
  );
}
```

## Using children as a Function

A common variation passes the function as `children` instead of a `render` prop — this reads a bit more naturally in JSX.

```jsx
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, []);

  return children(position);
}

function App() {
  return (
    <MouseTracker>
      {({ x, y }) => (
        <p>
          Position: {x}, {y}
        </p>
      )}
    </MouseTracker>
  );
}
```

Both approaches (`render` prop or `children` function) are functionally identical — it's purely a naming/style choice.

## Render Props vs Custom Hooks

Since Hooks were introduced, most render props use cases can be replaced with a custom Hook, usually with less nesting.

```jsx
// Render Props version
<MouseTracker
  render={({ x, y }) => (
    <p>
      {x}, {y}
    </p>
  )}
/>;

// Custom Hook version
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  useEffect(() => {
    const handleMouseMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, []);
  return position;
}

function App() {
  const { x, y } = useMousePosition();
  return (
    <p>
      {x}, {y}
    </p>
  );
}
```

| Feature           | Render Props                                                                         | Custom Hooks                                      |
| ----------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------- |
| Nesting           | Can lead to nested JSX ("callback hell") with multiple providers                     | Flat — just function calls                        |
| Reuse             | Reuses rendering + logic together                                                    | Reuses only logic; you control rendering directly |
| Readability       | Explicit about what data flows where                                                 | Generally cleaner in modern codebases             |
| When still useful | Libraries with a JSX-first API (e.g. some UI libraries, render-based animation libs) | Preferred default for new code                    |

## Render Props vs Higher-Order Components (HOCs)

```jsx
// HOC version
function withMouse(Component) {
  return function WrappedComponent(props) {
    const [position, setPosition] = useState({ x: 0, y: 0 });
    // ...tracking logic...
    return <Component {...props} mouse={position} />;
  };
}

// Render Props version
<MouseTracker render={(mouse) => <MyComponent mouse={mouse} />} />;
```

Render Props generally avoid the two biggest HOC annoyances:

- **Prop name collisions** — HOCs can silently overwrite props with the same name; render props make the data explicit at the call site.
- **Unclear component hierarchy** — wrapped components in HOCs are hard to trace in React DevTools; render props keep everything visible in the JSX tree.

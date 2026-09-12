# Lifecycle Hooks in React

React doesn't have a hook literally called `useLifecycle` — instead, **`useEffect`** (along with `useLayoutEffect`) replaces the lifecycle methods that class components used (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). 

## Table of Contents

- [The Three Phases of a Component](#the-three-phases-of-a-component)
- [Class Lifecycle Methods vs Hooks](#class-lifecycle-methods-vs-hooks)
- [useEffect Basics](#useeffect-basics)
- [Example 1: Mounting — componentDidMount Equivalent](#example-1-mounting--componentdidmount-equivalent)
- [Example 2: Updating — componentDidUpdate Equivalent](#example-2-updating--componentdidupdate-equivalent)
- [Example 3: Unmounting — componentWillUnmount Equivalent](#example-3-unmounting--componentwillunmount-equivalent)
- [Example 4: Combining All Three Phases](#example-4-combining-all-three-phases)
- [Example 5: useLayoutEffect for DOM Measurements](#example-5-uselayouteffect-for-dom-measurements)
- [Example 6: Error Boundaries (No Hook Equivalent)](#example-6-error-boundaries-no-hook-equivalent)
- [The Dependency Array Explained](#the-dependency-array-explained)


## The Three Phases of a Component

Every component goes through three broad phases:

1. **Mounting** — the component is created and inserted into the DOM for the first time.
2. **Updating** — the component re-renders because its props or state changed.
3. **Unmounting** — the component is removed from the DOM.

Class components had dedicated methods for each phase. Function components use `useEffect` (and its cleanup function) to handle all three.

## Class Lifecycle Methods vs Hooks

| Class Method | Hook Equivalent | Phase |
|---|---|---|
| `constructor` | `useState` (for initializing state) | Mounting |
| `componentDidMount` | `useEffect(() => {...}, [])` | Mounting |
| `componentDidUpdate` | `useEffect(() => {...}, [dep1, dep2])` | Updating |
| `componentWillUnmount` | Cleanup function returned from `useEffect` | Unmounting |
| `shouldComponentUpdate` | `React.memo`, `useMemo`, `useCallback` | Updating (optimization) |
| `getDerivedStateFromProps` | Calculating values directly during render | Updating |
| `componentDidCatch` / `getDerivedStateFromError` | No hook equivalent — still requires a class-based Error Boundary | Error handling |

## useEffect Basics

```jsx
import { useEffect } from "react";

useEffect(() => {
  // runs after render (mount + updates, depending on deps)

  return () => {
    // optional cleanup — runs before the next effect and on unmount
  };
}, [/* dependency array */]);
```

The dependency array controls **when** the effect runs — covered in detail below.

## Example 1: Mounting — componentDidMount Equivalent

**Class component:**

```jsx
class UserProfile extends React.Component {
  componentDidMount() {
    console.log("Component mounted — fetching user data");
    fetch(`/api/users/${this.props.userId}`)
      .then((res) => res.json())
      .then((data) => this.setState({ user: data }));
  }

  render() {
    return <div>{this.state.user?.name}</div>;
  }
}
```

**Function component with hooks:**

```jsx
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    console.log("Component mounted — fetching user data");
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => setUser(data));
  }, []); // empty array = runs once, only on mount

  return <div>{user?.name}</div>;
}
```

## Example 2: Updating — componentDidUpdate Equivalent

**Class component:**

```jsx
class SearchResults extends React.Component {
  componentDidUpdate(prevProps) {
    if (prevProps.query !== this.props.query) {
      console.log("Query changed — refetching");
      this.fetchResults(this.props.query);
    }
  }

  render() {
    return <ul>{/* results */}</ul>;
  }
}
```

**Function component with hooks:**

```jsx
import { useEffect } from "react";

function SearchResults({ query }) {
  useEffect(() => {
    console.log("Query changed — refetching");
    fetchResults(query);
  }, [query]); // runs on mount AND whenever `query` changes

  return <ul>{/* results */}</ul>;
}
```

## Example 3: Unmounting — componentWillUnmount Equivalent

**Class component:**

```jsx
class Timer extends React.Component {
  componentDidMount() {
    this.interval = setInterval(() => console.log("tick"), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return <p>Timer running...</p>;
  }
}
```

**Function component with hooks:**

```jsx
import { useEffect } from "react";

function Timer() {
  useEffect(() => {
    const interval = setInterval(() => console.log("tick"), 1000);

    return () => clearInterval(interval); // cleanup — runs on unmount
  }, []);

  return <p>Timer running...</p>;
}
```

## Example 4: Combining All Three Phases

A single `useEffect` can cover mount, update, and unmount logic together, since the same cleanup function runs both before re-running the effect (on update) and on unmount.

```jsx
import { useState, useEffect } from "react";

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    console.log(`Connecting to room ${roomId}`); // like componentDidMount / componentDidUpdate
    const connection = createConnection(roomId);

    connection.on("message", (msg) => {
      setMessages((prev) => [...prev, msg]);
    });

    connection.connect();

    return () => {
      console.log(`Disconnecting from room ${roomId}`); // like componentWillUnmount
      connection.disconnect();
    };
  }, [roomId]); // re-runs (cleanup + re-connect) whenever roomId changes

  return (
    <ul>
      {messages.map((m, i) => (
        <li key={i}>{m}</li>
      ))}
    </ul>
  );
}
```

## Example 5: useLayoutEffect for DOM Measurements

`useEffect` runs **after** the browser paints the screen. `useLayoutEffect` runs **synchronously before** the paint — useful when you need to measure or mutate the DOM before the user sees it (avoiding a visual flicker).

```jsx
import { useState, useLayoutEffect, useRef } from "react";

function Tooltip({ text }) {
  const ref = useRef(null);
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    const { width } = ref.current.getBoundingClientRect();
    setWidth(width); // measured and applied before the browser paints
  }, [text]);

  return (
    <div ref={ref} style={{ marginLeft: -width / 2 }}>
      {text}
    </div>
  );
}
```

Use `useLayoutEffect` sparingly — it blocks painting, so prefer `useEffect` unless you specifically need to prevent a visual flash.

## Example 6: Error Boundaries (No Hook Equivalent)

There is currently **no hook equivalent** for `componentDidCatch` or `getDerivedStateFromError`. Catching render errors for a subtree still requires a class component (though libraries like `react-error-boundary` wrap this for you).

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error("Caught by ErrorBoundary:", error, info);
  }

  render() {
    if (this.state.hasError) return <h1>Something went wrong.</h1>;
    return this.props.children;
  }
}
```

> Note: this is different from `useRouteError`/`errorElement` in React Router, which handles errors thrown during route loading/rendering specifically within the router's data APIs.

## The Dependency Array Explained

| Dependency Array | Behavior | Equivalent to |
|---|---|---|
| No array — `useEffect(fn)` | Runs after **every** render | No direct class equivalent (rarely what you want) |
| Empty array — `useEffect(fn, [])` | Runs **once**, after the initial render | `componentDidMount` |
| With values — `useEffect(fn, [a, b])` | Runs after the initial render, then again whenever `a` or `b` changes | `componentDidMount` + `componentDidUpdate` (conditionally) |
| Cleanup function returned | Runs before the effect re-runs, and on unmount | `componentWillUnmount` |


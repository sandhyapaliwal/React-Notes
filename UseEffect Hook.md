# React `useEffect` Hook

`useEffect` is a React Hook that lets you run side effects in function components, such as fetching data, setting up subscriptions, or working with timers and the DOM.

Side effects are operations that affect something outside the component, for example network calls, logging, or updating the browser title.
```js
import React, { useEffect } from "react";

useEffect(() => {
  // side effect code here
}, [dependencies]);
```

---

## Table of Contents

1. [Why useEffect?](#1-why-useeffect)
2. [Basic Syntax](#2-basic-syntax)
3. [Dependency Array Patterns](#3-dependency-array-patterns)
4. [Example: Updating Document Title](#4-example-updating-document-title)
5. [Example: Fetching Data on Mount](#5-example-fetching-data-on-mount)
6. [Cleanup Functions](#6-cleanup-functions)
7. [Common Mistakes and Best Practices](#7-common-mistakes-and-best-practices)
8. [useEffect vs Other Hooks](#8-useeffect-vs-other-hooks)

---

## 1. Why `useEffect`?

Before hooks, side effects in React were handled with class lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.[web:14]

`useEffect` brings this logic into function components and gives one unified way to manage side effects.

Typical use cases:

- Fetching data from an API.
- Subscribing / unsubscribing to events.
- Setting timers or intervals.
- Updating the document title or using browser APIs.

---

## 2. Basic Syntax

```js
useEffect(() => {
  // effect logic
  // runs after render
}, [dependency1, dependency2]);
```

- First argument: a function that contains the side effect.
- Second argument: an optional dependency array that controls when the effect runs.
If you omit the dependency array, the effect runs after every render.

---

## 3. Dependency Array Patterns

### 3.1 No Dependency Array

```js
useEffect(() => {
  console.log("Effect runs on every render");
});
```

- Runs after every render (initial and all updates).
- Use carefully for heavy effects because this can impact performance.

### 3.2 Empty Dependency Array `[]`

```js
useEffect(() => {
  console.log("Runs once on mount");
}, []);
```

- Runs only once after the initial render (on mount).
- Good for one‑time tasks like initial data fetch or setting up a subscription.

### 3.3 With Dependencies

```js
useEffect(() => {
  console.log("Runs when `count` changes");
}, [count]);
```

- Runs on the initial render and whenever any listed dependency changes.

> Best practice: Put only the values that the effect actually uses (state/props) into the dependency array.

---

## 4. Example: Updating Document Title

```js
import React, { useState, useEffect } from "react";

function CounterWithTitle() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Clicked ${count} times`;
  }, [count]); // runs whenever `count` changes

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(prev => prev + 1)}>
        Click me
      </button>
    </div>
  );
}

export default CounterWithTitle;
```

This effect updates the browser tab title each time the user clicks the button, because it depends on `count`.

---

## 5. Example: Fetching Data on Mount

```js
import React, { useState, useEffect } from "react";

function Todos() {
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchTodos() {
      try {
        const res = await fetch(
          "https://jsonplaceholder.typicode.com/todos?_limit=5"
        );
        const data = await res.json();
        setTodos(data);
      } catch (error) {
        console.error("Error fetching todos:", error);
      } finally {
        setLoading(false);
      }
    }

    fetchTodos();
  }, []); // runs once on mount

  if (loading) return <p>Loading...</p>;

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}

export default Todos;
```

Here the effect runs once when the component mounts, performs an async fetch, and stores the result in state.

---

## 6. Cleanup Functions

Some side effects need cleanup to avoid memory leaks or unwanted behavior, such as intervals, subscriptions, or event listeners.

You can return a cleanup function from the effect; React calls it before the component unmounts and before re‑running the effect.

### 6.1 Timer Example

```js
import React, { useState, useEffect } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    return () => {
      clearInterval(id); // cleanup on unmount
    };
  }, []);

  return <p>Timer: {seconds}s</p>;
}

export default Timer;
```

The cleanup function clears the interval and prevents the timer from continuing to run after the component is removed.

### 6.2 Event Listener Example

```js
import React, { useState, useEffect } from "react";

function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return <p>Window width: {width}px</p>;
}

export default WindowWidth;
```

Without the cleanup, multiple listeners could be added on re‑renders and cause bugs; the cleanup keeps the effect safe.

---

## 7. Common Mistakes and Best Practices

### 7.1 Forgetting the Dependency Array

- Forgetting `[]` or missing dependencies can cause effects to run on every render and repeat API calls.

### 7.2 Missing Real Dependencies

- Every state or prop used in the effect should be included in the dependency array.
- Use ESLint’s `react-hooks/exhaustive-deps` rule to catch missing dependencies.

### 7.3 Including Unnecessary Items

- Do not include stable functions like `setState` from `useState` because they never change.
- Use `useCallback` or move functions outside the component if you need stable function references.

### 7.4 Side Effects vs Rendering

- Only use `useEffect` for side effects; pure calculations should be done during render.
- Avoid state changes that cause infinite re‑render loops.

---

## 8. `useEffect` vs Other Hooks

- `useState`: manages local component state.
- `useEffect`: runs side effects after render.
- `useMemo`: memoizes expensive computations (no side effects).
- `useCallback`: memoizes functions used in dependencies or passed as props.




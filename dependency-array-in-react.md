# Dependency Array 

The **dependency array** is the second argument passed to several React hooks — `useEffect`, `useMemo`, `useCallback`, and `useImperativeHandle` — that tells React *when* to re-run the hook's logic. It's one of the most important (and most misunderstood) concepts in modern React.

## Table of Contents

- [What Is a Dependency Array?](#what-is-a-dependency-array)
- [The Three Forms](#the-three-forms)
- [useEffect and the Dependency Array](#useeffect-and-the-dependency-array)
- [useMemo and the Dependency Array](#usememo-and-the-dependency-array)
- [useCallback and the Dependency Array](#usecallback-and-the-dependency-array)
- [How React Compares Dependencies](#how-react-compares-dependencies)
- [Example 1: Missing Dependencies Bug](#example-1-missing-dependencies-bug)
- [Example 2: Object/Array Dependencies Causing Infinite Loops](#example-2-objectarray-dependencies-causing-infinite-loops)
- [Example 3: Functions as Dependencies](#example-3-functions-as-dependencies)
- [Example 4: Using the Functional Update Form to Avoid a Dependency](#example-4-using-the-functional-update-form-to-avoid-a-dependency)
- [The ESLint Plugin: react-hooks/exhaustive-deps](#the-eslint-plugin-react-hooksexhaustive-deps)


## What Is a Dependency Array?

It's a plain JavaScript array of values that a hook "watches." React compares each value in the array between renders — if any value has changed, the hook's logic re-runs; if nothing changed, React skips it.

```jsx
useEffect(() => {
  console.log("This runs when `count` changes");
}, [count]); // ← dependency array
```

## The Three Forms

There are three ways to write (or omit) the dependency array, and each means something very different:

```jsx
// 1. No array at all — runs after EVERY render
useEffect(() => {
  console.log("Runs every render");
});

// 2. Empty array — runs ONCE, after the initial render only
useEffect(() => {
  console.log("Runs once, on mount");
}, []);

// 3. Array with values — runs on mount, and again whenever any listed value changes
useEffect(() => {
  console.log("Runs on mount and whenever `userId` changes");
}, [userId]);
```

## useEffect and the Dependency Array

`useEffect` is the most common place developers use a dependency array — it controls when a side effect (data fetching, subscriptions, DOM manipulation, timers) re-runs.

```jsx
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        if (!isCancelled) setUser(data);
      });

    return () => {
      isCancelled = true; // cleanup
    };
  }, [userId]); // re-fetch whenever userId changes

  return <div>{user ? user.name : "Loading..."}</div>;
}
```

## useMemo and the Dependency Array

`useMemo` re-computes an expensive value only when one of its dependencies changes, instead of on every render.

```jsx
import { useMemo } from "react";

function ProductList({ products, searchTerm }) {
  const filteredProducts = useMemo(() => {
    console.log("Filtering...");
    return products.filter((p) => p.name.toLowerCase().includes(searchTerm.toLowerCase()));
  }, [products, searchTerm]); // only re-filter when products or searchTerm change

  return (
    <ul>
      {filteredProducts.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

## useCallback and the Dependency Array

`useCallback` memoizes a function itself (not its return value), so the same function reference is reused across renders unless a dependency changes. This matters when passing callbacks to memoized child components (`React.memo`).

```jsx
import { useCallback, useState } from "react";

function TodoList() {
  const [todos, setTodos] = useState([]);

  const addTodo = useCallback((text) => {
    setTodos((prev) => [...prev, { id: Date.now(), text }]);
  }, []); // no dependencies — function never needs to change

  return <TodoForm onAdd={addTodo} />;
}
```

## How React Compares Dependencies

React compares each value in the array to its previous value using **`Object.is`** (essentially the same as `===`, with a couple of edge-case differences for `NaN` and `-0`). This is a **shallow comparison**:

- Primitives (numbers, strings, booleans) compare by value: `5 === 5` → true, no re-run.
- Objects, arrays, and functions compare by **reference**: two objects with identical contents but different references are considered *different*, triggering a re-run.

```jsx
const a = { count: 1 };
const b = { count: 1 };
console.log(a === b); // false — different references, even though contents match
```

This is the root cause of most dependency-array bugs (see examples below).

## Example 1: Missing Dependencies Bug

```jsx
// ❌ Bug: `count` is used inside the effect but missing from the array
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // always logs the initial value (0) due to stale closure
    }, 1000);
    return () => clearInterval(timer);
  }, []); // missing `count`

  return <button onClick={() => setCount(count + 1)}>Increment</button>;
}
```

```jsx
// ✅ Fixed: include `count`, or use the functional update form (see Example 4)
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(timer);
}, [count]);
```

## Example 2: Object/Array Dependencies Causing Infinite Loops

```jsx
// ❌ Bug: a new object is created on every render, so the effect re-runs every time,
// which updates state, which triggers another render, which creates a new object...
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const filters = { category: "electronics" }; // new object reference every render

  useEffect(() => {
    fetchResults(query, filters).then(setResults);
  }, [query, filters]); // `filters` is never equal to its previous self

  return <ResultsList results={results} />;
}
```

```jsx
// ✅ Fixed: move the object outside the component, or memoize it, or depend on primitives instead
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    fetchResults(query, { category: "electronics" }).then(setResults);
  }, [query]); // only depend on the primitive that actually changes

  return <ResultsList results={results} />;
}
```

## Example 3: Functions as Dependencies

```jsx
// ❌ Bug: `onSearch` is a new function every render (if defined inline in the parent),
// causing this effect to re-run constantly
function SearchBox({ onSearch }) {
  useEffect(() => {
    const timer = setTimeout(() => onSearch(), 500);
    return () => clearTimeout(timer);
  }, [onSearch]);
}
```

```jsx
// ✅ Fixed: wrap onSearch in useCallback where it's defined (in the parent)
function Parent() {
  const handleSearch = useCallback(() => {
    // search logic
  }, []); // stable reference across renders

  return <SearchBox onSearch={handleSearch} />;
}
```

## Example 4: Using the Functional Update Form to Avoid a Dependency

When updating state based on its previous value, you can often avoid adding that state as a dependency entirely by using the functional updater form.

```jsx
// Instead of depending on `count`...
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1); // needs `count` in deps
  }, 1000);
  return () => clearInterval(timer);
}, [count]);

// ...use the functional form and drop the dependency:
useEffect(() => {
  const timer = setInterval(() => {
    setCount((prev) => prev + 1); // doesn't need `count` in deps
  }, 1000);
  return () => clearInterval(timer);
}, []); // effect only runs once, on mount
```

## The ESLint Plugin: react-hooks/exhaustive-deps

The official `eslint-plugin-react-hooks` package includes a rule, `exhaustive-deps`, that warns when your dependency array is missing a value used inside the hook. It's strongly recommended to keep this rule enabled rather than disable it or silence warnings with comments.

```bash
npm install eslint-plugin-react-hooks --save-dev
```

```json
// .eslintrc
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}


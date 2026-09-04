# Custom Hooks in React

Custom Hooks are JavaScript functions that let you extract and reuse stateful logic across components without changing the component hierarchy. They follow React's Hook rules and always start with the word `use`.

## Table of Contents

- [Why Custom Hooks?](#why-custom-hooks)
- [Rules of Hooks](#rules-of-hooks)
- [Anatomy of a Custom Hook](#anatomy-of-a-custom-hook)
- [Example 1: useToggle](#example-1-usetoggle)
- [Example 2: useFetch](#example-2-usefetch)
- [Example 3: useLocalStorage](#example-3-uselocalstorage)
- [Example 4: useDebounce](#example-4-usedebounce)

## Why Custom Hooks?

Before Hooks, sharing stateful logic between components required patterns like **Higher-Order Components (HOCs)** or **Render Props**, both of which often led to deeply nested component trees ("wrapper hell").

Custom Hooks solve this by letting you:

- **Reuse logic** across multiple components without duplicating code.
- **Keep components clean** by moving complex logic out of the render function.
- **Compose behavior** by combining multiple built-in Hooks (`useState`, `useEffect`, `useRef`, etc.) into a single reusable unit.
- **Test logic independently** from the UI that consumes it.

## Rules of Hooks

Custom Hooks must follow the same two rules as built-in Hooks:

1. **Only call Hooks at the top level.** Don't call Hooks inside loops, conditions, or nested functions.
2. **Only call Hooks from React functions.** Call them from React function components or from other custom Hooks — not from regular JavaScript functions.

A function only counts as a Hook if its name starts with `use`, which allows React (and linters) to check these rules automatically.

## Anatomy of a Custom Hook

A custom Hook is simply a function that:

1. Starts with `use`.
2. Calls one or more built-in Hooks internally.
3. Returns whatever data or functions the consuming component needs (a value, an array, or an object).

```jsx
function useSomething(initialValue) {
  const [state, setState] = React.useState(initialValue);

  React.useEffect(() => {
    // side effect logic
  }, [state]);

  return [state, setState];
}
```

## Example 1: useToggle

A simple Hook to toggle a boolean value — useful for modals, dropdowns, and switches.

```jsx
import { useState, useCallback } from "react";

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue((prev) => !prev), []);

  return [value, toggle];
}

export default useToggle;
```

**Usage:**

```jsx
function Modal() {
  const [isOpen, toggleOpen] = useToggle(false);

  return (
    <div>
      <button onClick={toggleOpen}>{isOpen ? "Close" : "Open"} Modal</button>
      {isOpen && <div className="modal">Modal Content</div>}
    </div>
  );
}
```

## Example 2: useFetch

A Hook that encapsulates data-fetching logic, including loading and error states.

```jsx
import { useState, useEffect } from "react";

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    async function fetchData() {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
        const result = await response.json();
        if (!isCancelled) setData(result);
      } catch (err) {
        if (!isCancelled) setError(err.message);
      } finally {
        if (!isCancelled) setLoading(false);
      }
    }

    fetchData();

    return () => {
      isCancelled = true; // avoid state updates after unmount
    };
  }, [url]);

  return { data, loading, error };
}

export default useFetch;
```

**Usage:**

```jsx
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return <h1>{data.name}</h1>;
}
```

## Example 3: useLocalStorage

Syncs component state with `localStorage`, persisting values across page reloads.

```jsx
import { useState, useEffect } from "react";

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const stored = window.localStorage.getItem(key);
      return stored !== null ? JSON.parse(stored) : initialValue;
    } catch (error) {
      console.error("Error reading localStorage key:", key, error);
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error("Error writing localStorage key:", key, error);
    }
  }, [key, value]);

  return [value, setValue];
}

export default useLocalStorage;
```

**Usage:**

```jsx
function Settings() {
  const [theme, setTheme] = useLocalStorage("theme", "light");

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      Current theme: {theme}
    </button>
  );
}
```

## Example 4: useDebounce

Delays updating a value until the user has stopped changing it for a specified time — commonly used for search inputs.

```jsx
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

export default useDebounce;
```

**Usage:**

```jsx
function SearchBox() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 400);

  useEffect(() => {
    if (debouncedQuery) {
      // trigger API call with debouncedQuery
      console.log("Searching for:", debouncedQuery);
    }
  }, [debouncedQuery]);

  return (
    <input
      type="text"
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```


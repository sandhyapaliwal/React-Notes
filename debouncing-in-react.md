# Debouncing in React

Debouncing is a technique that delays executing a function until a certain amount of time has passed since the **last time it was invoked**. In React, it's most commonly used to prevent expensive operations (API calls, filtering, calculations) from firing on every keystroke, resize, or scroll event.

## Table of Contents

- [Why Debouncing?](#why-debouncing)
- [Debouncing vs Throttling](#debouncing-vs-throttling)
- [Approach 1: Plain Debounce with useEffect](#approach-1-plain-debounce-with-useeffect)
- [Approach 2: Custom useDebounce Hook (Debounced Value)](#approach-2-custom-usedebounce-hook-debounced-value)
- [Approach 3: Custom useDebouncedCallback Hook (Debounced Function)](#approach-3-custom-usedebouncedcallback-hook-debounced-function)
- [Example: Debounced Search Input with API Call](#example-debounced-search-input-with-api-call)
- [Using Lodash's debounce](#using-lodashs-debounce)

## Why Debouncing?

Without debouncing, an event like typing in a search box fires a handler on **every single keystroke**. If that handler triggers an API call, you end up:

- Sending far more requests than necessary
- Wasting bandwidth and backend resources
- Risking race conditions where older responses arrive after newer ones
- Making the UI feel janky if the handler does heavy computation

Debouncing fixes this by waiting until the user **pauses** (e.g., stops typing for 400ms) before running the function — so only the final value triggers the action.

## Debouncing vs Throttling

These two are often confused but solve different problems:

|          | Debouncing                                              | Throttling                                                              |
| -------- | ------------------------------------------------------- | ----------------------------------------------------------------------- |
| Behavior | Waits until activity **stops** for X ms, then runs once | Runs at most once every X ms, **regardless** of activity                |
| Use case | Search input, form validation, resize-end               | Scroll position tracking, button-click spam prevention, infinite scroll |
| Example  | Wait until user stops typing, then search               | Log scroll position every 200ms while scrolling                         |

## Approach 1: Plain Debounce with useEffect

The simplest way to debounce in React — delay a side effect using `setTimeout` inside `useEffect`, and clean up the timer on every re-run.

```jsx
import { useState, useEffect } from "react";

function SearchBox() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);

  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }

    const timer = setTimeout(() => {
      fetch(`/api/search?q=${query}`)
        .then((res) => res.json())
        .then((data) => setResults(data));
    }, 500);

    return () => clearTimeout(timer); // cancels the previous timer if query changes again
  }, [query]);

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**How it works:** every keystroke updates `query` and schedules a new timer, but the cleanup function cancels the previous timer first — so only the last keystroke's timer actually completes.

## Approach 2: Custom useDebounce Hook (Debounced Value)

A reusable hook that returns a debounced **value** — cleaner when you want to debounce the dependency itself rather than writing `setTimeout` logic in every component.

```jsx
// useDebounce.js
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
import { useState, useEffect } from "react";
import useDebounce from "./useDebounce";

function SearchBox() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (!debouncedQuery) return;

    fetch(`/api/search?q=${debouncedQuery}`)
      .then((res) => res.json())
      .then((data) => console.log(data));
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

This separates concerns neatly: `query` updates instantly (so the input stays responsive), while `debouncedQuery` only updates after the user pauses.

## Approach 3: Custom useDebouncedCallback Hook (Debounced Function)

Sometimes you want to debounce a **function** itself (not just a value) — for example, a save-on-change handler.

```jsx
// useDebouncedCallback.js
import { useRef, useCallback, useEffect } from "react";

function useDebouncedCallback(callback, delay = 500) {
  const timerRef = useRef(null);

  const debouncedFn = useCallback(
    (...args) => {
      if (timerRef.current) clearTimeout(timerRef.current);
      timerRef.current = setTimeout(() => callback(...args), delay);
    },
    [callback, delay],
  );

  useEffect(() => {
    return () => {
      if (timerRef.current) clearTimeout(timerRef.current);
    };
  }, []);

  return debouncedFn;
}

export default useDebouncedCallback;
```

**Usage:**

```jsx
import { useState } from "react";
import useDebouncedCallback from "./useDebouncedCallback";

function NoteEditor() {
  const [text, setText] = useState("");

  const saveNote = (value) => {
    console.log("Saving to server:", value);
    // fetch("/api/notes", { method: "POST", body: value })
  };

  const debouncedSave = useDebouncedCallback(saveNote, 800);

  const handleChange = (e) => {
    setText(e.target.value);
    debouncedSave(e.target.value); // only fires 800ms after typing stops
  };

  return (
    <textarea
      value={text}
      onChange={handleChange}
      placeholder="Start typing..."
    />
  );
}
```

## Example: Debounced Search Input with API Call

Putting it together — a realistic debounced search with loading state and stale-response protection.

```jsx
import { useState, useEffect } from "react";
import useDebounce from "./useDebounce";

function ProductSearch() {
  const [query, setQuery] = useState("");
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState([]);
  const debouncedQuery = useDebounce(query, 400);

  useEffect(() => {
    if (!debouncedQuery) {
      setResults([]);
      return;
    }

    let isCancelled = false;
    setLoading(true);

    fetch(`/api/products?search=${debouncedQuery}`)
      .then((res) => res.json())
      .then((data) => {
        if (!isCancelled) {
          setResults(data);
          setLoading(false);
        }
      });

    return () => {
      isCancelled = true; // ignores stale responses if debouncedQuery changes again quickly
    };
  }, [debouncedQuery]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      {loading && <p>Loading...</p>}
      <ul>
        {results.map((p) => (
          <li key={p.id}>{p.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Using Lodash's debounce

For production apps already using Lodash, its battle-tested `debounce` utility is a solid alternative to writing your own.

```bash
npm install lodash.debounce
```

```jsx
import { useMemo, useState, useEffect } from "react";
import debounce from "lodash.debounce";

function SearchBox() {
  const [query, setQuery] = useState("");

  const debouncedSearch = useMemo(
    () =>
      debounce((value) => {
        fetch(`/api/search?q=${value}`)
          .then((res) => res.json())
          .then((data) => console.log(data));
      }, 500),
    [],
  );

  useEffect(() => {
    return () => debouncedSearch.cancel(); // cleanup on unmount
  }, [debouncedSearch]);

  const handleChange = (e) => {
    setQuery(e.target.value);
    debouncedSearch(e.target.value);
  };

  return (
    <input value={query} onChange={handleChange} placeholder="Search..." />
  );
}
```

`useMemo` (not `useCallback`) is used here so the debounced function instance is only created once and persists across renders, and `.cancel()` cleans up any pending call on unmount.

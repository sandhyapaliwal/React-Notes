# Sibling Communication in React

**Sibling communication** refers to how two (or more) components that share the same parent — but have no direct parent-child relationship with each other — exchange data or trigger actions in one another. Since React data flows one-way (parent → child via props), siblings can't talk to each other directly. 

## Table of Contents

- [The Problem](#the-problem)
- [Pattern 1: Lifting State Up](#pattern-1-lifting-state-up)
- [Pattern 2: Context API](#pattern-2-context-api)
- [Pattern 3: Custom Event Bus (Pub/Sub)](#pattern-3-custom-event-bus-pubsub)
- [Pattern 4: State Management Libraries](#pattern-4-state-management-libraries)
- [Pattern 5: URL / Query Params as Shared State](#pattern-5-url--query-params-as-shared-state)

## The Problem

Consider two sibling components, `SearchBox` and `ResultsList`, both rendered inside `App`:

```jsx
function App() {
  return (
    <div>
      <SearchBox />   {/* user types a query here */}
      <ResultsList /> {/* needs to know the query to show results */}
    </div>
  );
}
```

`SearchBox` and `ResultsList` are siblings — neither can pass props directly to the other, since props only flow downward from parent to child. `SearchBox` has no way to "hand" its input value to `ResultsList` on its own.

## Pattern 1: Lifting State Up

The most common and idiomatic solution: move the shared state up to the **nearest common ancestor** (here, `App`), then pass it down to both siblings as props — data down, callbacks up.

```jsx
import { useState } from "react";

function App() {
  const [query, setQuery] = useState("");

  return (
    <div>
      <SearchBox query={query} onQueryChange={setQuery} />
      <ResultsList query={query} />
    </div>
  );
}

function SearchBox({ query, onQueryChange }) {
  return (
    <input
      value={query}
      onChange={(e) => onQueryChange(e.target.value)}
      placeholder="Search..."
    />
  );
}

function ResultsList({ query }) {
  const filtered = ["Apple", "Banana", "Cherry"].filter((item) =>
    item.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <ul>
      {filtered.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}
```

`SearchBox` updates `query` via the `onQueryChange` callback; `App` re-renders and passes the new `query` down to `ResultsList`. This works well for shallow trees where the common ancestor is close by.

## Pattern 2: Context API

When siblings are deeply nested under a common ancestor (so lifting state up would mean passing props through many unrelated layers — "prop drilling"), **Context** lets you skip the middle layers entirely.

```jsx
// QueryContext.js
import { createContext, useState, useContext } from "react";

const QueryContext = createContext(null);

export function QueryProvider({ children }) {
  const [query, setQuery] = useState("");
  return (
    <QueryContext.Provider value={{ query, setQuery }}>
      {children}
    </QueryContext.Provider>
  );
}

export function useQuery() {
  return useContext(QueryContext);
}
```

```jsx
// App.jsx
import { QueryProvider } from "./QueryContext";

function App() {
  return (
    <QueryProvider>
      <Layout>
        <SearchBox />
        <ResultsList />
      </Layout>
    </QueryProvider>
  );
}
```

```jsx
// SearchBox.jsx
import { useQuery } from "./QueryContext";

function SearchBox() {
  const { query, setQuery } = useQuery();
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

```jsx
// ResultsList.jsx
import { useQuery } from "./QueryContext";

function ResultsList() {
  const { query } = useQuery();
  const filtered = ["Apple", "Banana", "Cherry"].filter((item) =>
    item.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <ul>
      {filtered.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}
```

Both `SearchBox` and `ResultsList` read/write the same context value, regardless of how deeply nested `Layout` is.

## Pattern 3: Custom Event Bus (Pub/Sub)

Less common in modern React, but occasionally useful for decoupled, cross-cutting communication (e.g., triggering a toast notification from anywhere). A simple pub/sub pattern:

```jsx
// eventBus.js
const listeners = {};

export const eventBus = {
  on(event, callback) {
    (listeners[event] ||= []).push(callback);
  },
  off(event, callback) {
    listeners[event] = (listeners[event] || []).filter((cb) => cb !== callback);
  },
  emit(event, data) {
    (listeners[event] || []).forEach((cb) => cb(data));
  },
};
```

```jsx
// Notifier.jsx
import { useEffect, useState } from "react";
import { eventBus } from "./eventBus";

function Notifier() {
  const [message, setMessage] = useState(null);

  useEffect(() => {
    const handler = (msg) => setMessage(msg);
    eventBus.on("notify", handler);
    return () => eventBus.off("notify", handler);
  }, []);

  return message ? <div className="toast">{message}</div> : null;
}
```

```jsx
// SaveButton.jsx
import { eventBus } from "./eventBus";

function SaveButton() {
  const handleSave = () => {
    // ...save logic
    eventBus.emit("notify", "Saved successfully!");
  };

  return <button onClick={handleSave}>Save</button>;
}
```

This decouples `SaveButton` and `Notifier` completely — they don't need to share an ancestor at all. Use sparingly, since it bypasses React's data flow and can make state harder to trace.

## Pattern 4: State Management Libraries

For larger apps with many siblings needing shared state across the whole tree, dedicated state libraries scale better than Context (which can cause broad re-renders if not carefully split).

```jsx
// store.js (Zustand example)
import { create } from "zustand";

export const useQueryStore = create((set) => ({
  query: "",
  setQuery: (query) => set({ query }),
}));
```

```jsx
// SearchBox.jsx
import { useQueryStore } from "./store";

function SearchBox() {
  const { query, setQuery } = useQueryStore();
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

```jsx
// ResultsList.jsx
import { useQueryStore } from "./store";

function ResultsList() {
  const query = useQueryStore((state) => state.query);
  // ...filter and render
}
```

Other common options: **Redux Toolkit**, **Jotai**, **Recoil**.

## Pattern 5: URL / Query Params as Shared State

When the shared value should also be shareable/bookmarkable (like a search term or filter), storing it in the URL via `useSearchParams` (React Router) lets siblings read/write the same source of truth without any extra state management.

```jsx
import { useSearchParams } from "react-router-dom";

function SearchBox() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get("q") || "";

  return (
    <input
      value={query}
      onChange={(e) => setSearchParams({ q: e.target.value })}
    />
  );
}

function ResultsList() {
  const [searchParams] = useSearchParams();
  const query = searchParams.get("q") || "";
  // ...filter and render
}
```


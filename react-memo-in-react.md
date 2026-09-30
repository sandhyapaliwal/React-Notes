# React.memo in React

`React.memo` is a **higher-order component (HOC)** that memoizes a functional component, preventing it from re-rendering when its props haven't changed. It's React's primary tool for optimizing render performance at the component level.

## Table of Contents

- [What is React.memo?](#what-is-reactmemo)
- [Basic Usage](#basic-usage)
- [How It Works](#how-it-works)
- [Example 1: Preventing Unnecessary Re-renders](#example-1-preventing-unnecessary-re-renders)
- [Example 2: Custom Comparison Function](#example-2-custom-comparison-function)
- [Example 3: React.memo with useCallback](#example-3-reactmemo-with-usecallback)
- [Example 4: React.memo with useMemo for Object/Array Props](#example-4-reactmemo-with-usememo-for-objectarray-props)
- [Example 5: React.memo with children](#example-5-reactmemo-with-children)
- [React.memo vs useMemo vs useCallback](#reactmemo-vs-usememo-vs-usecallback)

## What is React.memo?

By default, when a parent component re-renders, **all of its child components re-render too**, even if their props haven't changed. `React.memo` wraps a component and tells React: "skip re-rendering this component if its props are the same as last time."

It performs a **shallow comparison** of props by default — comparing each prop with `Object.is()`.

## Basic Usage

```jsx
import { memo } from "react";

function Greeting({ name }) {
  console.log("Greeting rendered");
  return <h1>Hello, {name}!</h1>;
}

export default memo(Greeting);
```

Now `Greeting` only re-renders when the `name` prop actually changes — not every time its parent re-renders.

## How It Works

```
Parent re-renders
   ↓
Child wrapped in memo?
   ↓
   ├── Props are the same (shallow equal) → Skip re-render, reuse last output
   └── Props changed → Re-render normally
```

`React.memo` does **not** prevent a component from re-rendering due to its _own_ state or context changes — it only affects re-renders triggered by the _parent_ passing the same props again.

## Example 1: Preventing Unnecessary Re-renders

```jsx
import { useState, memo } from "react";

const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log("ExpensiveList rendered");
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});

function App() {
  const [count, setCount] = useState(0);
  const [items] = useState([
    { id: 1, name: "Apple" },
    { id: 2, name: "Banana" },
  ]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      {/* ExpensiveList does NOT re-render when count changes,
          because `items` reference stays the same */}
      <ExpensiveList items={items} />
    </div>
  );
}
```

Without `memo`, clicking the button would re-render `ExpensiveList` every time, even though `items` never changes.

## Example 2: Custom Comparison Function

By default, `memo` does a shallow comparison. You can pass a second argument — a custom comparison function — for finer control.

```jsx
import { memo } from "react";

function UserCard({ user }) {
  return (
    <div>
      <p>{user.name}</p>
      <p>{user.email}</p>
    </div>
  );
}

function arePropsEqual(prevProps, nextProps) {
  return (
    prevProps.user.id === nextProps.user.id &&
    prevProps.user.name === nextProps.user.name
  );
  // ignores other fields like `email` changing, for example
}

export default memo(UserCard, arePropsEqual);
```

> The comparison function should return `true` if the props are equal (skip re-render) and `false` if they're different (re-render). This is the opposite of a typical `shouldComponentUpdate`, which returns `true` to re-render.

## Example 3: React.memo with useCallback

A very common pitfall: passing a new function as a prop on every render defeats `memo`, because a new function reference is never equal to the last one. `useCallback` fixes this.

```jsx
import { useState, useCallback, memo } from "react";

const Button = memo(function Button({ onClick, label }) {
  console.log(`${label} rendered`);
  return <button onClick={onClick}>{label}</button>;
});

function App() {
  const [count, setCount] = useState(0);

  // Without useCallback, this function is recreated on every render,
  // so Button would re-render every time despite being memoized.
  const handleClick = useCallback(() => {
    console.log("Clicked!");
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Button onClick={handleClick} label="Static Button" />
    </div>
  );
}
```

## Example 4: React.memo with useMemo for Object/Array Props

The same reference issue applies to objects and arrays. Use `useMemo` to keep the reference stable across renders.

```jsx
import { useState, useMemo, memo } from "react";

const Chart = memo(function Chart({ config }) {
  console.log("Chart rendered");
  return <div>Chart with theme: {config.theme}</div>;
});

function Dashboard() {
  const [count, setCount] = useState(0);

  // Without useMemo, a new object is created every render,
  // breaking memo's shallow comparison.
  const config = useMemo(() => ({ theme: "dark" }), []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Chart config={config} />
    </div>
  );
}
```

## Example 5: React.memo with children

Passing `children` as JSX also creates a new reference on every parent render, which can still trigger re-renders even with `memo` — unless the JSX itself is hoisted or passed down from a higher, non-re-rendering ancestor.

```jsx
const Card = memo(function Card({ children }) {
  console.log("Card rendered");
  return <div className="card">{children}</div>;
});

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      {/* This children JSX is recreated each time App renders,
          so Card still re-renders despite being memoized */}
      <Card>
        <p>Static content</p>
      </Card>
    </div>
  );
}
```

To truly avoid this, lift the `<Card>` element itself out of the re-rendering component, or pass it down as a prop from a parent that doesn't re-render on `count` changes.

## React.memo vs useMemo vs useCallback

| Feature | `React.memo`                                         | `useMemo`                               | `useCallback`                         |
| ------- | ---------------------------------------------------- | --------------------------------------- | ------------------------------------- |
| Type    | Higher-order component                               | Hook                                    | Hook                                  |
| Purpose | Skip re-rendering a component if props are unchanged | Memoize a computed **value**            | Memoize a **function** reference      |
| Used on | Components                                           | Any expensive calculation               | Functions passed as props/callbacks   |
| Example | `memo(MyComponent)`                                  | `useMemo(() => computeX(a, b), [a, b])` | `useCallback(() => doThing(), [dep])` |

They're often used **together**: `useCallback`/`useMemo` keep prop references stable, and `React.memo` uses that stability to actually skip re-renders.

## When NOT to Use React.memo

`React.memo` isn't free — it adds a comparison step on every render. Skip it when:

- The component is **cheap to render** (simple JSX, no heavy computation) — the comparison overhead may outweigh the savings.
- Props **change on almost every render anyway** — there's nothing to memoize against.
- The component rarely re-renders in the first place (e.g., it's already at a leaf with stable props from context).

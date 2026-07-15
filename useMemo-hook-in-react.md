# useMemo Hook

`useMemo` is a React Hook that lets you cache (memoize) the result of an expensive calculation between renders, so it's only recomputed when its dependencies change.

## Table of Contents

- [Why useMemo?](#why-usememo)
- [Basic Syntax](#basic-syntax)
- [Step-by-Step Example](#step-by-step-example)
- [Memoizing Objects to Prevent Re-renders](#memoizing-objects-to-prevent-re-renders)
- [useMemo vs useCallback](#usememo-vs-usecallback)
- [Common Pitfalls](#common-pitfalls)
- [When to Use / Avoid](#when-to-use--avoid)
---

## Why useMemo?

Every time a component re-renders, all the code inside its function body runs again — including expensive calculations, even if their inputs haven't changed.

```jsx
function ProductList({ products, filter }) {
  // This filtering runs on every render, even if `products` and `filter` haven't changed
  const filteredProducts = products.filter((p) => p.category === filter);

  return (
    <ul>
      {filteredProducts.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

If `ProductList` re-renders for an unrelated reason (like a parent state update), that filtering logic reruns unnecessarily. `useMemo` lets you skip recalculating a value unless its dependencies actually change.

## Basic Syntax

```jsx
import { useMemo } from 'react';

const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- The first argument is a function that returns the value you want to memoize.
- The second argument is a dependency array — the function only re-runs when one of these values changes.

## Step-by-Step Example

```jsx
import { useMemo, useState } from 'react';

function ProductList({ products }) {
  const [filter, setFilter] = useState('all');
  const [darkMode, setDarkMode] = useState(false);

  const filteredProducts = useMemo(() => {
    console.log('Filtering products...');
    return filter === 'all'
      ? products
      : products.filter((p) => p.category === filter);
  }, [products, filter]);

  return (
    <div className={darkMode ? 'dark' : 'light'}>
      <button onClick={() => setDarkMode(!darkMode)}>Toggle Dark Mode</button>
      <select value={filter} onChange={(e) => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
        <option value="books">Books</option>
      </select>
      <ul>
        {filteredProducts.map((p) => (
          <li key={p.id}>{p.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

Toggling `darkMode` causes a re-render, but because `products` and `filter` haven't changed, `useMemo` skips re-running the filter logic and returns the cached array instead — you won't see `"Filtering products..."` logged again.

## Memoizing Objects to Prevent Re-renders

`useMemo` is also useful for keeping object or array references stable across renders, which matters when passing them as props to memoized child components (`React.memo`).

```jsx
import { useMemo, useState } from 'react';

const UserCard = React.memo(function UserCard({ user }) {
  console.log('UserCard rendered');
  return <p>{user.name}</p>;
});

function App() {
  const [count, setCount] = useState(0);

  // Without useMemo, this object is a *new reference* every render,
  // causing UserCard to re-render even though the data hasn't changed.
  const user = useMemo(() => ({ name: 'Alex' }), []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <UserCard user={user} />
    </div>
  );
}
```

Without `useMemo`, `{ name: 'Alex' }` would be a brand-new object on every render, defeating `React.memo`'s shallow comparison and causing `UserCard` to re-render unnecessarily.

## useMemo vs useCallback

`useMemo` and `useCallback` are closely related — `useCallback(fn, deps)` is essentially shorthand for `useMemo(() => fn, deps)`.

| | `useMemo` | `useCallback` |
|---|---|---|
| Memoizes | A **value** (result of a computation) | A **function** itself |
| Use case | Expensive calculations, stable object/array references | Stable function references (e.g., passed to child components or dependency arrays) |
| Example | `useMemo(() => a + b, [a, b])` | `useCallback(() => doSomething(a, b), [a, b])` |

```jsx
// These are conceptually equivalent:
const memoizedFn = useCallback(() => doSomething(a), [a]);
const memoizedFn2 = useMemo(() => () => doSomething(a), [a]);
```

## Common Pitfalls

1. **Using it for everything** — `useMemo` itself has a cost (it still runs on every render to check dependencies). Wrapping cheap calculations in `useMemo` can add overhead without meaningful benefit.
2. **Missing or incorrect dependencies** — If you omit a value used inside the memoized function from the dependency array, you'll get stale results. Always include every value from the outer scope that the function reads.
3. **Treating useMemo as a guarantee** — React may choose to discard cached values in some cases (e.g., to free memory), so `useMemo` should be treated as a performance optimization, not a way to guarantee a calculation runs exactly once.
4. **Memoizing values that change every render anyway** — If a dependency changes on every render, memoizing provides no benefit and just adds complexity.

## When to Use / Avoid

**Good use cases:**
- Expensive computations (sorting, filtering, or transforming large lists)
- Preventing unnecessary re-renders of memoized child components by stabilizing object/array props
- Derived values used in multiple places within a component

**Avoid when:**
- The calculation is cheap (simple arithmetic, string concatenation) — the memoization overhead isn't worth it
- You're using it just to "feel like you're optimizing" without measuring an actual performance issue


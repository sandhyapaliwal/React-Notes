# useRef Hook

`useRef` is a React Hook that lets you create a mutable value that persists across renders **without** causing a re-render when it changes. It's most commonly used to directly access DOM elements, but it's also useful for storing any mutable value that shouldn't trigger a re-render.

## Table of Contents

- [Why useRef?](#why-useref)
- [Basic Syntax](#basic-syntax)
- [Use Case 1: Accessing DOM Elements](#use-case-1-accessing-dom-elements)
- [Use Case 2: Storing Mutable Values](#use-case-2-storing-mutable-values)
- [Use Case 3: Tracking Previous Values](#use-case-3-tracking-previous-values)
- [useRef vs useState](#useref-vs-usestate)
- [Common Pitfalls](#common-pitfalls)
- [When to Use / Avoid](#when-to-use--avoid)


---

## Why useRef?

React re-renders a component whenever its state changes. But sometimes you need to:

- Directly access or manipulate a DOM node (focus an input, scroll to an element, measure size)
- Store a value that changes over time but shouldn't cause the UI to re-render (like a timer ID or a render count)
- Keep track of a previous value between renders

`useState` isn't ideal for these cases because updating state always re-renders the component. `useRef` gives you a "box" that holds a value and survives across renders, but changing `.current` doesn't trigger a re-render.

## Basic Syntax

```jsx
import { useRef } from 'react';

function MyComponent() {
  const myRef = useRef(initialValue);

  // Access or update the value
  console.log(myRef.current);
  myRef.current = newValue;

  return <div>...</div>;
}
```

`useRef` returns a plain object: `{ current: initialValue }`. This object is the same object on every render — only `.current` changes.

## Use Case 1: Accessing DOM Elements

The most common use of `useRef` is attaching it to a DOM element via the `ref` attribute, giving you direct access to that node.

```jsx
import { useRef } from 'react';

function TextInputWithFocusButton() {
  const inputRef = useRef(null);

  const handleClick = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" placeholder="Click the button to focus me" />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

Here, `inputRef.current` points to the actual `<input>` DOM node, so you can call native DOM methods like `.focus()`, `.scrollIntoView()`, or read `.offsetWidth`.

## Use Case 2: Storing Mutable Values

`useRef` can hold any value — not just DOM nodes — that you want to persist between renders without triggering a re-render.

```jsx
import { useRef, useState } from 'react';

function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);

  const start = () => {
    if (intervalRef.current !== null) return; // already running
    intervalRef.current = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);
  };

  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

The interval ID is stored in `intervalRef.current`. Storing it in state instead would cause an unnecessary re-render every time it's set.

## Use Case 3: Tracking Previous Values

`useRef` combined with `useEffect` is a common pattern for remembering a previous prop or state value.

```jsx
import { useRef, useEffect, useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  }, [count]);

  const prevCount = prevCountRef.current;

  return (
    <div>
      <p>Now: {count}, Before: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

The ref is updated *after* the render, so during rendering it still holds the previous value.

## useRef vs useState

| | `useState` | `useRef` |
|---|---|---|
| Triggers re-render on change | Yes | No |
| Value persists across renders | Yes | Yes |
| Typical use | UI data that affects what's rendered | DOM access, timers, mutable values that don't affect rendering |
| Access pattern | `value` directly | `ref.current` |

Rule of thumb: **if changing the value should update the UI, use `useState`. If it shouldn't, use `useRef`.**

## Common Pitfalls

1. **Reading `.current` during render for DOM refs** — DOM refs are only attached after the component mounts. Reading `inputRef.current` during the initial render (before the DOM exists) returns `null`.
2. **Using a ref to store data that affects the UI** — Since updating `.current` doesn't trigger a re-render, the UI won't reflect the change until something else causes a re-render. Use `useState` instead for anything the user should see change.
3. **Overusing refs to avoid re-renders** — Refs bypass React's rendering model. Relying on them too heavily can make component behavior harder to reason about and debug.
4. **Mutating `.current` during rendering** — Refs should generally be read and written in event handlers or effects, not directly in the render body (except for the one-time initialization pattern).

## When to Use / Avoid

**Good use cases:**
- Managing focus, text selection, or media playback
- Triggering imperative animations
- Integrating with third-party DOM libraries
- Storing timeout/interval IDs
- Tracking previous prop/state values
- Counting renders without causing more renders

**Avoid when:**
- The value should be reflected in the UI (use `useState` instead)
- You need to derive computed values on every render (use plain variables or `useMemo`)


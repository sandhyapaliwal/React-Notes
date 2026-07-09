#  `useEffect` Hook 

`useEffect` lets you perform **side effects** in function components — things like data fetching, subscriptions, timers, manually changing the DOM, or logging. It's the function-component replacement for lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.

## Basic Syntax

```jsx
import { useEffect } from "react";

useEffect(() => {
  // side effect logic here

  return () => {
    // optional cleanup logic here
  };
}, [/* dependency array */]);
```

- **Effect function**: runs after the component renders.
- **Cleanup function** (optional): runs before the component re-renders (with new effect) and when it unmounts.
- **Dependency array**: controls when the effect re-runs.

## Dependency Array Behavior

| Dependency Array | Effect Runs |
|---|---|
| Omitted | After **every** render |
| `[]` (empty) | Only **once**, after the initial render |
| `[a, b]` | After initial render, and whenever `a` or `b` changes |

### 1. Runs on every render
```jsx
useEffect(() => {
  console.log("Runs after every render");
});
```

### 2. Runs only once (on mount)
```jsx
useEffect(() => {
  console.log("Runs once, like componentDidMount");
}, []);
```

### 3. Runs when specific values change
```jsx
useEffect(() => {
  console.log(`Count changed to ${count}`);
}, [count]);
```

## Cleanup Function

Return a function from your effect to clean up subscriptions, timers, or listeners and avoid memory leaks.

```jsx
useEffect(() => {
  const interval = setInterval(() => {
    console.log("Tick");
  }, 1000);

  return () => clearInterval(interval); // cleanup
}, []);
```

The cleanup runs:
- Before the effect runs again (if dependencies changed)
- When the component unmounts

## Common Use Cases

### Fetching Data
```jsx
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let ignore = false;

    async function fetchUser() {
      const res = await fetch(`/api/users/${userId}`);
      const data = await res.json();
      if (!ignore) setUser(data);
    }

    fetchUser();

    return () => {
      ignore = true; // avoid setting state after unmount
    };
  }, [userId]);

  if (!user) return <p>Loading...</p>;
  return <p>{user.name}</p>;
}
```

### Subscribing to an Event
```jsx
useEffect(() => {
  function handleResize() {
    console.log(window.innerWidth);
  }

  window.addEventListener("resize", handleResize);

  return () => window.removeEventListener("resize", handleResize);
}, []);
```

### Updating the Document Title
```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]);
```


## Quick Reference

| Goal | Pattern |
|---|---|
| Run once on mount | `useEffect(() => {...}, [])` |
| Run on every render | `useEffect(() => {...})` |
| Run when a value changes | `useEffect(() => {...}, [value])` |
| Clean up on unmount | Return a function from the effect |


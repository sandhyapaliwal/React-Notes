# Cleanup Function in React

A **cleanup function** is an optional function you return from a `useEffect` callback. React runs it to undo whatever the effect set up — before the effect runs again, and when the component unmounts — preventing memory leaks, stale subscriptions, and other bugs common in components that set up timers, subscriptions, or event listeners.

## Table of Contents

- [Why Cleanup Functions Matter](#why-cleanup-functions-matter)
- [Basic Syntax](#basic-syntax)
- [When Does Cleanup Run?](#when-does-cleanup-run)
- [Example 1: Clearing a Timer](#example-1-clearing-a-timer)
- [Example 2: Removing an Event Listener](#example-2-removing-an-event-listener)
- [Example 3: Unsubscribing from a Subscription](#example-3-unsubscribing-from-a-subscription)
- [Example 4: Cancelling a Fetch Request](#example-4-cancelling-a-fetch-request)
- [Example 5: Cleanup with Dependencies](#example-5-cleanup-with-dependencies)
- [Cleanup and React 18 Strict Mode](#cleanup-and-react-18-strict-mode)
- [Common Pitfalls](#common-pitfalls)
- [Best Practices](#best-practices)

## Why Cleanup Functions Matter

Many side effects — timers, subscriptions, DOM event listeners, WebSocket connections — create something that persists outside of React's normal render cycle. If you don't clean these up:

- **Memory leaks**: timers and subscriptions keep running after a component unmounts.
- **Duplicate effects**: every re-render (when dependencies change) could stack a new subscription on top of the old one instead of replacing it.
- **State updates on unmounted components**: async operations that complete after unmount can trigger React warnings or bugs by trying to update state that no longer exists.
- **Stale closures**: old event listeners referencing outdated props/state can fire unexpected behavior.

## Basic Syntax

```jsx
import { useEffect } from "react";

function MyComponent() {
  useEffect(() => {
    // Effect logic — runs after render

    return () => {
      // Cleanup logic — runs before the next effect, and on unmount
    };
  }, []); // dependency array

  return <div>My Component</div>;
}
```

The function you `return` from inside `useEffect` **is** the cleanup function. It's optional — if your effect doesn't set up anything that needs undoing, you can skip it entirely.

## When Does Cleanup Run?

1. **Before the effect re-runs** — if any value in the dependency array changes, React runs the cleanup from the *previous* render before running the new effect.
2. **When the component unmounts** — React runs the cleanup one final time to tear down whatever was set up.

```
Render 1 → Effect runs
Render 2 (deps changed) → Cleanup (from Render 1) runs → Effect runs again
Unmount → Cleanup (from latest effect) runs
```

## Example 1: Clearing a Timer

Without cleanup, every re-render of a component using `setInterval` would spawn a new interval, stacking multiple timers.

```jsx
import { useState, useEffect } from "react";

function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => clearInterval(intervalId); // stops the timer
  }, []);

  return <p>Current time: {time.toLocaleTimeString()}</p>;
}
```

## Example 2: Removing an Event Listener

```jsx
import { useState, useEffect } from "react";

function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);

    window.addEventListener("resize", handleResize);

    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return <p>Window width: {width}px</p>;
}
```

Without removing the listener, every mount of `WindowWidth` would add another listener that's never removed, even after the component unmounts.

## Example 3: Unsubscribing from a Subscription

Common pattern with real-time data sources like WebSockets, Firebase, or custom pub/sub systems.

```jsx
import { useState, useEffect } from "react";
import { chatRoomAPI } from "./chatRoomAPI";

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = chatRoomAPI.connect(roomId);

    connection.on("message", (msg) => {
      setMessages((prev) => [...prev, msg]);
    });

    return () => connection.disconnect(); // unsubscribe when roomId changes or unmount
  }, [roomId]);

  return (
    <ul>
      {messages.map((msg, i) => (
        <li key={i}>{msg}</li>
      ))}
    </ul>
  );
}
```

Here, cleanup is especially important because `roomId` is a dependency — switching rooms should disconnect from the old room before connecting to the new one, not leave both connections open.

## Example 4: Cancelling a Fetch Request

Prevents a "Can't perform a React state update on an unmounted component" warning when a fetch resolves after the component has already unmounted.

```jsx
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    async function fetchUser() {
      const res = await fetch(`/api/users/${userId}`);
      const data = await res.json();
      if (!isCancelled) {
        setUser(data);
      }
    }

    fetchUser();

    return () => {
      isCancelled = true; // prevents state update after unmount
    };
  }, [userId]);

  if (!user) return <p>Loading...</p>;
  return <h1>{user.name}</h1>;
}
```

For built-in fetch cancellation, you can also use `AbortController`:

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then((res) => res.json())
    .then((data) => setUser(data))
    .catch((err) => {
      if (err.name !== "AbortError") console.error(err);
    });

  return () => controller.abort();
}, [userId]);
```

## Example 5: Cleanup with Dependencies

Cleanup functions "see" the props/state from the render they belong to — not the latest ones. This matters when a component re-renders frequently.

```jsx
import { useEffect } from "react";

function RoomLogger({ roomId }) {
  useEffect(() => {
    console.log(`Connecting to room: ${roomId}`);

    return () => {
      console.log(`Disconnecting from room: ${roomId}`); // logs the OLD roomId, correctly
    };
  }, [roomId]);

  return null;
}
```

If `roomId` changes from `"lobby"` to `"general"`, the console correctly logs:
```
Disconnecting from room: lobby
Connecting to room: general
```

## Cleanup and React 18 Strict Mode

In **development**, React 18's `<StrictMode>` intentionally runs effects **twice** (mount → cleanup → mount again) to help you catch effects that aren't properly cleaned up. This does **not** happen in production builds.

```jsx
<StrictMode>
  <App />
</StrictMode>
```

If your app breaks or behaves oddly only in development with Strict Mode, it's usually a sign the effect's cleanup function is missing or incomplete — this is a debugging feature, not a bug in React.

## Common Pitfalls

- **Forgetting cleanup for subscriptions/listeners** — leads to memory leaks and duplicate handlers over time.
- **Cleaning up the wrong reference** — make sure the cleanup function closes over the same timer ID / listener reference the effect created, not a stale or new one.
- **Updating state after unmount** — always guard async callbacks with a cancellation flag or `AbortController`, as shown above.
- **Confusing cleanup timing** — remember cleanup runs *before* the next effect execution, not only on unmount; this is essential for effects with a non-empty dependency array.
- **Returning something other than a function** — `useEffect`'s callback must return either `undefined`/nothing, or a function. Returning a Promise (e.g., by making the effect callback `async`) is not allowed.

```jsx
// ❌ Wrong — async function returns a Promise, not a cleanup function
useEffect(async () => {
  await fetchData();
}, []);

// ✅ Correct — define the async function inside, call it, don't return it
useEffect(() => {
  async function load() {
    await fetchData();
  }
  load();
}, []);
```

## Best Practices

- Always clean up anything that persists outside the component: timers, listeners, subscriptions, WebSocket/DOM connections.
- Prefer `AbortController` for fetch cancellation when the API/browser environment supports it.
- Keep the setup and corresponding teardown logic close together and symmetrical — for every `addEventListener`, a matching `removeEventListener`; for every `subscribe`, a matching `unsubscribe`.
- Test your effects with `<StrictMode>` enabled in development to catch missing or broken cleanup early.
- If an effect doesn't create anything that needs tearing down (e.g., a simple `document.title` update), it's fine to skip the cleanup function entirely.

---

**Further Reading:**
- [React Docs – Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [React Docs – You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [React Docs – Lifecycle of Reactive Effects](https://react.dev/learn/lifecycle-of-reactive-effects)

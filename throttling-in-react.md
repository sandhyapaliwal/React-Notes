# Throttling in React

Throttling is a performance technique that limits how often a function can run, ensuring it executes at most once in a specified time interval — no matter how many times the triggering event fires. In React, it's commonly used to control the rate of expensive operations like scroll handlers, window resize listeners, API calls, or button clicks.

## Table of Contents

- [What is Throttling?](#what-is-throttling)
- [Throttling vs Debouncing](#throttling-vs-debouncing)
- [Building a Throttle Function](#building-a-throttle-function)
- [Example 1: useThrottle Custom Hook (Value)](#example-1-usethrottle-custom-hook-value)
- [Example 2: useThrottledCallback Custom Hook (Function)](#example-2-usethrottledcallback-custom-hook-function)
- [Example 3: Throttling a Scroll Event](#example-3-throttling-a-scroll-event)
- [Example 4: Throttling a Window Resize Handler](#example-4-throttling-a-window-resize-handler)
- [Example 5: Throttling a Button Click (Prevent Spam Clicks)](#example-5-throttling-a-button-click-prevent-spam-clicks)
- [Using Lodash's throttle](#using-lodashs-throttle)

## What is Throttling?

Imagine a user scrolling a page — the `scroll` event can fire dozens of times per second. If your scroll handler does something expensive (like updating state or making an API call), it'll run far more often than necessary, hurting performance.

**Throttling** ensures the function runs at most once every X milliseconds, dropping extra calls in between:

```
Without throttling:  call call call call call call call call call call  (every scroll pixel)
With throttling:     call . . . call . . . call . . . call               (once per interval)
```

## Throttling vs Debouncing

These two are often confused but solve different problems:

| Feature   | Throttling                                                   | Debouncing                                                       |
| --------- | ------------------------------------------------------------ | ---------------------------------------------------------------- |
| Behavior  | Runs at most once every X ms, at regular intervals           | Runs only after the user stops triggering the event for X ms     |
| Best for  | Scroll, resize, mouse-move, drag events — continuous streams | Search input, form validation — waiting for the user to "finish" |
| Guarantee | Executes periodically, even during continuous activity       | May never execute if the event keeps firing non-stop             |
| Example   | Update scroll-progress bar every 200ms while scrolling       | Fire a search API call 400ms after the user stops typing         |

```
Throttle (every 300ms):   |--call-----call-----call-----call-->
Debounce (wait 300ms):    |-----------------------------call--> (fires once, after activity stops)
```

## Building a Throttle Function

A basic vanilla JS throttle implementation using a timestamp check:

```js
function throttle(fn, delay) {
  let lastCall = 0;

  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```

## Example 1: useThrottle Custom Hook (Value)

Throttles a rapidly changing **value** (not a function) — useful when you want a throttled version of state, like a live input value.

```jsx
import { useState, useEffect, useRef } from "react";

function useThrottle(value, delay = 500) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastUpdated = useRef(Date.now());

  useEffect(() => {
    const now = Date.now();
    const remaining = delay - (now - lastUpdated.current);

    if (remaining <= 0) {
      lastUpdated.current = now;
      setThrottledValue(value);
    } else {
      const timer = setTimeout(() => {
        lastUpdated.current = Date.now();
        setThrottledValue(value);
      }, remaining);

      return () => clearTimeout(timer);
    }
  }, [value, delay]);

  return throttledValue;
}

export default useThrottle;
```

**Usage:**

```jsx
function LiveCounter() {
  const [count, setCount] = useState(0);
  const throttledCount = useThrottle(count, 1000);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <p>Raw count: {count}</p>
      <p>Throttled count (updates at most once/sec): {throttledCount}</p>
    </div>
  );
}
```

## Example 2: useThrottledCallback Custom Hook (Function)

Throttles a **function** itself — useful for event handlers like scroll or resize.

```jsx
import { useRef, useCallback, useEffect } from "react";

function useThrottledCallback(callback, delay = 300) {
  const lastCall = useRef(0);
  const timeoutRef = useRef(null);
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  useEffect(() => {
    return () => clearTimeout(timeoutRef.current);
  }, []);

  return useCallback(
    (...args) => {
      const now = Date.now();
      const remaining = delay - (now - lastCall.current);

      if (remaining <= 0) {
        lastCall.current = now;
        callbackRef.current(...args);
      } else {
        clearTimeout(timeoutRef.current);
        timeoutRef.current = setTimeout(() => {
          lastCall.current = Date.now();
          callbackRef.current(...args);
        }, remaining);
      }
    },
    [delay],
  );
}

export default useThrottledCallback;
```

## Example 3: Throttling a Scroll Event

```jsx
import { useState, useEffect } from "react";
import useThrottledCallback from "./useThrottledCallback";

function ScrollProgress() {
  const [scrollY, setScrollY] = useState(0);

  const handleScroll = useThrottledCallback(() => {
    setScrollY(window.scrollY);
  }, 200);

  useEffect(() => {
    window.addEventListener("scroll", handleScroll);
    return () => window.removeEventListener("scroll", handleScroll);
  }, [handleScroll]);

  return <p>Scroll position: {scrollY}px</p>;
}
```

Without throttling, `setScrollY` could be called hundreds of times per second during a fast scroll — throttling caps it to once every 200ms.

## Example 4: Throttling a Window Resize Handler

```jsx
import { useState, useEffect } from "react";
import useThrottledCallback from "./useThrottledCallback";

function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  const handleResize = useThrottledCallback(() => {
    setSize({ width: window.innerWidth, height: window.innerHeight });
  }, 250);

  useEffect(() => {
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, [handleResize]);

  return (
    <p>
      Window size: {size.width} x {size.height}
    </p>
  );
}
```

## Example 5: Throttling a Button Click (Prevent Spam Clicks)

Prevents a user from firing an action (like submitting a form or triggering an API call) multiple times in quick succession.

```jsx
import useThrottledCallback from "./useThrottledCallback";

function LikeButton() {
  const handleLike = useThrottledCallback(() => {
    console.log("Like registered!");
    // API call to register a "like" goes here
  }, 1000);

  return <button onClick={handleLike}>👍 Like</button>;
}
```

## Using Lodash's throttle

For production apps, many teams prefer the battle-tested `lodash.throttle` instead of a hand-rolled version:

```bash
npm install lodash
```

```jsx
import { useMemo, useEffect } from "react";
import throttle from "lodash/throttle";

function ScrollTracker() {
  const handleScroll = useMemo(
    () =>
      throttle(() => {
        console.log("Scroll Y:", window.scrollY);
      }, 200),
    [],
  );

  useEffect(() => {
    window.addEventListener("scroll", handleScroll);
    return () => {
      window.removeEventListener("scroll", handleScroll);
      handleScroll.cancel(); // cancel any pending trailing call
    };
  }, [handleScroll]);

  return <p>Check the console while scrolling</p>;
}
```

`lodash.throttle` also supports `leading`/`trailing` options for fine control over whether the first and/or last call in a burst fires.

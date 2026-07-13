# useLayoutEffect Hook 
`useLayoutEffect` is a React Hook that works almost identically to `useEffect`, but with one key difference: it fires **synchronously after DOM mutations but before the browser paints the screen**. This makes it useful for reading layout information from the DOM and making visual adjustments before the user sees anything.

## Table of Contents

- [Why useLayoutEffect?](#why-uselayouteffect)
- [Basic Syntax](#basic-syntax)
- [useLayoutEffect vs useEffect Timing](#uselayouteffect-vs-useeffect-timing)
- [Example: Measuring an Element](#example-measuring-an-element)
- [Example: Preventing Visual Flicker](#example-preventing-visual-flicker)
- [Example: Synchronizing Scroll Position](#example-synchronizing-scroll-position)
- [Common Pitfalls](#common-pitfalls)

---

## Why useLayoutEffect?

`useEffect` runs *after* the browser has painted the screen. That's usually fine — but if your effect needs to measure or change the DOM in a way that affects what the user sees (like positioning a tooltip based on an element's size), a `useEffect` update can cause a visible flicker: the browser paints once, then your effect changes something, and it paints again.

`useLayoutEffect` runs *before* the paint, so any DOM reads/writes it does happen invisibly to the user — no flicker.

```
render → DOM updated → useLayoutEffect runs → browser paints → useEffect runs
```

## Basic Syntax

```jsx
import { useLayoutEffect } from 'react';

function MyComponent() {
  useLayoutEffect(() => {
    // runs synchronously after DOM mutations, before paint
    return () => {
      // optional cleanup, runs before the next effect or on unmount
    };
  }, [dependencies]);

  return <div>...</div>;
}
```

The signature is identical to `useEffect`: a callback function and a dependency array.

## useLayoutEffect vs useEffect Timing

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Timing | After the browser paints | Before the browser paints |
| Blocking | Non-blocking (async-like) | Blocking (synchronous) |
| Visual flicker risk | Possible for layout-affecting changes | Avoided |
| Typical use | Data fetching, subscriptions, logging | DOM measurements, layout adjustments |
| Performance | Doesn't delay paint | Can delay paint if the effect is slow |

## Example: Measuring an Element

A common use case is measuring a DOM node's size right after it renders, before the user sees the layout.

```jsx
import { useLayoutEffect, useRef, useState } from 'react';

function Tooltip({ text }) {
  const tooltipRef = useRef(null);
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    const rect = tooltipRef.current.getBoundingClientRect();
    setWidth(rect.width);
  }, [text]);

  return (
    <div ref={tooltipRef} className="tooltip">
      {text} (width: {width}px)
    </div>
  );
}
```

Because this runs before paint, the measured `width` is available and any resulting layout change (like repositioning based on width) happens before anything is shown.

## Example: Preventing Visual Flicker

```jsx
import { useLayoutEffect, useRef } from 'react';

function AutoScrollBox({ messages }) {
  const containerRef = useRef(null);

  useLayoutEffect(() => {
    const container = containerRef.current;
    container.scrollTop = container.scrollHeight;
  }, [messages]);

  return (
    <div ref={containerRef} style={{ overflowY: 'auto', height: '200px' }}>
      {messages.map((msg, i) => (
        <p key={i}>{msg}</p>
      ))}
    </div>
  );
}
```

If this scroll adjustment used `useEffect` instead, the user might briefly see the box in its old scroll position before it jumps — `useLayoutEffect` avoids that.

## Example: Synchronizing Scroll Position

```jsx
import { useLayoutEffect, useState } from 'react';

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);

  useLayoutEffect(() => {
    function handleScroll() {
      setScrollY(window.scrollY);
    }
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return <p>Scroll position: {scrollY}px</p>;
}
```

This works the same as it would with `useEffect` for a non-layout-critical case like this — but is shown here to illustrate the identical hook signature.

## Common Pitfalls

1. **Using it by default instead of `useEffect`** — Because `useLayoutEffect` blocks the browser from painting until it finishes, overusing it (or putting slow logic inside it) can hurt performance and make the UI feel less responsive.
2. **Server-side rendering (SSR) warnings** — `useLayoutEffect` doesn't run during server rendering, and React will emit a warning if it's used in a component rendered on the server without a client-only guard. Use `useEffect` instead for SSR-safe code, or conditionally use `useLayoutEffect` only after the app fully hydrates on the client.
3. **Using it for non-layout side effects** — Data fetching, subscriptions, and logging don't need to block paint. Use `useEffect` for these to keep the UI responsive.
4. **Forgetting cleanup functions** — Just like `useEffect`, if you add event listeners, timers, or subscriptions, return a cleanup function to remove them.


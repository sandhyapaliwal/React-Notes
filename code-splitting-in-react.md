# Code Splitting in React

Code splitting is a technique that breaks a large JavaScript bundle into smaller chunks that are loaded **on demand**, instead of all at once. This reduces the initial bundle size, speeds up first load, and improves overall performance — especially in larger applications.

## Table of Contents

- [Why Code Splitting?](#why-code-splitting)
- [How Bundlers Handle It](#how-bundlers-handle-it)
- [React.lazy and Suspense](#reactlazy-and-suspense)
- [Example 1: Basic Lazy-Loaded Component](#example-1-basic-lazy-loaded-component)
- [Example 2: Route-Based Code Splitting](#example-2-route-based-code-splitting)
- [Example 3: Component-Based Splitting (On Interaction)](#example-3-component-based-splitting-on-interaction)
- [Example 4: Named Exports with React.lazy](#example-4-named-exports-with-reactlazy)
- [Example 5: Handling Load Errors with Error Boundaries](#example-5-handling-load-errors-with-error-boundaries)
- [Preloading Chunks](#preloading-chunks)
- [Verifying Code Splitting Works](#verifying-code-splitting-works)

## Why Code Splitting?

Without code splitting, bundlers like Webpack or Vite combine your entire app — every page, every component, every library — into one large JavaScript file. The browser has to download, parse, and execute all of it before the app becomes interactive, even for parts of the UI the user may never visit.

Code splitting fixes this by:

- **Reducing initial load time** — users only download the code needed for the page they're currently viewing.
- **Improving performance on slow networks/devices** — smaller chunks mean faster Time to Interactive (TTI).
- **Loading heavy features lazily** — things like charts, rich text editors, or admin panels only load when actually needed.
- **Better caching** — splitting vendor code from app code means users don't re-download unchanged libraries after every deploy.

## How Bundlers Handle It

Modern bundlers (Webpack, Vite, Rollup) support code splitting through **dynamic `import()`**. Instead of a static import at the top of a file:

```js
import MyComponent from "./MyComponent"; // bundled into the main chunk
```

A dynamic import returns a Promise and creates a **separate chunk** that's fetched only when called:

```js
import("./MyComponent").then((module) => {
  // module.default is MyComponent
});
```

React builds directly on top of this browser/bundler feature via `React.lazy`.

## React.lazy and Suspense

`React.lazy` lets you render a dynamically imported component as if it were a regular one. It must be paired with `<Suspense>`, which shows fallback UI (like a spinner) while the chunk is loading.

```jsx
import { lazy, Suspense } from "react";

const Profile = lazy(() => import("./Profile"));

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <Profile />
    </Suspense>
  );
}
```

- `lazy()` takes a function that returns a dynamic `import()` — it must resolve to a module with a **default export**.
- `<Suspense>` can wrap one or many lazy components, and shows `fallback` until all of them are ready.

## Example 1: Basic Lazy-Loaded Component

```jsx
// HeavyChart.jsx
function HeavyChart() {
  // imagine this imports a large charting library
  return <div>📊 Rendering complex chart...</div>;
}

export default HeavyChart;
```

```jsx
// App.jsx
import { lazy, Suspense } from "react";

const HeavyChart = lazy(() => import("./HeavyChart"));

function App() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<p>Loading chart...</p>}>
        <HeavyChart />
      </Suspense>
    </div>
  );
}

export default App;
```

`HeavyChart.jsx` (and anything it imports) is now split into its own chunk, downloaded only when `App` renders.

## Example 2: Route-Based Code Splitting

The most common and impactful place to apply code splitting — each page/route becomes its own chunk, loaded only when the user navigates to it. Pairs naturally with React Router.

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route } from "react-router-dom";

const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));
const Dashboard = lazy(() => import("./pages/Dashboard"));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<p>Loading page...</p>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

export default App;
```

Now visiting `/` never downloads the code for `/dashboard` unless the user actually navigates there.

## Example 3: Component-Based Splitting (On Interaction)

Splitting doesn't have to be route-based — you can defer loading a component until the user triggers an action (like opening a modal).

```jsx
import { lazy, Suspense, useState } from "react";

const ImageEditor = lazy(() => import("./ImageEditor"));

function Gallery() {
  const [showEditor, setShowEditor] = useState(false);

  return (
    <div>
      <button onClick={() => setShowEditor(true)}>Edit Image</button>

      {showEditor && (
        <Suspense fallback={<p>Loading editor...</p>}>
          <ImageEditor />
        </Suspense>
      )}
    </div>
  );
}

export default Gallery;
```

The (potentially large) `ImageEditor` bundle is only fetched the moment the user clicks "Edit Image" — never on initial page load.

## Example 4: Named Exports with React.lazy

`React.lazy` only supports default exports directly. If a module uses named exports, re-export it as default in a small wrapper or resolve it inline:

```jsx
// Chart.js — has a named export
export function Chart() {
  /* ... */
}
```

```jsx
const Chart = lazy(() =>
  import("./Chart").then((module) => ({ default: module.Chart })),
);
```

## Example 5: Handling Load Errors with Error Boundaries

Dynamic imports can fail (e.g., network issues, or a stale chunk after a redeploy). Wrap lazy components in an **Error Boundary** to catch these gracefully.

```jsx
import { Component } from "react";

class ChunkErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <p>Something went wrong loading this section. Please refresh.</p>;
    }
    return this.props.children;
  }
}

export default ChunkErrorBoundary;
```

```jsx
import { lazy, Suspense } from "react";
import ChunkErrorBoundary from "./ChunkErrorBoundary";

const Settings = lazy(() => import("./Settings"));

function App() {
  return (
    <ChunkErrorBoundary>
      <Suspense fallback={<p>Loading settings...</p>}>
        <Settings />
      </Suspense>
    </ChunkErrorBoundary>
  );
}
```

## Preloading Chunks

To avoid a loading flicker on predictable navigation (e.g., hovering a link), you can trigger the dynamic import ahead of time:

```jsx
const Dashboard = lazy(() => import("./pages/Dashboard"));

function NavLink() {
  const preload = () => import("./pages/Dashboard");

  return (
    <a href="/dashboard" onMouseEnter={preload}>
      Dashboard
    </a>
  );
}
```

Calling `import()` again simply resolves from the browser's module cache if already loaded, so this is safe to call multiple times.

## Verifying Code Splitting Works

- **Build output**: after running your production build (`npm run build`), check the `dist`/`build` folder — you should see multiple `.js` chunk files instead of one large bundle.
- **Network tab**: open DevTools → Network, reload the app, and confirm only the initial chunk loads; navigate to a lazy route and watch a new chunk fetch on demand.
- **Bundle analyzer**: tools like `rollup-plugin-visualizer` (Vite) or `webpack-bundle-analyzer` (Webpack) visualize chunk sizes to confirm splitting is effective.

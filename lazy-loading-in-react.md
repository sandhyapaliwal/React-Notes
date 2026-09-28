# Lazy Loading in React

**Lazy loading** is a performance technique where you load parts of your app (components, routes, images, data) only when they are actually needed, instead of downloading everything upfront. In React, this shrinks the initial JavaScript bundle, so the first page loads faster.

React provides two built-in tools for this: **`React.lazy()`** and **`<Suspense>`**.

## Table of Contents

- [Why Lazy Loading?](#why-lazy-loading)
- [How It Works](#how-it-works)
- [Basic Usage: React.lazy and Suspense](#basic-usage-reactlazy-and-suspense)
- [Example 1: Lazy Loading a Component](#example-1-lazy-loading-a-component)
- [Example 2: Route-Based Lazy Loading](#example-2-route-based-lazy-loading)
- [Example 3: Lazy Loading on User Interaction](#example-3-lazy-loading-on-user-interaction)
- [Example 4: Handling Load Errors with Error Boundaries](#example-4-handling-load-errors-with-error-boundaries)
- [Example 5: Lazy Loading Images](#example-5-lazy-loading-images)
- [Named Exports with React.lazy](#named-exports-with-reactlazy)
- [Lazy Loading with Vite](#lazy-loading-with-vite)

## Why Lazy Loading?

By default, bundlers (Vite, Webpack) combine all your code into one large JavaScript file. Users must download all of it before seeing anything, even code for pages they may never visit.

Lazy loading helps by:

- **Reducing the initial bundle size**, so the first paint is faster.
- **Improving performance metrics** such as Lighthouse score, First Contentful Paint (FCP), and Time to Interactive (TTI).
- **Saving bandwidth**, since users only download code for features they actually use.
- **Scaling better** as the app grows, because new pages don't slow down the initial load.

## How It Works

Lazy loading relies on **code splitting** using the dynamic `import()` syntax. Instead of importing a component at the top of the file, you import it _when it is needed_. The bundler automatically splits that component into a separate file (a "chunk") that loads on demand.

```jsx
// Regular (eager) import: included in the main bundle
import Dashboard from "./Dashboard";

// Lazy import: split into a separate chunk, loaded when first rendered
const Dashboard = React.lazy(() => import("./Dashboard"));
```

## Basic Usage: React.lazy and Suspense

- **`React.lazy(fn)`** takes a function that returns a dynamic `import()`, and gives you a component that loads on demand.
- **`<Suspense fallback={...}>`** wraps lazy components and shows a fallback UI (like a spinner) while the chunk is downloading.

```jsx
import { lazy, Suspense } from "react";

const HeavyChart = lazy(() => import("./HeavyChart"));

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <HeavyChart />
    </Suspense>
  );
}

export default App;
```

A lazy component **must** be rendered inside a `<Suspense>` boundary, or React will throw an error.

## Example 1: Lazy Loading a Component

```jsx
import { lazy, Suspense, useState } from "react";

const Comments = lazy(() => import("./Comments"));

function Post() {
  const [showComments, setShowComments] = useState(false);

  return (
    <div>
      <h1>My Blog Post</h1>
      <button onClick={() => setShowComments(true)}>Show Comments</button>

      {showComments && (
        <Suspense fallback={<p>Loading comments...</p>}>
          <Comments />
        </Suspense>
      )}
    </div>
  );
}
```

The `Comments` code is only downloaded after the user clicks the button.

## Example 2: Route-Based Lazy Loading

Routes are the most common and effective place to lazy load, because users only visit one page at a time.

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route } from "react-router-dom";

const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));
const Dashboard = lazy(() => import("./pages/Dashboard"));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div className="spinner">Loading page...</div>}>
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

Each page becomes its own chunk. Visiting `/about` downloads only the About chunk.

## Example 3: Lazy Loading on User Interaction

Load heavy libraries (charts, editors, PDF generators) only when the user needs them.

```jsx
import { useState } from "react";

function ExportButton({ data }) {
  const [loading, setLoading] = useState(false);

  const handleExport = async () => {
    setLoading(true);
    // Library is downloaded only when the button is clicked
    const { default: jsPDF } = await import("jspdf");
    const doc = new jsPDF();
    doc.text(JSON.stringify(data), 10, 10);
    doc.save("export.pdf");
    setLoading(false);
  };

  return (
    <button onClick={handleExport} disabled={loading}>
      {loading ? "Preparing..." : "Export as PDF"}
    </button>
  );
}
```

This uses dynamic `import()` directly (without `React.lazy`) for non-component code like libraries.

## Example 4: Handling Load Errors with Error Boundaries

A lazy chunk can fail to load (network drop, deployment changed file names). Wrap lazy components in an **Error Boundary** so the app doesn't crash.

```jsx
import { Component, lazy, Suspense } from "react";

class ErrorBoundary extends Component {
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

const Reports = lazy(() => import("./Reports"));

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<p>Loading...</p>}>
        <Reports />
      </Suspense>
    </ErrorBoundary>
  );
}
```

## Example 5: Lazy Loading Images

Images are often the heaviest assets. Modern browsers support native lazy loading with a single attribute:

```jsx
function Gallery({ images }) {
  return (
    <div>
      {images.map((img) => (
        <img
          key={img.id}
          src={img.url}
          alt={img.alt}
          loading="lazy"
          width="300"
          height="200"
        />
      ))}
    </div>
  );
}
```

`loading="lazy"` tells the browser to load the image only when it is near the viewport. Setting `width` and `height` prevents layout shift while images load.

## Named Exports with React.lazy

`React.lazy` only works with **default exports**. If your component uses a named export, re-map it:

```jsx
// Components.js has: export const Chart = () => {...}

const Chart = lazy(() =>
  import("./Components").then((module) => ({ default: module.Chart })),
);
```

## Lazy Loading with Vite

If you use Vite, no extra setup is needed. Vite automatically code-splits every dynamic `import()` into its own chunk. You can verify this by running:

```bash
npm run build
```

The build output lists separate JS files for each lazy-loaded component or page.

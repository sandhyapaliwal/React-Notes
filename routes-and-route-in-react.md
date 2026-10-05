# Routes & Route in React Router

`Routes` and `Route` are the core building blocks of **React Router** (v6+) used to define which component renders for a given URL. `Routes` acts as a container that looks through its child `Route` elements and renders the one whose `path` matches the current URL.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Basic Usage](#basic-usage)
- [How Matching Works](#how-matching-works)
- [Example 1: Simple Route Setup](#example-1-simple-route-setup)
- [Example 2: Dynamic Routes with Params](#example-2-dynamic-routes-with-params)
- [Example 3: Nested Routes](#example-3-nested-routes)
- [Example 4: Index Routes](#example-4-index-routes)
- [Example 5: 404 / Catch-All Route](#example-5-404--catch-all-route)
- [Example 6: Layout Routes (Shared UI)](#example-6-layout-routes-shared-ui)
- [Routes vs Switch (v5 vs v6)](#routes-vs-switch-v5-vs-v6)

## Prerequisites

`Routes` and `Route` require **React Router v6 or later**:

```bash
npm install react-router-dom
```

Your app must be wrapped in a `<BrowserRouter>` (or another Router) for routing to work:

```jsx
import { BrowserRouter } from "react-router-dom";
import App from "./App";

function Root() {
  return (
    <BrowserRouter>
      <App />
    </BrowserRouter>
  );
}
```

## Basic Usage

```jsx
import { Routes, Route } from "react-router-dom";
import Home from "./Home";
import About from "./About";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
    </Routes>
  );
}
```

- `<Routes>` wraps all your route definitions.
- Each `<Route>` maps a `path` to an `element` (the component to render).
- Only **one** matching route renders at a time — React Router v6 picks the best match automatically.

## How Matching Works

Unlike v5's `<Switch>`, React Router v6's `<Routes>` uses a **ranking algorithm** to pick the best match, rather than rendering the first route that matches top-to-bottom. This means route order generally doesn't matter as much as it did in v5 — more specific paths win over more general ones automatically.

```jsx
<Routes>
  <Route path="/users/:id" element={<UserDetail />} />
  <Route path="/users/new" element={<NewUser />} />
</Routes>
```

Visiting `/users/new` correctly renders `NewUser`, not `UserDetail`, even though `/users/:id` could technically match — v6 ranks the static segment (`new`) as more specific than the dynamic one (`:id`).

## Example 1: Simple Route Setup

```jsx
import { Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import Contact from "./pages/Contact";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  );
}

export default App;
```

## Example 2: Dynamic Routes with Params

Use a colon (`:paramName`) to define a dynamic segment, then read it with `useParams`.

```jsx
import { Routes, Route } from "react-router-dom";
import ProductList from "./ProductList";
import ProductDetail from "./ProductDetail";

function App() {
  return (
    <Routes>
      <Route path="/products" element={<ProductList />} />
      <Route path="/products/:productId" element={<ProductDetail />} />
    </Routes>
  );
}
```

```jsx
// ProductDetail.jsx
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { productId } = useParams();
  return <p>Showing product #{productId}</p>;
}
```

## Example 3: Nested Routes

Nested `<Route>` elements render inside their parent's `<Outlet />`, letting you share layout between related pages.

```jsx
import { Routes, Route } from "react-router-dom";
import Dashboard from "./Dashboard";
import DashboardHome from "./DashboardHome";
import Settings from "./Settings";
import Profile from "./Profile";

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />}>
        <Route index element={<DashboardHome />} />
        <Route path="settings" element={<Settings />} />
        <Route path="profile" element={<Profile />} />
      </Route>
    </Routes>
  );
}
```

```jsx
// Dashboard.jsx
import { Outlet, Link } from "react-router-dom";

function Dashboard() {
  return (
    <div>
      <nav>
        <Link to="/dashboard">Home</Link>
        <Link to="/dashboard/settings">Settings</Link>
        <Link to="/dashboard/profile">Profile</Link>
      </nav>
      <Outlet /> {/* renders DashboardHome, Settings, or Profile here */}
    </div>
  );
}
```

Note that nested route paths are **relative** — `path="settings"` under `/dashboard` automatically becomes `/dashboard/settings`.

## Example 4: Index Routes

An `index` route renders by default when the parent route's path matches exactly, with no further segments.

```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route index element={<DashboardHome />} />{" "}
  {/* renders at exactly /dashboard */}
  <Route path="settings" element={<Settings />} />
</Route>
```

Without the `index` route, visiting `/dashboard` would render `Dashboard`'s layout but leave the `<Outlet />` empty.

## Example 5: 404 / Catch-All Route

Use `path="*"` to catch any URL that doesn't match another route.

```jsx
import { Routes, Route } from "react-router-dom";
import Home from "./Home";
import NotFound from "./NotFound";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

Place the `*` route last — while v6's ranking generally handles order correctly, keeping the catch-all at the end keeps the code readable and predictable.

## Example 6: Layout Routes (Shared UI)

A parent route can provide shared UI (navbar, sidebar, footer) without itself requiring a path segment, using a pathless layout route.

```jsx
import { Routes, Route } from "react-router-dom";
import MainLayout from "./MainLayout";
import Home from "./Home";
import About from "./About";

function App() {
  return (
    <Routes>
      <Route element={<MainLayout />}>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Route>
    </Routes>
  );
}
```

```jsx
// MainLayout.jsx
import { Outlet } from "react-router-dom";

function MainLayout() {
  return (
    <div>
      <header>My Site Header</header>
      <Outlet />
      <footer>My Site Footer</footer>
    </div>
  );
}
```

## Routes vs Switch (v5 vs v6)

| Feature       | v5 `<Switch>`                                       | v6 `<Routes>`                                        |
| ------------- | --------------------------------------------------- | ---------------------------------------------------- |
| Matching      | Renders the **first** matching route, top to bottom | Automatically picks the **best** match via ranking   |
| Route prop    | `component` or `render`                             | `element` (takes JSX directly: `element={<Home />}`) |
| Nested routes | Required manually re-declaring full paths           | Relative paths + `<Outlet />` for shared layouts     |
| Route order   | Mattered a lot                                      | Rarely matters                                       |

```jsx
// v5
<Switch>
  <Route path="/about" component={About} />
</Switch>

// v6
<Routes>
  <Route path="/about" element={<About />} />
</Routes>
```

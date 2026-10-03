# React Router Setup

React Router is the standard library for handling client-side routing in React applications — it lets you build multi-page-feeling apps (with URLs, navigation, nested layouts) inside a single-page application (SPA).

## Table of Contents

- [Installation](#installation)
- [Step 1: Wrap Your App in a Router](#step-1-wrap-your-app-in-a-router)
- [Step 2: Define Routes](#step-2-define-routes)
- [Step 3: Add Navigation Links](#step-3-add-navigation-links)
- [Step 4: Create a 404 / Not Found Route](#step-4-create-a-404--not-found-route)
- [Nested Routes and Layouts](#nested-routes-and-layouts)
- [Dynamic Routes (URL Params)](#dynamic-routes-url-params)
- [Index Routes](#index-routes)
- [Protected / Private Routes](#protected--private-routes)
- [Data-Router Setup (createBrowserRouter)](#data-router-setup-createbrowserrouter)
- [Types of Routers](#types-of-routers)

## Installation

```bash
npm install react-router-dom
```

or with yarn:

```bash
yarn add react-router-dom
```

## Step 1: Wrap Your App in a Router

Every component that uses routing hooks (`useNavigate`, `useParams`, `<Link>`, etc.) must be a descendant of a Router. The most common choice for web apps is `BrowserRouter`.

```jsx
// main.jsx (or index.jsx)
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>,
);
```

## Step 2: Define Routes

Inside your `App` component, use `<Routes>` and `<Route>` to map URL paths to components.

```jsx
// App.jsx
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

## Step 3: Add Navigation Links

Use `<Link>` (or `<NavLink>` for automatic active-state styling) instead of plain `<a>` tags — this avoids full page reloads and keeps the SPA behavior intact.

```jsx
// Navbar.jsx
import { NavLink } from "react-router-dom";

function Navbar() {
  return (
    <nav>
      <NavLink
        to="/"
        end
        className={({ isActive }) => (isActive ? "active" : "")}
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) => (isActive ? "active" : "")}
      >
        About
      </NavLink>
      <NavLink
        to="/contact"
        className={({ isActive }) => (isActive ? "active" : "")}
      >
        Contact
      </NavLink>
    </nav>
  );
}

export default Navbar;
```

> Use `end` on the root `"/"` link so it's only marked active on an exact match — otherwise it stays "active" on every route, since `/` is a prefix of all paths.

## Step 4: Add a 404 / Not Found Route

Catch any URL that doesn't match a defined route using `path="*"`.

```jsx
import { Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import NotFound from "./pages/NotFound";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

## Nested Routes and Layouts

Nested routes let you share a layout (navbar, sidebar) across multiple child pages, rendering children into an `<Outlet />`.

```jsx
// Layout.jsx
import { Outlet } from "react-router-dom";
import Navbar from "./Navbar";

function Layout() {
  return (
    <div>
      <Navbar />
      <main>
        <Outlet /> {/* child route renders here */}
      </main>
    </div>
  );
}

export default Layout;
```

```jsx
// App.jsx
import { Routes, Route } from "react-router-dom";
import Layout from "./Layout";
import Home from "./pages/Home";
import Dashboard from "./pages/Dashboard";
import Settings from "./pages/Settings";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="dashboard" element={<Dashboard />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}
```

## Dynamic Routes (URL Params)

Use a colon (`:paramName`) to capture dynamic segments, then read them with `useParams`.

```jsx
<Route path="/users/:userId" element={<UserProfile />} />
```

```jsx
import { useParams } from "react-router-dom";

function UserProfile() {
  const { userId } = useParams();
  return <h1>Viewing user {userId}</h1>;
}
```

## Index Routes

An `index` route renders by default when the parent path matches exactly, without needing its own path segment — useful for a layout's default/home child.

```jsx
<Route path="/" element={<Layout />}>
  <Route index element={<Home />} /> {/* renders at exactly "/" */}
  <Route path="about" element={<About />} />
</Route>
```

## Protected / Private Routes

A common pattern for guarding routes that require authentication.

```jsx
// ProtectedRoute.jsx
import { Navigate, Outlet } from "react-router-dom";

function ProtectedRoute({ isAuthenticated }) {
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
}

export default ProtectedRoute;
```

```jsx
// App.jsx
<Routes>
  <Route path="/login" element={<Login />} />

  <Route element={<ProtectedRoute isAuthenticated={isAuthenticated} />}>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/settings" element={<Settings />} />
  </Route>
</Routes>
```

## Data-Router Setup (createBrowserRouter)

React Router v6.4+ introduced a newer, data-loading-friendly API using `createBrowserRouter`, which supports `loader`, `action`, and `errorElement` per route. This is the recommended setup for new projects that need data fetching tied to routes.

```jsx
// router.jsx
import { createBrowserRouter } from "react-router-dom";
import Layout from "./Layout";
import Home from "./pages/Home";
import About from "./pages/About";
import NotFound from "./pages/NotFound";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,
    errorElement: <NotFound />,
    children: [
      { index: true, element: <Home /> },
      { path: "about", element: <About /> },
    ],
  },
]);

export default router;
```

```jsx
// main.jsx
import ReactDOM from "react-dom/client";
import { RouterProvider } from "react-router-dom";
import router from "./router";

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />,
);
```

> Note: with `createBrowserRouter`, you do **not** also use `<BrowserRouter>` — `RouterProvider` replaces it.

## Types of Routers

| Router                | Use Case                                                                     |
| --------------------- | ---------------------------------------------------------------------------- |
| `BrowserRouter`       | Standard web apps using the HTML5 History API (clean URLs like `/about`)     |
| `HashRouter`          | Static file hosting without server-side URL rewriting (URLs like `/#/about`) |
| `createBrowserRouter` | Modern data-router API with loaders/actions (recommended for new apps)       |
| `MemoryRouter`        | Non-browser environments — testing, React Native, embedded widgets           |

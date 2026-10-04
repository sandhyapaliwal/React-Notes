# BrowserRouter in React Router

`BrowserRouter` is the core routing component from **React Router** that enables client-side routing in a React application using the HTML5 History API. It keeps your UI in sync with the URL, allowing navigation without full page reloads — the foundation on which hooks like `useNavigate`, `useParams`, and `useMatch` all depend.

## Table of Contents

- [What is BrowserRouter?](#what-is-browserrouter)
- [Installation](#installation)
- [Basic Setup](#basic-setup)
- [Example 1: Defining Routes](#example-1-defining-routes)
- [Example 2: Navigation with Link](#example-2-navigation-with-link)
- [Example 3: Nested Routes](#example-3-nested-routes)
- [Example 4: 404 / Catch-All Route](#example-4-404--catch-all-route)
- [BrowserRouter vs Other Routers](#browserrouter-vs-other-routers)

## What is BrowserRouter?

`BrowserRouter` wraps your entire application (or the part of it that needs routing) and provides routing context to every component inside it. Without it, hooks like `useNavigate`, `useParams`, `useMatch`, and components like `<Routes>`, `<Route>`, and `<Link>` won't work — they all rely on the context `BrowserRouter` sets up.

It uses the browser's native **History API** (`pushState`, `replaceState`, `popstate`) to manage navigation, which means URLs look clean — like `/users/42` — instead of using a hash (`/#/users/42`).

## Installation

```bash
npm install react-router-dom
```

## Basic Setup

Wrap your app's root component with `BrowserRouter`, typically in your entry file (`main.jsx` or `index.js`):

```jsx
// main.jsx
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";

createRoot(document.getElementById("root")).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
);
```

Everything rendered inside `<App />` can now use routing components and hooks.

## Example 1: Defining Routes

Routes are typically defined using `<Routes>` and `<Route>` inside the `BrowserRouter`.

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

Visiting `/about` renders the `About` component in place, without reloading the page.

## Example 2: Navigation with Link

Use `<Link>` instead of `<a>` tags to navigate without a full page refresh.

```jsx
import { Link } from "react-router-dom";

function Navbar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>
    </nav>
  );
}

export default Navbar;
```

> Using a plain `<a href="/about">` would trigger a full page reload, losing the benefits of client-side routing — always use `<Link>` (or `<NavLink>`) inside a `BrowserRouter` app.

## Example 3: Nested Routes

`BrowserRouter` supports nested route structures, useful for shared layouts (like a dashboard with a sidebar).

```jsx
import { Routes, Route, Outlet } from "react-router-dom";

function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <Outlet /> {/* nested route content renders here */}
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} />
        <Route path="settings" element={<Settings />} />
        <Route path="profile" element={<Profile />} />
      </Route>
    </Routes>
  );
}
```

Visiting `/dashboard/settings` renders `DashboardLayout`, with `Settings` injected at the `<Outlet />`.

## Example 4: 404 / Catch-All Route

Use a wildcard path (`*`) to catch any URL that doesn't match a defined route.

```jsx
import { Routes, Route } from "react-router-dom";
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

## BrowserRouter vs Other Routers

React Router provides several router implementations for different environments:

| Router                | Use Case                                                                                          | URL Style                                      |
| --------------------- | ------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| `BrowserRouter`       | Standard web apps with server support for client-side routes                                      | Clean URLs: `/users/42`                        |
| `HashRouter`          | Static file hosting with no server-side routing support (e.g., GitHub Pages without extra config) | Hash-based: `/#/users/42`                      |
| `MemoryRouter`        | Testing, or non-browser environments (React Native, unit tests)                                   | URL kept in memory, not visible in address bar |
| `createBrowserRouter` | Newer data-router API (v6.4+) supporting loaders, actions, and better data-fetching patterns      | Clean URLs, same as `BrowserRouter`            |

```jsx
// HashRouter example — useful for static hosts without server-side route config
import { HashRouter } from "react-router-dom";

function App() {
  return <HashRouter>{/* routes */}</HashRouter>;
}
```

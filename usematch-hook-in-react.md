# useMatch Hook in React Router

`useMatch` is a hook from **React Router** (v6+) that checks whether a given path pattern matches the current URL. It returns a match object (with params, pathname, etc.) if there's a match, or `null` if there isn't — making it useful for conditional rendering, active-link styling, and reading route data without a `<Route>` element.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Basic Usage](#basic-usage)
- [What useMatch Returns](#what-usematch-returns)
- [Example 1: Highlighting an Active Nav Link](#example-1-highlighting-an-active-nav-link)
- [Example 2: Conditional Rendering Based on Route](#example-2-conditional-rendering-based-on-route)
- [Example 3: Reading Params Without a Route Element](#example-3-reading-params-without-a-route-element)
- [Example 4: Matching Nested or Partial Paths](#example-4-matching-nested-or-partial-paths)
- [Example 5: Case-Sensitive and End Options](#example-5-case-sensitive-and-end-options)
- [useMatch vs useLocation vs NavLink](#usematch-vs-uselocation-vs-navlink)

## Prerequisites

`useMatch` requires **React Router v6 or later**:

```bash
npm install react-router-dom
```

Your component must be rendered inside a `<BrowserRouter>` (or another Router) for the hook to work:

```jsx
import { BrowserRouter } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      {/* routes and components go here */}
    </BrowserRouter>
  );
}
```

## Basic Usage

```jsx
import { useMatch } from "react-router-dom";

function ProductLink() {
  const match = useMatch("/products/:productId");

  return match ? <p>You're viewing product {match.params.productId}</p> : null;
}
```

`useMatch` takes a path pattern (string) as its argument and compares it against the current URL's pathname.

## What useMatch Returns

If the pattern matches the current URL, `useMatch` returns an object like:

```js
{
  params: { productId: "42" },
  pathname: "/products/42",
  pathnameBase: "/products/42",
  pattern: { path: "/products/:productId", caseSensitive: false, end: true }
}
```

If it doesn't match, `useMatch` returns `null`.

## Example 1: Highlighting an Active Nav Link

A common use case — styling a navigation link differently when its route is active.

```jsx
import { useMatch, Link } from "react-router-dom";

function NavLink({ to, children }) {
  const match = useMatch(to);

  return (
    <Link to={to} style={{ fontWeight: match ? "bold" : "normal", color: match ? "#1F3864" : "#333" }}>
      {children}
    </Link>
  );
}

function Navbar() {
  return (
    <nav>
      <NavLink to="/">Home</NavLink>
      <NavLink to="/about">About</NavLink>
      <NavLink to="/contact">Contact</NavLink>
    </nav>
  );
}
```

> Note: React Router also ships a built-in `<NavLink>` component that does this automatically (see comparison below) — `useMatch` is handy when you need custom logic beyond simple styling.

## Example 2: Conditional Rendering Based on Route

```jsx
import { useMatch } from "react-router-dom";

function Header() {
  const isDashboard = useMatch("/dashboard/*");

  return (
    <header>
      <h1>My App</h1>
      {isDashboard && <p className="subtitle">Dashboard Mode</p>}
    </header>
  );
}
```

## Example 3: Reading Params Without a Route Element

Sometimes you need to know route params from a component that isn't directly rendered by a matching `<Route>` — for example, a shared layout or sidebar.

```jsx
import { useMatch } from "react-router-dom";

function Sidebar() {
  const match = useMatch("/teams/:teamId/*");

  if (!match) return <p>Select a team to see details</p>;

  return <p>Currently viewing team: {match.params.teamId}</p>;
}
```

## Example 4: Matching Nested or Partial Paths

By default, `useMatch` requires an **exact** match (`end: true`). To match a path and anything nested under it, add a trailing `/*`:

```jsx
import { useMatch } from "react-router-dom";

function SettingsSection() {
  // Matches /settings, /settings/profile, /settings/billing, etc.
  const match = useMatch("/settings/*");

  return match ? <p>Inside settings area</p> : <p>Not in settings</p>;
}
```

## Example 5: Case-Sensitive and End Options

For more control, pass an object instead of a plain string:

```jsx
import { useMatch } from "react-router-dom";

function Page() {
  const match = useMatch({
    path: "/Products/:id",
    caseSensitive: true, // "/products/1" would NOT match this
    end: false,          // allows matching nested paths too
  });

  return match ? <p>Matched!</p> : <p>No match</p>;
}
```

## useMatch vs useLocation vs NavLink

| Feature | `useMatch` | `useLocation` | `<NavLink>` |
|---|---|---|---|
| Type | Hook | Hook | Component |
| Returns | Match object or `null` for a given pattern | Current location object (`pathname`, `search`, `hash`, `state`) | Renders an `<a>` with active-state class/style automatically |
| Best for | Custom conditional logic based on whether a pattern matches | Reading raw URL info (query strings, hash, navigation state) | Simple active-link styling without writing your own matching logic |
| Example | `useMatch("/blog/:slug")` | `useLocation().pathname` | `<NavLink to="/blog" className={({isActive}) => isActive ? "active" : ""}>` |

```jsx
import { useLocation, useMatch } from "react-router-dom";

function DebugInfo() {
  const location = useLocation();
  const blogMatch = useMatch("/blog/:slug");

  return (
    <div>
      <p>Current path: {location.pathname}</p>
      <p>Is blog post: {blogMatch ? "Yes" : "No"}</p>
    </div>
  );
}
```


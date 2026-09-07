# useLocation Hook in React Router

`useLocation` is a hook from **React Router** (v6+) that returns the current location object, representing the app's current URL. It's the standard way to read the pathname, query string, hash, and any state passed during navigation — all without needing props drilling.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Basic Usage](#basic-usage)
- [The Location Object](#the-location-object)
- [Example 1: Reading the Current Pathname](#example-1-reading-the-current-pathname)
- [Example 2: Highlighting Active Navigation Links](#example-2-highlighting-active-navigation-links)
- [Example 3: Reading Passed State](#example-3-reading-passed-state)
- [Example 4: Scroll to Top on Route Change](#example-4-scroll-to-top-on-route-change)
- [Example 5: Tracking Page Views (Analytics)](#example-5-tracking-page-views-analytics)
- [Example 6: Reading Query Strings with useLocation](#example-6-reading-query-strings-with-uselocation)


## Prerequisites

`useLocation` requires **React Router v6 or later**:

```bash
npm install react-router-dom
```

Like other React Router hooks, the component using `useLocation` must be rendered inside a `<BrowserRouter>` (or another Router):

```jsx
import { BrowserRouter } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      {/* routes go here */}
    </BrowserRouter>
  );
}
```

## Basic Usage

```jsx
import { useLocation } from "react-router-dom";

function CurrentPath() {
  const location = useLocation();

  return <p>You are at: {location.pathname}</p>;
}
```

Every time the URL changes, `useLocation` returns a new location object, which triggers a re-render of the component using it.

## The Location Object

The object returned by `useLocation` has the following shape:

```js
{
  pathname: "/products/42",   // the path portion of the URL
  search: "?sort=asc",        // the query string, including "?"
  hash: "#reviews",           // the hash fragment, including "#"
  state: { from: "cart" },    // custom state passed via navigate() or <Link state={...}>
  key: "ac3df4"               // a unique key for this location entry
}
```

## Example 1: Reading the Current Pathname

```jsx
import { useLocation } from "react-router-dom";

function Breadcrumb() {
  const location = useLocation();
  const segments = location.pathname.split("/").filter(Boolean);

  return (
    <nav>
      {segments.map((seg, i) => (
        <span key={i}> / {seg}</span>
      ))}
    </nav>
  );
}
```

Visiting `/shop/shoes/running` renders: ` / shop / shoes / running`.

## Example 2: Highlighting Active Navigation Links

A common pattern is comparing the current pathname against a link's target to apply an "active" style.

```jsx
import { useLocation, Link } from "react-router-dom";

function NavBar() {
  const location = useLocation();

  const links = [
    { to: "/", label: "Home" },
    { to: "/about", label: "About" },
    { to: "/contact", label: "Contact" },
  ];

  return (
    <nav>
      {links.map((link) => (
        <Link
          key={link.to}
          to={link.to}
          style={{
            fontWeight: location.pathname === link.to ? "bold" : "normal",
            color: location.pathname === link.to ? "blue" : "black",
          }}
        >
          {link.label}
        </Link>
      ))}
    </nav>
  );
}
```

> Note: React Router also provides `<NavLink>`, which does this automatically via an `isActive` state — prefer it over manual `useLocation` comparisons for simple nav highlighting.

## Example 3: Reading Passed State

When you navigate with `navigate(path, { state })` or `<Link to={path} state={...}>`, the receiving component reads that data via `useLocation().state`.

```jsx
// Sender
navigate("/order-confirmation", {
  state: { orderId: "12345", total: 49.99 },
});
```

```jsx
// Receiver
import { useLocation } from "react-router-dom";

function OrderConfirmation() {
  const location = useLocation();
  const { orderId, total } = location.state || {};

  return (
    <p>
      Order {orderId} confirmed. Total: ${total}
    </p>
  );
}
```

This is also useful for redirect-after-login flows, where you store the page the user originally tried to visit:

```jsx
// In a protected route redirect
navigate("/login", { state: { from: location } });
```

## Example 4: Scroll to Top on Route Change

React Router doesn't automatically reset scroll position on navigation. `useLocation` lets you build a simple fix.

```jsx
import { useEffect } from "react";
import { useLocation } from "react-router-dom";

function ScrollToTop() {
  const { pathname } = useLocation();

  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);

  return null;
}
```

Render `<ScrollToTop />` once near the top of your app, inside the Router, so it runs on every route change.

## Example 5: Tracking Page Views (Analytics)

`useLocation` is the natural hook to trigger analytics events whenever the route changes.

```jsx
import { useEffect } from "react";
import { useLocation } from "react-router-dom";

function usePageTracking() {
  const location = useLocation();

  useEffect(() => {
    // Replace with your analytics provider's call
    window.gtag?.("event", "page_view", {
      page_path: location.pathname + location.search,
    });
  }, [location]);
}

export default usePageTracking;
```

Call `usePageTracking()` once inside a top-level layout component.

## Example 6: Reading Query Strings with useLocation

While `useSearchParams` is the recommended way to work with query strings, you can also parse them manually from `location.search` if needed:

```jsx
import { useLocation } from "react-router-dom";

function useQuery() {
  const { search } = useLocation();
  return new URLSearchParams(search);
}

function SearchResults() {
  const query = useQuery();
  const keyword = query.get("q");

  return <p>Results for: {keyword}</p>;
}
```


# Link & NavLink in React Router

`Link` and `NavLink` are components from **React Router** used to navigate between routes in a single-page application — without triggering a full page reload, unlike a regular HTML `<a>` tag. `NavLink` builds on `Link` by adding built-in support for styling the "active" link.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Why Not a Regular `<a>` Tag?](#why-not-a-regular-a-tag)
- [Link — Basic Usage](#link--basic-usage)
- [Link Props](#link-props)
- [NavLink — Basic Usage](#navlink--basic-usage)
- [NavLink Props](#navlink-props)
- [Example 1: Simple Navigation Bar with Link](#example-1-simple-navigation-bar-with-link)
- [Example 2: Styled Active Links with NavLink](#example-2-styled-active-links-with-navlink)
- [Example 3: Passing State with Link](#example-3-passing-state-with-link)
- [Example 4: Replacing History Instead of Pushing](#example-4-replacing-history-instead-of-pushing)
- [Example 5: NavLink with end Prop](#example-5-navlink-with-end-prop)
- [Example 6: Dynamic Styling with className Function](#example-6-dynamic-styling-with-classname-function)
- [Link vs NavLink vs useNavigate](#link-vs-navlink-vs-usenavigate)

## Prerequisites

Both components come from **React Router v6+**:

```bash
npm install react-router-dom
```

Your app must be wrapped in a `<BrowserRouter>` (or another Router) for these components to work:

```jsx
import { BrowserRouter } from "react-router-dom";

function App() {
  return <BrowserRouter>{/* routes and links go here */}</BrowserRouter>;
}
```

## Why Not a Regular `<a>` Tag?

A plain `<a href="/about">` triggers a **full browser page reload**, which re-downloads the entire app, loses React state, and defeats the purpose of a single-page application. `Link` and `NavLink` intercept the click, update the URL via the History API, and re-render only the parts of the UI that need to change — keeping navigation fast and stateful.

## Link — Basic Usage

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
```

`Link` renders as an `<a>` element under the hood, so it's accessible, supports right-click → "Open in new tab", and works with keyboard navigation out of the box.

## Link Props

| Prop             | Type                 | Description                                                        |
| ---------------- | -------------------- | ------------------------------------------------------------------ |
| `to`             | `string` \| `object` | Destination path, or an object with `pathname`, `search`, `hash`   |
| `replace`        | `boolean`            | Replaces the current history entry instead of pushing a new one    |
| `state`          | `any`                | Data passed to the new route, accessible via `useLocation().state` |
| `reloadDocument` | `boolean`            | Forces a full page reload (like a regular `<a>`) if set to `true`  |

## NavLink — Basic Usage

`NavLink` works exactly like `Link`, but automatically knows whether its `to` path matches the current URL — making it ideal for navigation menus where the active page needs distinct styling.

```jsx
import { NavLink } from "react-router-dom";

function Navbar() {
  return (
    <nav>
      <NavLink
        to="/"
        className={({ isActive }) => (isActive ? "active-link" : "")}
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) => (isActive ? "active-link" : "")}
      >
        About
      </NavLink>
    </nav>
  );
}
```

## NavLink Props

`NavLink` supports everything `Link` does, plus:

| Prop        | Type                   | Description                                                                                           |
| ----------- | ---------------------- | ----------------------------------------------------------------------------------------------------- |
| `className` | `string` \| `function` | Static string, or a function `({ isActive, isPending }) => string`                                    |
| `style`     | `object` \| `function` | Static object, or a function `({ isActive, isPending }) => object`                                    |
| `end`       | `boolean`              | Requires an exact match (no partial/nested match) — see [Example 5](#example-5-navlink-with-end-prop) |
| `children`  | `node` \| `function`   | Can be a function `({ isActive }) => node` for fully custom rendering                                 |

## Example 1: Simple Navigation Bar with Link

```jsx
import { Link } from "react-router-dom";

function Header() {
  return (
    <header>
      <h1>MyApp</h1>
      <nav>
        <Link to="/">Home</Link> | <Link to="/products">Products</Link> |{" "}
        <Link to="/cart">Cart</Link>
      </nav>
    </header>
  );
}
```

## Example 2: Styled Active Links with NavLink

```jsx
import { NavLink } from "react-router-dom";

const linkStyle = {
  marginRight: "1rem",
  textDecoration: "none",
  color: "#333",
};
const activeStyle = { ...linkStyle, color: "#1f3864", fontWeight: "bold" };

function Sidebar() {
  return (
    <aside>
      <NavLink
        to="/dashboard"
        style={({ isActive }) => (isActive ? activeStyle : linkStyle)}
      >
        Dashboard
      </NavLink>
      <NavLink
        to="/settings"
        style={({ isActive }) => (isActive ? activeStyle : linkStyle)}
      >
        Settings
      </NavLink>
    </aside>
  );
}
```

## Example 3: Passing State with Link

Pass data to the destination route without putting it in the URL — useful for things like "came from" context.

```jsx
import { Link, useLocation } from "react-router-dom";

// Sender
function ProductCard({ product }) {
  return (
    <Link
      to={`/products/${product.id}`}
      state={{ fromCategory: product.category }}
    >
      {product.name}
    </Link>
  );
}

// Receiver
function ProductDetail() {
  const location = useLocation();
  const fromCategory = location.state?.fromCategory;

  return (
    <p>{fromCategory ? `Browsing from: ${fromCategory}` : "Direct visit"}</p>
  );
}
```

## Example 4: Replacing History Instead of Pushing

Use `replace` when you don't want the current page to remain in browser history — common after login or a redirect.

```jsx
import { Link } from "react-router-dom";

function LoginSuccessRedirect() {
  return (
    <Link to="/dashboard" replace>
      Continue to Dashboard
    </Link>
  );
}
```

With `replace`, clicking "Back" after landing on `/dashboard` won't return to this redirect page.

## Example 5: NavLink with end Prop

By default, `NavLink` treats a parent route as "active" even when a nested child route is active too. The `end` prop restricts this to an **exact** match.

```jsx
import { NavLink } from "react-router-dom";

function Nav() {
  return (
    <nav>
      {/* Without `end`, this stays active on /dashboard AND /dashboard/settings */}
      <NavLink to="/dashboard">Dashboard</NavLink>

      {/* With `end`, this is active ONLY on exactly /dashboard */}
      <NavLink to="/dashboard" end>
        Dashboard (exact)
      </NavLink>
    </nav>
  );
}
```

This matters most for a root-level link like `"/"`, which would otherwise match every route in the app — always add `end` to a home link.

## Example 6: Dynamic Styling with className Function

Combine `isActive` and `isPending` (useful with data loaders) for richer feedback:

```jsx
import { NavLink } from "react-router-dom";

function NavItem({ to, children }) {
  return (
    <NavLink
      to={to}
      className={({ isActive, isPending }) =>
        [
          "nav-item",
          isActive ? "nav-item--active" : "",
          isPending ? "nav-item--loading" : "",
        ]
          .filter(Boolean)
          .join(" ")
      }
    >
      {children}
    </NavLink>
  );
}
```

```css
.nav-item {
  color: #333;
}
.nav-item--active {
  color: #1f3864;
  font-weight: bold;
}
.nav-item--loading {
  opacity: 0.6;
}
```

## Link vs NavLink vs useNavigate

| Feature         | `Link`                     | `NavLink`                              | `useNavigate`                                               |
| --------------- | -------------------------- | -------------------------------------- | ----------------------------------------------------------- |
| Type            | Component                  | Component                              | Hook                                                        |
| Knows if active | No                         | Yes (`isActive`)                       | No                                                          |
| Use case        | Plain clickable navigation | Nav menus needing active-state styling | Imperative navigation after logic (form submit, auth check) |
| Renders         | `<a>`                      | `<a>`                                  | Nothing — just a function call                              |

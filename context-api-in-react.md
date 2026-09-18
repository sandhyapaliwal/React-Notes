# Context API in React

The **Context API** is React's built-in solution for sharing data across a component tree without manually passing props down through every level — a problem commonly known as **prop drilling**. It lets you define data (theme, authenticated user, language, cart state, etc.) once and read it anywhere it's needed.

## Table of Contents

- [The Problem: Prop Drilling](#the-problem-prop-drilling)
- [Core Concepts](#core-concepts)
- [Basic Usage](#basic-usage)
  - [Step 1: Create the Context](#step-1-create-the-context)
  - [Step 2: Provide the Context](#step-2-provide-the-context)
  - [Step 3: Consume the Context](#step-3-consume-the-context)
- [Example 1: Theme Context](#example-1-theme-context)
- [Example 2: Authentication Context](#example-2-authentication-context)
- [Example 3: Combining Context with useReducer](#example-3-combining-context-with-usereducer)
- [Example 4: Multiple Contexts](#example-4-multiple-contexts)
- [Class Components: Context.Consumer](#class-components-contextconsumer)
- [Context API vs Redux/Zustand](#context-api-vs-reduxzustand)

## The Problem: Prop Drilling

Without Context, sharing data across deeply nested components means passing props through every intermediate component, even ones that don't use the data themselves:

```jsx
function App() {
  const user = { name: "Sandhya" };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />; // Layout doesn't need `user`, just passes it along
}

function Sidebar({ user }) {
  return <UserGreeting user={user} />; // Sidebar doesn't need it either
}

function UserGreeting({ user }) {
  return <p>Hello, {user.name}</p>; // finally used here
}
```

This gets messy fast as the tree grows deeper or more values need to be shared. Context solves this by letting `UserGreeting` read `user` directly, skipping the intermediate components entirely.

## Core Concepts

The Context API has three main pieces:

1. **`createContext()`** — creates a Context object.
2. **`<Context.Provider>`** — a component that supplies a value to all components below it in the tree.
3. **`useContext()`** — a hook that reads the nearest matching Provider's value.

## Basic Usage

### Step 1: Create the Context

```jsx
// UserContext.js
import { createContext } from "react";

export const UserContext = createContext(null);
```

The argument passed to `createContext()` is the **default value**, used only when a component reads the context without any matching Provider above it in the tree.

### Step 2: Provide the Context

```jsx
// App.jsx
import { UserContext } from "./UserContext";
import UserGreeting from "./UserGreeting";

function App() {
  const user = { name: "Sandhya" };

  return (
    <UserContext.Provider value={user}>
      <UserGreeting />
    </UserContext.Provider>
  );
}

export default App;
```

### Step 3: Consume the Context

```jsx
// UserGreeting.jsx
import { useContext } from "react";
import { UserContext } from "./UserContext";

function UserGreeting() {
  const user = useContext(UserContext);

  return <p>Hello, {user.name}</p>;
}

export default UserGreeting;
```

No props needed — `UserGreeting` reads `user` directly from context, no matter how deeply it's nested under `<UserContext.Provider>`.

## Example 1: Theme Context

A classic use case — sharing a light/dark theme across the whole app.

```jsx
// ThemeContext.js
import { createContext, useState, useContext } from "react";

const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  const toggleTheme = () => {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error("useTheme must be used within a ThemeProvider");
  return context;
}
```

```jsx
// App.jsx
import { ThemeProvider } from "./ThemeContext";
import Page from "./Page";

function App() {
  return (
    <ThemeProvider>
      <Page />
    </ThemeProvider>
  );
}
```

```jsx
// Page.jsx
import { useTheme } from "./ThemeContext";

function Page() {
  const { theme, toggleTheme } = useTheme();

  return (
    <div className={theme}>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}
```

## Example 2: Authentication Context

Sharing the logged-in user and auth actions across the app.

```jsx
// AuthContext.js
import { createContext, useContext, useState } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

```jsx
// LoginButton.jsx
import { useAuth } from "./AuthContext";

function LoginButton() {
  const { user, login, logout } = useAuth();

  if (user) {
    return (
      <div>
        <p>Welcome, {user.name}</p>
        <button onClick={logout}>Logout</button>
      </div>
    );
  }

  return <button onClick={() => login({ name: "Sandhya" })}>Login</button>;
}
```

## Example 3: Combining Context with useReducer

For more complex state logic, pairing Context with `useReducer` gives you a lightweight Redux-like pattern without extra libraries.

```jsx
// CartContext.js
import { createContext, useContext, useReducer } from "react";

const CartContext = createContext(null);

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      return { ...state, items: [...state.items, action.payload] };
    case "REMOVE_ITEM":
      return { ...state, items: state.items.filter((item) => item.id !== action.payload) };
    case "CLEAR_CART":
      return { ...state, items: [] };
    default:
      return state;
  }
}

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}

export function useCart() {
  return useContext(CartContext);
}
```

```jsx
// ProductList.jsx
import { useCart } from "./CartContext";

function ProductList() {
  const { dispatch } = useCart();

  return (
    <button onClick={() => dispatch({ type: "ADD_ITEM", payload: { id: 1, name: "T-shirt" } })}>
      Add to Cart
    </button>
  );
}
```

## Example 4: Multiple Contexts

Contexts can be nested and combined freely for different concerns.

```jsx
function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>
          <MainApp />
        </CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

Each provider only wraps the value it's responsible for, keeping concerns separated even though all three are accessible deep in `MainApp`.

## Class Components: Context.Consumer

Before hooks, or in class components (which don't support `useContext`), context is read with a `Context.Consumer`:

```jsx
import { UserContext } from "./UserContext";

class UserGreeting extends React.Component {
  render() {
    return (
      <UserContext.Consumer>
        {(user) => <p>Hello, {user.name}</p>}
      </UserContext.Consumer>
    );
  }
}
```

Function components with `useContext` are simpler and are the recommended approach in modern React.

## Context API vs Redux/Zustand

| Feature | Context API | Redux / Zustand |
|---|---|---|
| Built into React | Yes | No, external library |
| Best for | Low/medium-frequency updates (theme, auth, locale) | Complex, high-frequency, or large-scale app state |
| DevTools | None built-in | Redux DevTools, time-travel debugging |
| Boilerplate | Minimal | Redux: more; Zustand: minimal |
| Performance at scale | Can cause unnecessary re-renders without care | Optimized for frequent updates and large state trees |

Context is a great fit for state that doesn't change often (theme, authenticated user, locale). For state that updates very frequently or is shared across a large, complex app (e.g., a large e-commerce cart with many interacting slices), a dedicated state library often performs and scales better.


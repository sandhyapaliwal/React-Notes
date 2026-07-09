# useContext Hook 

`useContext` is a React Hook that lets you read and subscribe to a **Context** from your component, without having to pass props down manually through every level of the component tree (a problem often called "prop drilling").

## Table of Contents

- [Why useContext?](#why-usecontext)
- [Basic Syntax](#basic-syntax)
- [Step-by-Step Example](#step-by-step-example)
- [Updating Context Values](#updating-context-values)
- [Custom Hook Pattern](#custom-hook-pattern)
- [When to Use / Avoid](#when-to-use--avoid)

---

## Why useContext?

Without Context, data must be passed from a parent component to a child through props — and if the child is deeply nested, every intermediate component must forward that prop, even if it doesn't need it.

```
App
 └─ Layout
     └─ Sidebar
         └─ UserPanel  <-- needs `user` data
```

Passing `user` through `Layout` and `Sidebar` just so `UserPanel` can use it is prop drilling. `useContext` solves this by letting `UserPanel` "reach up" and grab the value directly.

## Basic Syntax

```jsx
import { createContext, useContext } from 'react';

// 1. Create a context (usually in its own file)
const ThemeContext = createContext('light');

// 2. Provide a value somewhere higher in the tree
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

// 3. Consume the value anywhere below the Provider
function Toolbar() {
  const theme = useContext(ThemeContext);
  return <p>Current theme: {theme}</p>;
}
```

## Step-by-Step Example

### 1. Create the Context

```jsx
// ThemeContext.js
import { createContext } from 'react';

export const ThemeContext = createContext('light');
```

### 2. Wrap Components with a Provider

```jsx
// App.jsx
import { useState } from 'react';
import { ThemeContext } from './ThemeContext';
import Dashboard from './Dashboard';

export default function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <Dashboard />
    </ThemeContext.Provider>
  );
}
```

### 3. Consume the Context with useContext

```jsx
// Dashboard.jsx
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

export default function Dashboard() {
  const theme = useContext(ThemeContext);

  return (
    <div className={theme === 'dark' ? 'dark-mode' : 'light-mode'}>
      <p>Welcome to your dashboard!</p>
    </div>
  );
}
```

`Dashboard` never received `theme` as a prop — it read it directly from context, even though it might be several levels deep.

## Updating Context Values

Context isn't limited to static strings. A common pattern is to pass both a value **and** a setter function through the Provider using an object:

```jsx
// AuthContext.js
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (username) => setUser({ name: username });
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

Usage in a component:

```jsx
function LoginButton() {
  const { user, login, logout } = useAuth();

  return user ? (
    <button onClick={logout}>Log out {user.name}</button>
  ) : (
    <button onClick={() => login('Alex')}>Log in</button>
  );
}
```

## Custom Hook Pattern

Wrapping `useContext` in a custom hook (like `useAuth` above) is considered a best practice because it:

- Keeps the context import out of every consuming file
- Lets you add error handling (e.g., throw if used outside a Provider)
- Makes the API cleaner and more reusable

```jsx
export function useAuth() {
  const context = useContext(AuthContext);
  if (context === null) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```



## When to Use / Avoid

**Good use cases:**
- Theme (dark/light mode)
- Authenticated user data
- Language/locale preferences
- App-wide settings that many components need

**Consider alternatives when:**
- Only one or two components need the data (just pass props)
- State updates very frequently and performance matters (consider state management libraries like Redux, Zustand, or Jotai, or split contexts)


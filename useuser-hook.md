# useUser Hook in React

Like `useTheme`, **`useUser` is not a built-in React hook** — it's a pattern for exposing the currently authenticated user (and their loading/auth state) to any component in the tree, usually via the Context API, or a ready-made hook shipped by an authentication provider like **Clerk**, **Firebase**, or **NextAuth.js**.


## Table of Contents

- [Why useUser?](#why-useuser)
- [Building a Custom useUser Hook](#building-a-custom-useuser-hook)
  - [Step 1: Create the User Context](#step-1-create-the-user-context)
  - [Step 2: Create the Provider](#step-2-create-the-provider)
  - [Step 3: Create the useUser Hook](#step-3-create-the-useuser-hook)
  - [Step 4: Wrap Your App](#step-4-wrap-your-app)
  - [Step 5: Consume the User](#step-5-consume-the-user)
- [Example: Protecting a Route](#example-protecting-a-route)
- [Example: Login and Logout](#example-login-and-logout)
- [useUser in Popular Auth Libraries](#useuser-in-popular-auth-libraries)
  - [Clerk](#clerk)
  - [Firebase](#firebase)
  - [NextAuth.js](#nextauthjs)


## Why useUser?

Without a shared auth mechanism, every component that needs to know "who's logged in" would need the user object passed down as a prop, or would need to re-fetch/re-check auth state itself. A `useUser` hook backed by Context solves this by:

- Exposing the current user, loading state, and auth status from **any component**, at any depth.
- Centralizing sign-in/sign-out logic and session handling in one place.
- Keeping UI components decoupled from *how* authentication actually works underneath (JWT, cookies, third-party SDK, etc.).

## Building a Custom useUser Hook

### Step 1: Create the User Context

```jsx
// UserContext.js
import { createContext } from "react";

export const UserContext = createContext(null);
```

### Step 2: Create the Provider

```jsx
// UserProvider.jsx
import { useState, useEffect, useMemo, useCallback } from "react";
import { UserContext } from "./UserContext";

export function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    async function loadUser() {
      try {
        const res = await fetch("/api/me", { credentials: "include" });
        if (res.ok) {
          const data = await res.json();
          setUser(data);
        } else {
          setUser(null);
        }
      } catch (err) {
        setUser(null);
      } finally {
        setIsLoading(false);
      }
    }

    loadUser();
  }, []);

  const login = useCallback(async (credentials) => {
    const res = await fetch("/api/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(credentials),
      credentials: "include",
    });
    if (!res.ok) throw new Error("Login failed");
    const data = await res.json();
    setUser(data);
    return data;
  }, []);

  const logout = useCallback(async () => {
    await fetch("/api/logout", { method: "POST", credentials: "include" });
    setUser(null);
  }, []);

  const value = useMemo(
    () => ({
      user,
      isLoading,
      isSignedIn: !!user,
      login,
      logout,
    }),
    [user, isLoading, login, logout]
  );

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}
```

### Step 3: Create the useUser Hook

```jsx
// useUser.js
import { useContext } from "react";
import { UserContext } from "./UserContext";

export function useUser() {
  const context = useContext(UserContext);

  if (context === null) {
    throw new Error("useUser must be used within a UserProvider");
  }

  return context;
}
```

> Note: since a "logged-out" state is legitimately `user: null`, the check above guards against a missing **Provider**, not a missing **user** — the context object itself always exists once wrapped correctly.

### Step 4: Wrap Your App

```jsx
// App.jsx
import { UserProvider } from "./UserProvider";
import Dashboard from "./Dashboard";

function App() {
  return (
    <UserProvider>
      <Dashboard />
    </UserProvider>
  );
}

export default App;
```

### Step 5: Consume the User

```jsx
// Dashboard.jsx
import { useUser } from "./useUser";

function Dashboard() {
  const { user, isLoading, isSignedIn, logout } = useUser();

  if (isLoading) return <p>Loading...</p>;
  if (!isSignedIn) return <p>Please sign in to continue.</p>;

  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Log out</button>
    </div>
  );
}

export default Dashboard;
```

## Example: Protecting a Route

Combine `useUser` with React Router to redirect unauthenticated users away from protected pages.

```jsx
import { Navigate, useLocation } from "react-router-dom";
import { useUser } from "./useUser";

function ProtectedRoute({ children }) {
  const { isSignedIn, isLoading } = useUser();
  const location = useLocation();

  if (isLoading) return <p>Loading...</p>;

  if (!isSignedIn) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

export default ProtectedRoute;
```

```jsx
<Route
  path="/dashboard"
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

## Example: Login and Logout

```jsx
import { useState } from "react";
import { useUser } from "./useUser";

function LoginForm() {
  const { login } = useUser();
  const [error, setError] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);

    try {
      await login({
        email: formData.get("email"),
        password: formData.get("password"),
      });
    } catch (err) {
      setError("Invalid email or password");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" placeholder="Email" required />
      <input name="password" type="password" placeholder="Password" required />
      <button type="submit">Log In</button>
      {error && <p style={{ color: "red" }}>{error}</p>}
    </form>
  );
}
```

## useUser in Popular Auth Libraries

If your project already uses an auth provider, you likely don't need to build your own — a `useUser` hook is often included out of the box.

### Clerk

```bash
npm install @clerk/clerk-react
```

```jsx
import { useUser } from "@clerk/clerk-react";

function Profile() {
  const { isLoaded, isSignedIn, user } = useUser();

  if (!isLoaded) return <p>Loading...</p>;
  if (!isSignedIn) return <p>Not signed in</p>;

  return <p>Hello, {user.firstName}!</p>;
}
```

Clerk's `useUser` automatically syncs with its `<ClerkProvider>` and gives access to profile data, email addresses, and metadata without any manual context setup.

### Firebase

Firebase doesn't ship a `useUser` hook directly, but the pattern is commonly built on top of `onAuthStateChanged`:

```bash
npm install firebase
```

```jsx
import { useState, useEffect } from "react";
import { onAuthStateChanged } from "firebase/auth";
import { auth } from "./firebaseConfig";

function useUser() {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (firebaseUser) => {
      setUser(firebaseUser);
      setIsLoading(false);
    });
    return unsubscribe;
  }, []);

  return { user, isLoading, isSignedIn: !!user };
}
```

### NextAuth.js

```bash
npm install next-auth
```

```jsx
import { useSession } from "next-auth/react";

function Profile() {
  const { data: session, status } = useSession();

  if (status === "loading") return <p>Loading...</p>;
  if (!session) return <p>Not signed in</p>;

  return <p>Signed in as {session.user.email}</p>;
}
```

NextAuth.js calls its hook `useSession` rather than `useUser`, but it serves the same purpose — exposing the current authenticated user and session status.


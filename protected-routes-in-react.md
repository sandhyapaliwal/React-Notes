# Protected Routes in React

A **Protected Route** (also called a Private Route) is a route that only authenticated (or authorized) users can access. If an unauthenticated user tries to visit it, they're redirected elsewhere — typically to a login page — instead of seeing the protected content.

## Table of Contents

- [Why Protected Routes?](#why-protected-routes)
- [Prerequisites](#prerequisites)
- [Basic Pattern: A ProtectedRoute Wrapper](#basic-pattern-a-protectedroute-wrapper)
- [Example 1: Setting Up Auth Context](#example-1-setting-up-auth-context)
- [Example 2: Creating the ProtectedRoute Component](#example-2-creating-the-protectedroute-component)
- [Example 3: Wiring Up Routes](#example-3-wiring-up-routes)
- [Example 4: Redirecting Back After Login](#example-4-redirecting-back-after-login)
- [Example 5: Role-Based Protected Routes](#example-5-role-based-protected-routes)
- [Example 6: Handling a Loading/Auth-Check State](#example-6-handling-a-loadingauth-check-state)
- [Protecting Multiple Nested Routes at Once](#protecting-multiple-nested-routes-at-once)

## Why Protected Routes?

Without route protection, anyone who knows (or guesses) a URL like `/dashboard` or `/admin` could load that page directly, even without logging in. Protected Routes ensure:

- Only authenticated users can reach sensitive pages (dashboards, account settings, admin panels).
- Unauthenticated visitors are redirected to login, often with a way to return to the page they wanted afterward.
- Role- or permission-based access can be layered on top (e.g., only "admin" users can see `/admin`).

> **Note:** Client-side route protection improves UX, but it is **not a substitute for backend authorization**. Always enforce access control on the server/API too — a determined user can bypass frontend checks.

## Prerequisites

```bash
npm install react-router-dom
```

This guide assumes some form of authentication state (e.g., a logged-in user object, a token in `localStorage`, or a context/store tracking auth status).

## Basic Pattern: A ProtectedRoute Wrapper

The core idea: create a component that checks auth status, and either renders its children/`<Outlet>` or redirects using `<Navigate>`.

```jsx
import { Navigate } from "react-router-dom";

function ProtectedRoute({ isAuthenticated, children }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return children;
}
```

## Example 1: Setting Up Auth Context

A reusable auth context keeps authentication state available throughout the app.

```jsx
// AuthContext.jsx
import {
  createContext,
  useContext,
  useState,
  useMemo,
  useCallback,
} from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null); // null = logged out

  const login = useCallback((userData) => setUser(userData), []);
  const logout = useCallback(() => setUser(null), []);

  const value = useMemo(
    () => ({ user, isAuthenticated: !!user, login, logout }),
    [user, login, logout],
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error("useAuth must be used within an AuthProvider");
  return context;
}
```

## Example 2: Creating the ProtectedRoute Component

```jsx
// ProtectedRoute.jsx
import { Navigate, Outlet, useLocation } from "react-router-dom";
import { useAuth } from "./AuthContext";

function ProtectedRoute() {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    // Save the attempted location so we can return after login
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <Outlet />; // renders the matched nested route
}

export default ProtectedRoute;
```

Using `<Outlet />` (instead of `children`) lets `ProtectedRoute` wrap **multiple nested routes** cleanly — see [Protecting Multiple Nested Routes at Once](#protecting-multiple-nested-routes-at-once).

## Example 3: Wiring Up Routes

```jsx
// App.jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { AuthProvider } from "./AuthContext";
import ProtectedRoute from "./ProtectedRoute";
import Home from "./pages/Home";
import Login from "./pages/Login";
import Dashboard from "./pages/Dashboard";
import Settings from "./pages/Settings";

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />

          {/* Everything inside this parent route requires authentication */}
          <Route element={<ProtectedRoute />}>
            <Route path="/dashboard" element={<Dashboard />} />
            <Route path="/settings" element={<Settings />} />
          </Route>
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  );
}

export default App;
```

## Example 4: Redirecting Back After Login

Send the user to the page they originally wanted, once they log in successfully.

```jsx
// Login.jsx
import { useNavigate, useLocation } from "react-router-dom";
import { useAuth } from "./AuthContext";

function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  const from = location.state?.from?.pathname || "/dashboard";

  const handleLogin = async (credentials) => {
    const userData = await fakeAuthenticate(credentials); // your real auth call
    login(userData);
    navigate(from, { replace: true });
  };

  return <LoginForm onSubmit={handleLogin} />;
}
```

## Example 5: Role-Based Protected Routes

Extend the pattern to restrict access by role or permission, not just login status.

```jsx
// RoleProtectedRoute.jsx
import { Navigate, Outlet } from "react-router-dom";
import { useAuth } from "./AuthContext";

function RoleProtectedRoute({ allowedRoles }) {
  const { user, isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (!allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <Outlet />;
}

export default RoleProtectedRoute;
```

```jsx
// App.jsx (routes excerpt)
<Route element={<RoleProtectedRoute allowedRoles={["admin"]} />}>
  <Route path="/admin" element={<AdminPanel />} />
</Route>
```

## Example 6: Handling a Loading/Auth-Check State

If auth status is determined asynchronously (e.g., verifying a token with the server on app load), show a loading state instead of redirecting prematurely.

```jsx
function ProtectedRoute() {
  const { isAuthenticated, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <p>Checking authentication...</p>;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <Outlet />;
}
```

Without this check, a user who's actually logged in could be briefly (and incorrectly) redirected to `/login` while their auth status is still being verified.

## Protecting Multiple Nested Routes at Once

Because `ProtectedRoute` renders an `<Outlet />`, you can nest an entire section of your app under one guard:

```jsx
<Route element={<ProtectedRoute />}>
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/dashboard/analytics" element={<Analytics />} />
  <Route path="/settings" element={<Settings />} />
  <Route path="/profile" element={<Profile />} />
</Route>
```

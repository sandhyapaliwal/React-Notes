# useRouteError Hook in React Router

`useRouteError` is a hook from **React Router** (v6.4+) that returns the error thrown during route loading, rendering, or an action — for use inside an `errorElement` (or `ErrorBoundary`) component. It's the core building block for handling route-level errors gracefully instead of letting the whole app crash.

## Table of Contents

- [Why useRouteError?](#why-userouteerror)
- [Basic Usage](#basic-usage)
- [Example 1: A Simple Error Boundary](#example-1-a-simple-error-boundary)
- [Example 2: Handling Different Error Types](#example-2-handling-different-error-types)
- [Example 3: Error Boundary for a Loader](#example-3-error-boundary-for-a-loader)
- [Example 4: Nested Route Error Boundaries](#example-4-nested-route-error-boundaries)
- [Example 5: Throwing Custom Errors from a Loader/Action](#example-5-throwing-custom-errors-from-a-loaderaction)
- [isRouteErrorResponse Helper](#isrouteerrorresponse-helper)


## Prerequisites

`useRouteError` requires **React Router v6.4 or later**, and only works with the newer data router APIs (`createBrowserRouter`, `createRoutesFromElements`, etc.) — not the older plain `<BrowserRouter>` + `<Routes>` setup.

```bash
npm install react-router-dom
```

```jsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Home />,
    errorElement: <ErrorPage />,
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

## Why useRouteError?

Without route-level error handling, an error thrown while rendering, loading data, or submitting an action for a route would crash the entire app (or bubble up to a generic top-level error boundary with no route context). `errorElement` + `useRouteError` lets you:

- Catch errors **scoped to a specific route** (or a section of the app), instead of the whole page going blank.
- Show **contextual messages** — e.g., a 404 page for a missing resource vs. a generic error page for a server failure.
- Keep the rest of the UI (navbar, sidebar, etc.) intact even if one route's content fails.

## Basic Usage

```jsx
import { useRouteError } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();
  console.error(error);

  return (
    <div>
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <i>{error.statusText || error.message}</i>
      </p>
    </div>
  );
}
```

This component is assigned to a route's `errorElement`, and React Router automatically renders it whenever that route (or its loader/action) throws.

## Example 1: A Simple Error Boundary

```jsx
// ErrorPage.jsx
import { useRouteError, useNavigate } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();
  const navigate = useNavigate();

  return (
    <div style={{ textAlign: "center", padding: "3rem" }}>
      <h1>Something went wrong</h1>
      <p>{error?.message || "An unexpected error occurred."}</p>
      <button onClick={() => navigate("/")}>Go back home</button>
    </div>
  );
}

export default ErrorPage;
```

```jsx
// router.js
import { createBrowserRouter } from "react-router-dom";
import App from "./App";
import ErrorPage from "./ErrorPage";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
    errorElement: <ErrorPage />,
  },
]);

export default router;
```

## Example 2: Handling Different Error Types

Errors can come from different sources — a thrown `Response` (common with loaders), a JavaScript `Error`, or something custom. Branch your UI accordingly.

```jsx
import { useRouteError, isRouteErrorResponse } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    // Thrown Response, e.g. from a loader: throw new Response("Not Found", { status: 404 })
    return (
      <div>
        <h1>{error.status}</h1>
        <p>{error.statusText}</p>
      </div>
    );
  }

  if (error instanceof Error) {
    return (
      <div>
        <h1>Application Error</h1>
        <p>{error.message}</p>
      </div>
    );
  }

  return <h1>Unknown Error</h1>;
}
```

## Example 3: Error Boundary for a Loader

Loaders that fail (e.g., a failed `fetch`) trigger the nearest `errorElement`.

```jsx
// userLoader.js
export async function userLoader({ params }) {
  const res = await fetch(`/api/users/${params.userId}`);

  if (!res.ok) {
    throw new Response("User not found", { status: res.status });
  }

  return res.json();
}
```

```jsx
// router.js
{
  path: "/users/:userId",
  element: <UserProfile />,
  loader: userLoader,
  errorElement: <ErrorPage />,
}
```

```jsx
// ErrorPage.jsx
import { useRouteError, isRouteErrorResponse } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();

  if (isRouteErrorResponse(error) && error.status === 404) {
    return <h1>User not found</h1>;
  }

  return <h1>Something went wrong loading this user</h1>;
}
```

## Example 4: Nested Route Error Boundaries

Each route can have its own `errorElement`. An error bubbles up to the **nearest** ancestor route that defines one, so you can keep the rest of the layout (e.g., navbar/sidebar) visible while only the failing section shows an error.

```jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <RootErrorPage />, // catches errors with no closer boundary
    children: [
      {
        path: "settings",
        element: <Settings />,
        errorElement: <SettingsErrorPage />, // catches errors here first
        children: [
          { path: "profile", element: <Profile /> },
          { path: "billing", element: <Billing /> },
        ],
      },
    ],
  },
]);
```

If `Billing` throws, `SettingsErrorPage` renders inside `RootLayout` — the navbar/sidebar rendered by `RootLayout` stays visible.

## Example 5: Throwing Custom Errors from a Loader/Action

You aren't limited to `Response` objects — you can throw plain objects or custom `Error` subclasses too.

```jsx
class ValidationError extends Error {
  constructor(message, fields) {
    super(message);
    this.name = "ValidationError";
    this.fields = fields;
  }
}

export async function createUserAction({ request }) {
  const formData = await request.formData();
  const email = formData.get("email");

  if (!email.includes("@")) {
    throw new ValidationError("Invalid email", { email: "Must be a valid email address" });
  }

  // proceed with creating user...
}
```

```jsx
function ErrorPage() {
  const error = useRouteError();

  if (error?.name === "ValidationError") {
    return <p>Form error: {error.fields.email}</p>;
  }

  return <p>{error?.message || "Unexpected error"}</p>;
}
```

## isRouteErrorResponse Helper

`isRouteErrorResponse(error)` is a type-guard helper that tells you whether the error came from a thrown `Response` (the pattern React Router itself uses internally for things like 404s). It's the recommended way to distinguish "expected" HTTP-style errors from unexpected JavaScript errors.

```jsx
import { isRouteErrorResponse, useRouteError } from "react-router-dom";

function ErrorPage() {
  const error = useRouteError();

  return isRouteErrorResponse(error) ? (
    <h1>{error.status} {error.statusText}</h1>
  ) : (
    <h1>Unexpected Error</h1>
  );
}
```


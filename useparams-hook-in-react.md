# useParams Hook in React Router

`useParams` is a hook from **React Router** (v6+) that returns an object of key/value pairs of the dynamic parameters (URL params) matched by the current route. It's the standard way to read values like IDs or slugs directly from the URL inside a component.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Basic Usage](#basic-usage)
- [Example 1: Single Dynamic Param](#example-1-single-dynamic-param)
- [Example 2: Multiple Dynamic Params](#example-2-multiple-dynamic-params)
- [Example 3: Fetching Data Based on a Param](#example-3-fetching-data-based-on-a-param)
- [Example 4: Optional Params](#example-4-optional-params)
- [Example 5: Nested Routes and Params](#example-5-nested-routes-and-params)
- [Example 6: Wildcard / Splat Params](#example-6-wildcard--splat-params)
- [useParams vs useSearchParams](#useparams-vs-usesearchparams)

## Prerequisites

`useParams` requires **React Router v6 or later**:

```bash
npm install react-router-dom
```

Params only work if the route path actually defines a dynamic segment using a colon (`:paramName`):

```jsx
import { Routes, Route } from "react-router-dom";

function App() {
  return (
    <Routes>
      <Route path="/users/:userId" element={<UserProfile />} />
    </Routes>
  );
}
```

## Basic Usage

```jsx
import { useParams } from "react-router-dom";

function UserProfile() {
  const params = useParams();
  // If the URL is /users/42, params = { userId: "42" }

  return <h1>User ID: {params.userId}</h1>;
}
```

All values returned by `useParams` are **strings**, even if they look numeric — you'll need to convert them yourself (e.g., `Number(params.userId)`) if you need a number.

## Example 1: Single Dynamic Param

```jsx
// Route definition
<Route path="/products/:productId" element={<ProductDetail />} />
```

```jsx
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { productId } = useParams();

  return <p>Showing details for product #{productId}</p>;
}
```

Visiting `/products/101` renders: `Showing details for product #101`.

## Example 2: Multiple Dynamic Params

```jsx
// Route definition
<Route path="/blog/:category/:postId" element={<BlogPost />} />
```

```jsx
import { useParams } from "react-router-dom";

function BlogPost() {
  const { category, postId } = useParams();

  return (
    <div>
      <p>Category: {category}</p>
      <p>Post ID: {postId}</p>
    </div>
  );
}
```

Visiting `/blog/react/55` gives `category = "react"` and `postId = "55"`.

## Example 3: Fetching Data Based on a Param

A very common pattern — using the param to fetch the matching resource from an API.

```jsx
import { useState, useEffect } from "react";
import { useParams } from "react-router-dom";

function UserProfile() {
  const { userId } = useParams();
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let isCancelled = false;

    async function fetchUser() {
      setLoading(true);
      const res = await fetch(`/api/users/${userId}`);
      const data = await res.json();
      if (!isCancelled) {
        setUser(data);
        setLoading(false);
      }
    }

    fetchUser();

    return () => {
      isCancelled = true;
    };
  }, [userId]);

  if (loading) return <p>Loading...</p>;

  return <h1>{user.name}</h1>;
}
```

Note the `userId` in the dependency array — if the user navigates from `/users/1` to `/users/2` while staying on the same component, the effect re-runs automatically.

## Example 4: Optional Params

React Router v6.5+ supports optional dynamic segments with a trailing `?`:

```jsx
<Route path="/products/:category?/:productId" element={<Product />} />
```

```jsx
function Product() {
  const { category, productId } = useParams();
  // category may be undefined if the URL omits that segment

  return (
    <div>
      {category && <p>Category: {category}</p>}
      <p>Product: {productId}</p>
    </div>
  );
}
```

## Example 5: Nested Routes and Params

Params from parent routes remain accessible in nested child routes.

```jsx
<Route path="/teams/:teamId" element={<TeamLayout />}>
  <Route path="members/:memberId" element={<MemberDetail />} />
</Route>
```

```jsx
function MemberDetail() {
  const { teamId, memberId } = useParams();
  // Both are available here, even though teamId is defined on the parent route

  return (
    <p>
      Team {teamId}, Member {memberId}
    </p>
  );
}
```

## Example 6: Wildcard / Splat Params

A trailing `*` in a route path captures everything after it, accessible via `params["*"]`.

```jsx
<Route path="/files/*" element={<FileViewer />} />
```

```jsx
function FileViewer() {
  const params = useParams();
  const filePath = params["*"];
  // Visiting /files/docs/2024/report.pdf → filePath = "docs/2024/report.pdf"

  return <p>Viewing: {filePath}</p>;
}
```

## useParams vs useSearchParams

These are often confused, but they read different parts of the URL:

| Feature | `useParams` | `useSearchParams` |
|---|---|---|
| Reads | Path segments defined in the route (`:id`) | Query string (`?key=value`) |
| URL example | `/users/42` → `{ userId: "42" }` | `/users?sort=asc` → `sort=asc` |
| Route definition needed | Yes — path must declare `:param` | No — works on any route |
| Common use | Identifying a specific resource (ID, slug) | Filters, sorting, pagination, search terms |

```jsx
import { useParams, useSearchParams } from "react-router-dom";

function SearchResults() {
  const { category } = useParams();       // from /shop/:category
  const [searchParams] = useSearchParams(); // from ?q=shoes&page=2

  const query = searchParams.get("q");
  const page = searchParams.get("page");

  return (
    <p>
      Category: {category}, Query: {query}, Page: {page}
    </p>
  );
}
```


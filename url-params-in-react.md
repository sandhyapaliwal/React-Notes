# URL Params in React

URL Params are dynamic values embedded in a route's URL that let a single component render different content based on what's in the address bar — e.g., `/products/42` or `/blog/react-hooks`. In React apps, this is handled through **React Router**, and comes in two distinct flavors: **path params** and **query (search) params**.

## Table of Contents

- [What Are URL Params?](#what-are-url-params)
- [Path Params vs Query Params](#path-params-vs-query-params)
- [Setting Up React Router](#setting-up-react-router)
- [Path Params with useParams](#path-params-with-useparams)
  - [Single Param](#single-param)
  - [Multiple Params](#multiple-params)
  - [Optional Params](#optional-params)
- [Query Params with useSearchParams](#query-params-with-usesearchparams)
  - [Reading Query Params](#reading-query-params)
  - [Updating Query Params](#updating-query-params)
  - [Multiple Values for One Key](#multiple-values-for-one-key)
- [Example: Combining Both](#example-combining-both)
- [Example: Fetching Data Based on a Param](#example-fetching-data-based-on-a-param)
- [Wildcard / Splat Params](#wildcard--splat-params)

## What Are URL Params?

Instead of building a separate route and component for every single product, user, or blog post, URL params let one route template handle them all:

```
/products/1
/products/2
/products/87
```

All three URLs can be matched by a single route definition — `/products/:productId` — with the component reading `1`, `2`, or `87` from the URL at render time.

## Path Params vs Query Params

| Type            | Example URL                   | Defined in Route?         | Read With           | Typical Use                                |
| --------------- | ----------------------------- | ------------------------- | ------------------- | ------------------------------------------ |
| **Path param**  | `/users/42`                   | Yes — `path="/users/:id"` | `useParams()`       | Identifying a specific resource (ID, slug) |
| **Query param** | `/search?q=react&sort=newest` | No — works on any route   | `useSearchParams()` | Filters, sorting, pagination, search text  |

Path params are part of the route's structure; query params are optional key-value pairs appended after a `?`.

## Setting Up React Router

```bash
npm install react-router-dom
```

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/products/:productId" element={<ProductDetail />} />
        <Route path="/search" element={<SearchResults />} />
      </Routes>
    </BrowserRouter>
  );
}
```

## Path Params with useParams

### Single Param

```jsx
// Route: <Route path="/products/:productId" element={<ProductDetail />} />
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { productId } = useParams();
  return <p>Showing product #{productId}</p>;
}
```

Visiting `/products/101` renders: `Showing product #101`.

> All values from `useParams` are **strings** — convert with `Number(productId)` if you need a number.

### Multiple Params

```jsx
// Route: <Route path="/blog/:category/:postId" element={<BlogPost />} />
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

### Optional Params

React Router v6.5+ supports optional segments with a trailing `?`:

```jsx
// Route: <Route path="/products/:category?/:productId" element={<Product />} />
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

## Query Params with useSearchParams

### Reading Query Params

```jsx
import { useSearchParams } from "react-router-dom";

function SearchResults() {
  const [searchParams] = useSearchParams();

  const query = searchParams.get("q"); // "react"
  const sort = searchParams.get("sort"); // "newest"

  return (
    <p>
      Results for "{query}", sorted by {sort}
    </p>
  );
}
```

Visiting `/search?q=react&sort=newest` populates `query` and `sort` accordingly.

### Updating Query Params

```jsx
import { useSearchParams } from "react-router-dom";

function SortDropdown() {
  const [searchParams, setSearchParams] = useSearchParams();

  const handleChange = (e) => {
    setSearchParams({ sort: e.target.value });
  };

  return (
    <select value={searchParams.get("sort") || ""} onChange={handleChange}>
      <option value="newest">Newest</option>
      <option value="price-asc">Price: Low to High</option>
      <option value="price-desc">Price: High to Low</option>
    </select>
  );
}
```

Calling `setSearchParams` updates the URL (and triggers a re-render) without a full page reload.

### Multiple Values for One Key

```jsx
import { useSearchParams } from "react-router-dom";

function TagFilter() {
  const [searchParams] = useSearchParams();
  const tags = searchParams.getAll("tag"); // for /items?tag=react&tag=hooks

  return (
    <ul>
      {tags.map((tag) => (
        <li key={tag}>{tag}</li>
      ))}
    </ul>
  );
}
```

## Example: Combining Both

A product listing page that filters by category (path param) and sorts/searches via query params.

```jsx
// Route: <Route path="/category/:categoryName" element={<CategoryPage />} />
import { useParams, useSearchParams } from "react-router-dom";

function CategoryPage() {
  const { categoryName } = useParams();
  const [searchParams] = useSearchParams();

  const sort = searchParams.get("sort") || "default";
  const page = Number(searchParams.get("page")) || 1;

  return (
    <div>
      <h1>Category: {categoryName}</h1>
      <p>
        Sort: {sort} | Page: {page}
      </p>
    </div>
  );
}
```

Visiting `/category/shoes?sort=price-asc&page=2` gives `categoryName = "shoes"`, `sort = "price-asc"`, `page = 2`.

## Example: Fetching Data Based on a Param

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
  }, [userId]); // re-runs automatically if userId changes

  if (loading) return <p>Loading...</p>;
  return <h1>{user.name}</h1>;
}
```

## Wildcard / Splat Params

A trailing `*` in a route path captures everything after it:

```jsx
// Route: <Route path="/files/*" element={<FileViewer />} />
import { useParams } from "react-router-dom";

function FileViewer() {
  const params = useParams();
  const filePath = params["*"];
  // /files/docs/2024/report.pdf -> filePath = "docs/2024/report.pdf"
  return <p>Viewing: {filePath}</p>;
}
```

# Dynamic Routes in React (React Router)

Dynamic routes let a single route definition match many URLs by using **placeholder segments** (like `:userId` or `:slug`) instead of a fixed path. This is how apps render pages like `/products/42` or `/blog/my-first-post` without writing a separate route for every possible value.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Static vs Dynamic Routes](#static-vs-dynamic-routes)
- [Defining a Dynamic Route](#defining-a-dynamic-route)
- [Example 1: Single Dynamic Segment](#example-1-single-dynamic-segment)
- [Example 2: Multiple Dynamic Segments](#example-2-multiple-dynamic-segments)
- [Example 3: Fetching Data for a Dynamic Route](#example-3-fetching-data-for-a-dynamic-route)
- [Example 4: Nested Dynamic Routes](#example-4-nested-dynamic-routes)
- [Example 5: Optional Segments](#example-5-optional-segments)
- [Example 6: Catch-All (Wildcard) Routes](#example-6-catch-all-wildcard-routes)
- [Linking to Dynamic Routes](#linking-to-dynamic-routes)
- [Handling 404s for Invalid Dynamic Routes](#handling-404s-for-invalid-dynamic-routes)

## Prerequisites

Dynamic routing in React is typically handled by **React Router** (v6+):

```bash
npm install react-router-dom
```

Wrap your app in a Router so routing works:

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>{/* routes go here */}</Routes>
    </BrowserRouter>
  );
}
```

## Static vs Dynamic Routes

| Type    | Path Example           | Matches                                             |
| ------- | ---------------------- | --------------------------------------------------- |
| Static  | `/about`               | Only `/about`                                       |
| Dynamic | `/products/:productId` | `/products/1`, `/products/42`, `/products/anything` |

A **static route** has a fixed, unchanging path. A **dynamic route** contains one or more segments prefixed with `:` that act as variables, capturing whatever value appears in that position of the URL.

## Defining a Dynamic Route

A segment becomes dynamic by prefixing it with a colon in the `path` prop:

```jsx
<Route path="/products/:productId" element={<ProductDetail />} />
```

Inside `ProductDetail`, the value of that segment is read using the `useParams` hook:

```jsx
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { productId } = useParams();
  return <h1>Product ID: {productId}</h1>;
}
```

Visiting `/products/101` renders `Product ID: 101`. Visiting `/products/anything-else` renders that value instead — one route definition handles all of them.

## Example 1: Single Dynamic Segment

```jsx
<Routes>
  <Route path="/users/:username" element={<UserProfile />} />
</Routes>
```

```jsx
import { useParams } from "react-router-dom";

function UserProfile() {
  const { username } = useParams();
  return <h2>Welcome to {username}'s profile</h2>;
}
```

## Example 2: Multiple Dynamic Segments

A route can contain more than one dynamic segment.

```jsx
<Route path="/blog/:category/:postId" element={<BlogPost />} />
```

```jsx
import { useParams } from "react-router-dom";

function BlogPost() {
  const { category, postId } = useParams();

  return (
    <article>
      <p>Category: {category}</p>
      <p>Post ID: {postId}</p>
    </article>
  );
}
```

Visiting `/blog/react/55` gives `category = "react"` and `postId = "55"`.

## Example 3: Fetching Data for a Dynamic Route

The most common real-world use — using the dynamic segment to fetch the matching resource from an API.

```jsx
import { useState, useEffect } from "react";
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { productId } = useParams();
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let isCancelled = false;

    async function fetchProduct() {
      setLoading(true);
      const res = await fetch(`/api/products/${productId}`);
      const data = await res.json();
      if (!isCancelled) {
        setProduct(data);
        setLoading(false);
      }
    }

    fetchProduct();

    return () => {
      isCancelled = true;
    };
  }, [productId]); // re-fetches automatically when the URL param changes

  if (loading) return <p>Loading...</p>;

  return <h1>{product.name}</h1>;
}
```

## Example 4: Nested Dynamic Routes

Dynamic segments from a parent route remain available to nested child routes.

```jsx
<Route path="/teams/:teamId" element={<TeamLayout />}>
  <Route path="members/:memberId" element={<MemberDetail />} />
</Route>
```

```jsx
import { useParams, Outlet } from "react-router-dom";

function TeamLayout() {
  const { teamId } = useParams();
  return (
    <div>
      <h1>Team {teamId}</h1>
      <Outlet /> {/* renders MemberDetail when the nested path matches */}
    </div>
  );
}

function MemberDetail() {
  const { teamId, memberId } = useParams(); // both are accessible here
  return (
    <p>
      Team {teamId}, Member {memberId}
    </p>
  );
}
```

## Example 5: Optional Segments

React Router v6.5+ supports marking a dynamic segment as optional with a trailing `?`:

```jsx
<Route path="/shop/:category?/:productId" element={<ShopItem />} />
```

```jsx
function ShopItem() {
  const { category, productId } = useParams();
  // category may be undefined if the URL omits it, e.g. /shop/101

  return (
    <div>
      {category && <p>Category: {category}</p>}
      <p>Product: {productId}</p>
    </div>
  );
}
```

## Example 6: Catch-All (Wildcard) Routes

A trailing `*` captures everything after a given point in the URL — useful for file paths, nested unknown structures, or custom 404 handling.

```jsx
<Route path="/docs/*" element={<DocsViewer />} />
```

```jsx
import { useParams } from "react-router-dom";

function DocsViewer() {
  const params = useParams();
  const restOfPath = params["*"];
  // Visiting /docs/guides/setup/install → restOfPath = "guides/setup/install"

  return <p>Viewing doc: {restOfPath}</p>;
}
```

## Linking to Dynamic Routes

Use template literals with `<Link>` (or `navigate()`) to build URLs that include the dynamic value:

```jsx
import { Link } from "react-router-dom";

function ProductCard({ id, name }) {
  return <Link to={`/products/${id}`}>{name}</Link>;
}
```

```jsx
import { useNavigate } from "react-router-dom";

function BuyButton({ productId }) {
  const navigate = useNavigate();
  return (
    <button onClick={() => navigate(`/products/${productId}`)}>Buy Now</button>
  );
}
```

## Handling 404s for Invalid Dynamic Routes

A dynamic route matches _any_ value in that URL position — including ones that don't correspond to real data. Always handle the "not found" case explicitly:

```jsx
function ProductDetail() {
  const { productId } = useParams();
  const [product, setProduct] = useState(null);
  const [notFound, setNotFound] = useState(false);

  useEffect(() => {
    fetch(`/api/products/${productId}`)
      .then((res) => {
        if (res.status === 404) {
          setNotFound(true);
          return null;
        }
        return res.json();
      })
      .then((data) => data && setProduct(data));
  }, [productId]);

  if (notFound) return <p>Product not found.</p>;
  if (!product) return <p>Loading...</p>;

  return <h1>{product.name}</h1>;
}
```

A catch-all `<Route path="*" element={<NotFoundPage />} />` at the end of your `<Routes>` also handles URLs that don't match any defined route at all.

# useNavigate Hook in React Router

`useNavigate` is a hook from **React Router** (v6+) that returns a function letting you programmatically navigate to a different route — for example, after a form submission, a login, or a button click — instead of relying only on `<Link>` components.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Basic Usage](#basic-usage)
- [Navigation Options](#navigation-options)
- [Example 1: Navigate on Button Click](#example-1-navigate-on-button-click)
- [Example 2: Navigate After Form Submission](#example-2-navigate-after-form-submission)
- [Example 3: Redirect After Login](#example-3-redirect-after-login)
- [Example 4: Go Back / Go Forward](#example-4-go-back--go-forward)
- [Example 5: Passing State Between Routes](#example-5-passing-state-between-routes)


## Prerequisites

`useNavigate` requires **React Router v6 or later**:

```bash
npm install react-router-dom
```

Your app must be wrapped in a `<BrowserRouter>` (or another Router) for the hook to work:

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
import { useNavigate } from "react-router-dom";

function MyComponent() {
  const navigate = useNavigate();

  const handleClick = () => {
    navigate("/dashboard");
  };

  return <button onClick={handleClick}>Go to Dashboard</button>;
}
```

Calling `navigate("/path")` pushes a new entry onto the history stack, similar to clicking a `<Link to="/path">`.

## Navigation Options

`navigate()` accepts a second argument for extra control:

```jsx
navigate("/dashboard", {
  replace: true,   // replaces current history entry instead of pushing a new one
  state: { from: "settings" }, // pass data to the next route
});
```

You can also pass a **number** to move through history, similar to `window.history.go()`:

```jsx
navigate(-1); // go back one page
navigate(1);  // go forward one page
```

## Example 1: Navigate on Button Click

```jsx
import { useNavigate } from "react-router-dom";

function ProductCard({ productId }) {
  const navigate = useNavigate();

  return (
    <div onClick={() => navigate(`/products/${productId}`)}>
      View Product Details
    </div>
  );
}
```

## Example 2: Navigate After Form Submission

```jsx
import { useState } from "react";
import { useNavigate } from "react-router-dom";

function ContactForm() {
  const [submitted, setSubmitted] = useState(false);
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    // simulate API call
    await new Promise((resolve) => setTimeout(resolve, 500));
    setSubmitted(true);
    navigate("/thank-you");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="Your name" required />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Example 3: Redirect After Login

A very common use case — redirecting the user after successful authentication, including returning them to the page they originally tried to visit.

```jsx
import { useNavigate, useLocation } from "react-router-dom";

function Login() {
  const navigate = useNavigate();
  const location = useLocation();

  const from = location.state?.from?.pathname || "/dashboard";

  const handleLogin = async (credentials) => {
    const success = await fakeAuth(credentials);
    if (success) {
      navigate(from, { replace: true });
    }
  };

  return <LoginForm onSubmit={handleLogin} />;
}
```

Using `{ replace: true }` here prevents the user from hitting the "back" button and landing on the login page again.

## Example 4: Go Back / Go Forward

```jsx
import { useNavigate } from "react-router-dom";

function BackButton() {
  const navigate = useNavigate();

  return <button onClick={() => navigate(-1)}>← Go Back</button>;
}
```

## Example 5: Passing State Between Routes

You can pass data along with navigation without putting it in the URL:

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


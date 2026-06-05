# ⚛️ Conditional Rendering 


## 📚 Table of Contents

- [What is Conditional Rendering?](#what-is-conditional-rendering)
- [1. if / else Statement](#1-if--else-statement)
- [2. Ternary Operator](#2-ternary-operator)
- [3. Logical AND (&&) Operator](#3-logical-and--operator)
- [4. Logical OR (||) Operator](#4-logical-or--operator)
- [5. Nullish Coalescing (??) Operator](#5-nullish-coalescing--operator)
- [6. switch Statement](#6-switch-statement)
- [7. Immediately Invoked Function Expression (IIFE)](#7-immediately-invoked-function-expression-iife)
- [8. Enum / Object Map Pattern](#8-enum--object-map-pattern)
- [9. Optional Chaining with Conditional Rendering](#9-optional-chaining-with-conditional-rendering)
- [10. Higher-Order Component (HOC) Pattern](#10-higher-order-component-hoc-pattern)


---

## What is Conditional Rendering?

In React, **Conditional Rendering** means showing or hiding UI elements based on certain conditions — just like `if` statements in JavaScript, but inside your JSX.

> React does not have a special directive like `v-if` (Vue) or `*ngIf` (Angular).  
> Instead, it uses **plain JavaScript logic** inside JSX.

---

## 1. `if` / `else` Statement

The most basic and readable approach. Best used **outside** the return statement.

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back! 👋</h1>;
  } else {
    return <h1>Please sign in.</h1>;
  }
}
```

### With a variable inside the component:

```jsx
function Dashboard({ isAdmin }) {
  let content;

  if (isAdmin) {
    content = <AdminPanel />;
  } else {
    content = <UserPanel />;
  }

  return <div className="dashboard">{content}</div>;
}
```

✅ **When to use:** When the logic is complex or involves multiple conditions.

---

## 2. Ternary Operator

Short and inline. Best for **simple two-way** conditions inside JSX.

**Syntax:** `condition ? <ComponentA /> : <ComponentB />`

```jsx
function UserStatus({ isOnline }) {
  return (
    <span>
      {isOnline ? "🟢 Online" : "🔴 Offline"}
    </span>
  );
}
```

### Nested Ternary (use sparingly):

```jsx
function TrafficLight({ color }) {
  return (
    <p>
      {color === "green"
        ? "Go 🟢"
        : color === "yellow"
        ? "Slow Down 🟡"
        : "Stop 🔴"}
    </p>
  );
}
```

> ⚠️ Avoid deeply nested ternaries — they hurt readability. Use `if/else` or a `switch` instead.

✅ **When to use:** Simple inline two-option rendering.

---

## 3. Logical AND (`&&`) Operator

Renders something **only when the condition is true**. Nothing is rendered if the condition is false.

**Syntax:** `condition && <Component />`

```jsx
function Notifications({ hasMessages }) {
  return (
    <div>
      <h1>Inbox</h1>
      {hasMessages && <p>You have new messages! 📬</p>}
    </div>
  );
}
```

### ⚠️ Gotcha — Falsy Number `0`:

```jsx
// ❌ BUG: This renders "0" on screen when count is 0
{count && <Badge count={count} />}

// ✅ FIX: Convert to boolean
{count > 0 && <Badge count={count} />}
// or
{!!count && <Badge count={count} />}
```

✅ **When to use:** Show something only when a condition is true (no else needed).

---

## 4. Logical OR (`||`) Operator

Renders a **fallback** when the left side is falsy.

```jsx
function UserName({ name }) {
  return <p>{name || "Anonymous User"}</p>;
}
```

```jsx
function Avatar({ imageUrl }) {
  return <img src={imageUrl || "/default-avatar.png"} alt="avatar" />;
}
```

✅ **When to use:** Providing default/fallback values.

---

## 5. Nullish Coalescing (`??`) Operator

Similar to `||`, but only falls back for `null` or `undefined` — not for `0`, `""`, or `false`.

```jsx
function Score({ points }) {
  return <p>Score: {points ?? "Not available"}</p>;
}

// points = 0  → "Score: 0"       ✅ (0 is a valid value)
// points = null → "Score: Not available" ✅
```

✅ **When to use:** When `0`, `false`, or `""` are valid values that should NOT trigger the fallback.

---

## 6. `switch` Statement

Best for **multiple conditions** based on a single variable.

```jsx
function StatusBadge({ status }) {
  switch (status) {
    case "loading":
      return <span>⏳ Loading...</span>;
    case "success":
      return <span>✅ Success</span>;
    case "error":
      return <span>❌ Error occurred</span>;
    default:
      return <span>❓ Unknown status</span>;
  }
}
```

✅ **When to use:** Multiple distinct states/roles/modes.

---

## 7. Immediately Invoked Function Expression (IIFE)

Lets you write `if/else` or `switch` logic **inline** inside JSX.

```jsx
function App({ userRole }) {
  return (
    <div>
      {(() => {
        if (userRole === "admin") return <AdminDashboard />;
        if (userRole === "editor") return <EditorView />;
        return <GuestView />;
      })()}
    </div>
  );
}
```

✅ **When to use:** Complex inline logic when you can't easily extract to a separate function.

---

## 8. Enum / Object Map Pattern

A clean, scalable alternative to `switch` for rendering different components.

```jsx
const VIEWS = {
  home: <HomePage />,
  about: <AboutPage />,
  contact: <ContactPage />,
};

function App({ currentView }) {
  return <div>{VIEWS[currentView] || <NotFound />}</div>;
}
```

### With dynamic components:

```jsx
const ROLE_COMPONENTS = {
  admin: AdminPanel,
  editor: EditorPanel,
  viewer: ViewerPanel,
};

function Dashboard({ role }) {
  const Component = ROLE_COMPONENTS[role] || DefaultPanel;
  return <Component />;
}
```

✅ **When to use:** Many possible values/states that map to different components.

---

## 9. Optional Chaining with Conditional Rendering

Safely access deeply nested data before rendering.

```jsx
function UserProfile({ user }) {
  return (
    <div>
      <p>{user?.name ?? "Guest"}</p>
      <p>{user?.address?.city ?? "City not provided"}</p>
      {user?.isPremium && <PremiumBadge />}
    </div>
  );
}
```

✅ **When to use:** When data may be `null` or `undefined` (e.g., from an API).

---

## 10. Higher-Order Component (HOC) Pattern

Wrap a component to conditionally render it based on some prop.

```jsx
function withAuth(WrappedComponent) {
  return function AuthGuard({ isAuthenticated, ...props }) {
    if (!isAuthenticated) {
      return <p>🔒 Access Denied. Please log in.</p>;
    }
    return <WrappedComponent {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);

function App() {
  return <ProtectedDashboard isAuthenticated={true} />;
}
```

✅ **When to use:** Reusable auth/permission guards across multiple components.

---

## Common Mistakes to Avoid

| ❌ Mistake | ✅ Fix |
|---|---|
| `{0 && <Component />}` renders `0` | Use `{count > 0 && <Component />}` |
| Deeply nested ternaries | Use `if/else`, `switch`, or object map |
| Forgetting `null` return | Return `null` to render nothing |
| Using `||` when `0` is valid | Use `??` (nullish coalescing) instead |

### Returning `null` to Render Nothing:

```jsx
function Alert({ show, message }) {
  if (!show) return null; // renders nothing

  return <div className="alert">{message}</div>;
}
```

---

## Quick Reference Cheat Sheet

```
Condition Type             | Best Approach
---------------------------|---------------------------
Simple true/false          | Ternary ? : or &&
Only show if true          | && operator
Fallback/default value     | || or ??
Multiple cases             | switch or Object Map
Complex logic              | if/else outside JSX
Reusable auth guard        | HOC Pattern
Inline multi-branch        | IIFE
```

---


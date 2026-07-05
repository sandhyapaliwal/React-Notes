# Controlled vs Uncontrolled Components 

When working with forms in React, you'll come across two main approaches for handling input elements: **Controlled Components** and **Uncontrolled Components**. Understanding the difference is essential for managing form state effectively.

## Table of Contents

- [Controlled Components](#controlled-components)
- [Uncontrolled Components](#uncontrolled-components)
- [Key Differences](#key-differences)
- [When to Use Which](#when-to-use-which)
- [Code Examples](#code-examples)

---

## Controlled Components

In a **controlled component**, the form data is handled by the React component's state. The input element's value is always driven by state, and any change goes through a state update via an `onChange` handler.

### Characteristics
- The value of the input is bound to a state variable (`value={state}`).
- Every keystroke triggers an `onChange` event, which updates the state.
- React is the "single source of truth" for the input's value.
- Easier to validate, format, or conditionally control input in real time.

### Example

```jsx
import { useState } from "react";

function ControlledInput() {
  const [name, setName] = useState("");

  const handleChange = (e) => {
    setName(e.target.value);
  };

  return (
    <div>
      <input type="text" value={name} onChange={handleChange} />
      <p>Your name: {name}</p>
    </div>
  );
}

export default ControlledInput;
```

---

## Uncontrolled Components

In an **uncontrolled component**, the form data is handled by the DOM itself, not by React state. You typically use a `ref` to access the input's value only when you need it (e.g., on form submit).

### Characteristics
- The input maintains its own internal state in the DOM.
- React does not track every change; it only reads the value when required.
- Uses `ref` (via `useRef`) instead of `useState`.
- Closer to traditional HTML form behavior.

### Example

```jsx
import { useRef } from "react";

function UncontrolledInput() {
  const nameRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Your name: ${nameRef.current.value}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" ref={nameRef} defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}

export default UncontrolledInput;
```

---

## Key Differences

| Aspect | Controlled Component | Uncontrolled Component |
|---|---|---|
| Data source | React state | DOM itself |
| Value access | `value` prop + `onChange` | `ref` |
| Initial value | `value={state}` | `defaultValue` |
| Real-time validation | Easy | Harder (requires manual access) |
| Code amount | Slightly more boilerplate | Less boilerplate |
| Performance (large forms) | Can re-render on every keystroke | Fewer re-renders |
| Predictability | High (React fully controls state) | Lower (DOM manages state) |

---

## When to Use Which

**Use Controlled Components when:**
- You need instant validation or formatting (e.g., disabling a submit button until valid).
- You want to conditionally enable/disable fields based on other input values.
- You need to enforce specific input formats as the user types.
- You want a predictable, single source of truth for your form data.

**Use Uncontrolled Components when:**
- You're integrating with non-React code or a third-party DOM library.
- You have simple forms where you only need the value on submit.
- You want to avoid unnecessary re-renders for performance reasons.
- You're quickly prototyping and don't need tight control over input behavior.

---

## Code Examples

### Controlled Form with Multiple Fields

```jsx
import { useState } from "react";

function ControlledForm() {
  const [formData, setFormData] = useState({ username: "", email: "" });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={formData.username}
        onChange={handleChange}
        placeholder="Username"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

export default ControlledForm;
```

### Uncontrolled Form with Multiple Fields

```jsx
import { useRef } from "react";

function UncontrolledForm() {
  const usernameRef = useRef(null);
  const emailRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({
      username: usernameRef.current.value,
      email: emailRef.current.value,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={usernameRef} placeholder="Username" />
      <input ref={emailRef} placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}

export default UncontrolledForm;
```

---

## Summary

- **Controlled components** rely on React state as the single source of truth, offering more control and predictability at the cost of extra boilerplate.
- **Uncontrolled components** let the DOM manage form state, which is simpler and can be more performant for basic use cases, but offers less control.
- Most modern React applications favor controlled components for complex forms, while uncontrolled components are useful for simple or performance-sensitive cases.


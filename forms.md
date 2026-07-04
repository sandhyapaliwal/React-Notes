# Forms 

Forms are one of the most common ways users interact with a web application — login pages, sign-up pages, search bars, surveys, and more. In React, forms work a bit differently than in plain HTML because React likes to keep the UI in sync with a single source of truth: **state**.

This guide covers the core concepts of building forms in React, from basics to best practices.

---

## Table of Contents

1. [Controlled vs Uncontrolled Components](#controlled-vs-uncontrolled-components)
2. [Basic Controlled Form](#basic-controlled-form)
3. [Handling Multiple Inputs](#handling-multiple-inputs)
4. [Handling Different Input Types](#handling-different-input-types)
5. [Form Validation](#form-validation)
6. [Uncontrolled Forms with useRef](#uncontrolled-forms-with-useref)
7. [Using Third-Party Libraries](#using-third-party-libraries)

---

## Controlled vs Uncontrolled Components

There are two ways to handle form inputs in React:

### Controlled Components
The form data is handled by React state. The input's value is always driven by state, and any change updates that state via an event handler.

```jsx
<input value={name} onChange={(e) => setName(e.target.value)} />
```

**Pros:** Predictable, easy to validate, easy to reset, single source of truth.

### Uncontrolled Components
The form data is handled by the DOM itself. You access values using a `ref` instead of state.

```jsx
<input ref={inputRef} />
```

**Pros:** Less code for simple cases, useful when integrating with non-React code.

> In most modern React apps, **controlled components** are preferred because they make state management and validation much easier.

---

## Basic Controlled Form

```jsx
import { useState } from "react";

function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault(); // Prevent page reload
    console.log("Email:", email);
    console.log("Password:", password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Email:
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </label>

      <label>
        Password:
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </label>

      <button type="submit">Login</button>
    </form>
  );
}

export default LoginForm;
```

**Key points:**
- `e.preventDefault()` stops the default browser behavior of reloading the page on submit.
- Each input's `value` comes from state, and `onChange` updates that state.

---

## Handling Multiple Inputs

Instead of creating a separate state variable for every field, use a single state object.

```jsx
import { useState } from "react";

function SignupForm() {
  const [formData, setFormData] = useState({
    username: "",
    email: "",
    password: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value,
    }));
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
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <button type="submit">Sign Up</button>
    </form>
  );
}

export default SignupForm;
```

## Handling Different Input Types

### Checkbox

```jsx
const [isChecked, setIsChecked] = useState(false);

<input
  type="checkbox"
  checked={isChecked}
  onChange={(e) => setIsChecked(e.target.checked)}
/>
```

### Radio Buttons

```jsx
const [gender, setGender] = useState("male");

<label>
  <input
    type="radio"
    name="gender"
    value="male"
    checked={gender === "male"}
    onChange={(e) => setGender(e.target.value)}
  />
  Male
</label>
<label>
  <input
    type="radio"
    name="gender"
    value="female"
    checked={gender === "female"}
    onChange={(e) => setGender(e.target.value)}
  />
  Female
</label>
```

### Select Dropdown

```jsx
const [country, setCountry] = useState("india");

<select value={country} onChange={(e) => setCountry(e.target.value)}>
  <option value="india">India</option>
  <option value="usa">USA</option>
  <option value="uk">UK</option>
</select>
```

### Textarea

```jsx
const [message, setMessage] = useState("");

<textarea
  value={message}
  onChange={(e) => setMessage(e.target.value)}
/>
```

> Note: In React, `<textarea>` uses a `value` prop instead of children text, unlike plain HTML.

---

## Form Validation

Basic manual validation example:

```jsx
import { useState } from "react";

function ValidatedForm() {
  const [email, setEmail] = useState("");
  const [error, setError] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();

    if (!email.includes("@")) {
      setError("Please enter a valid email address.");
      return;
    }

    setError("");
    console.log("Form submitted:", email);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
      />
      {error && <p style={{ color: "red" }}>{error}</p>}
      <button type="submit">Submit</button>
    </form>
  );
}

export default ValidatedForm;
```

For more complex validation (multiple fields, regex rules, async checks), it's common to use a library instead of writing everything manually.

---

## Uncontrolled Forms with useRef

Sometimes you don't need React state for every keystroke — you just want the value at submit time.

```jsx
import { useRef } from "react";

function UncontrolledForm() {
  const nameRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Name:", nameRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" ref={nameRef} placeholder="Enter your name" />
      <button type="submit">Submit</button>
    </form>
  );
}

export default UncontrolledForm;
```

This avoids re-rendering the component on every keystroke, which can be a small performance win for very large forms.

---

## Using Third-Party Libraries

For real-world, production-grade forms, most React developers use libraries instead of managing everything by hand:

| Library | Purpose |
|---|---|
| **React Hook Form** | Lightweight, performant form state management with minimal re-renders |
| **Formik** | Popular form library with built-in validation support |
| **Yup / Zod** | Schema-based validation, often paired with React Hook Form or Formik |

Example using **React Hook Form**:

```jsx
import { useForm } from "react-hook-form";

function HookForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("email", { required: "Email is required" })}
        placeholder="Email"
      />
      {errors.email && <p>{errors.email.message}</p>}

      <button type="submit">Submit</button>
    </form>
  );
}

export default HookForm;
```


# ⚛️ Event Handling 



## 📌 Table of Contents

- [React vs Vanilla JS Events](#react-vs-vanilla-js-events)
- [Basic Event Handling](#basic-event-handling)
- [Passing Arguments to Handlers](#passing-arguments-to-handlers)
- [The Synthetic Event Object](#the-synthetic-event-object)
- [Common Event Types in React](#common-event-types-in-react)
- [Event Handling with Hooks](#event-handling-with-hooks)
- [Two-Way Binding (Controlled Components)](#two-way-binding-controlled-components)
- [preventDefault & stopPropagation](#preventdefault--stoppropagation)
- [Event Delegation in React](#event-delegation-in-react)
- [useRef for Direct DOM Events](#useref-for-direct-dom-events)
- [Custom Event Hooks](#custom-event-hooks)

---

## React vs Vanilla JS Events

| Feature                  | Vanilla JS                        | React                          |
|--------------------------|-----------------------------------|--------------------------------|
| Event name               | `onclick`, `onkeydown` (lowercase)| `onClick`, `onKeyDown` (camelCase) |
| Handler assignment       | String or function                | JSX expression `{handler}`     |
| Event object             | Native DOM event                  | Synthetic Event (wrapped)      |
| `addEventListener`       | Manual                            | Not needed (JSX handles it)    |
| Default prevention       | `event.preventDefault()`         | Same, but inside React handler |

```jsx
// ❌ Vanilla JS way (don't do this in React)
<button onclick="handleClick()">Click</button>

// ✅ React way
<button onClick={handleClick}>Click</button>
```

---

## Basic Event Handling

### Functional Component (Modern Approach)

```jsx
function MyButton() {
  function handleClick() {
    console.log("Button clicked!");
  }

  return <button onClick={handleClick}>Click Me</button>;
}
```

### Arrow Function Inline

```jsx
function MyButton() {
  return (
    <button onClick={() => console.log("Clicked!")}>
      Click Me
    </button>
  );
}
```

> ⚠️ Inline arrow functions create a new function on every render. For simple cases it's fine, but avoid for performance-critical components.

### Class Component (Legacy)

```jsx
class MyButton extends React.Component {
  handleClick() {
    console.log("Clicked!", this); // ⚠️ 'this' can be undefined
  }

  handleClickBound = () => {
    console.log("Clicked!", this); // ✅ Arrow function — 'this' is bound
  };

  render() {
    return <button onClick={this.handleClickBound}>Click Me</button>;
  }
}
```

---

## Passing Arguments to Handlers

### Method 1: Arrow Function in JSX

```jsx
function ItemList() {
  function handleDelete(id) {
    console.log("Deleting item:", id);
  }

  return (
    <ul>
      <li>Item 1 <button onClick={() => handleDelete(1)}>Delete</button></li>
      <li>Item 2 <button onClick={() => handleDelete(2)}>Delete</button></li>
    </ul>
  );
}
```

### Method 2: `.bind()` (Class Components)

```jsx
<button onClick={this.handleDelete.bind(this, itemId)}>Delete</button>
```

### Method 3: Currying (Clean Pattern)

```jsx
function handleDelete(id) {
  return function () {
    console.log("Deleting:", id);
  };
}

// Usage
<button onClick={handleDelete(item.id)}>Delete</button>
```

---

## The Synthetic Event Object

React wraps native browser events in a **SyntheticEvent** — a cross-browser wrapper that behaves consistently across all browsers.

```jsx
function MyInput() {
  function handleChange(event) {
    console.log(event.type);          // "change"
    console.log(event.target);        // <input> element
    console.log(event.target.value);  // Current input value
    console.log(event.nativeEvent);   // Original DOM event
  }

  return <input onChange={handleChange} />;
}
```

### Commonly Used Properties

| Property               | Description                                  |
|------------------------|----------------------------------------------|
| `event.target`         | Element that triggered the event             |
| `event.target.value`   | Value of input/select/textarea               |
| `event.target.name`    | `name` attribute of the element              |
| `event.target.checked` | Checked state of checkbox/radio              |
| `event.key`            | Key pressed (keyboard events)                |
| `event.clientX/Y`      | Mouse position                               |
| `event.preventDefault()`| Stops default behavior                     |
| `event.stopPropagation()`| Stops event bubbling                       |
| `event.nativeEvent`    | Access to the raw DOM event                  |

---

## Common Event Types in React

### 🖱️ Mouse Events

```jsx
<div
  onClick={(e) => console.log("Clicked", e.clientX, e.clientY)}
  onDoubleClick={() => console.log("Double clicked!")}
  onMouseEnter={() => console.log("Mouse entered")}
  onMouseLeave={() => console.log("Mouse left")}
  onMouseMove={(e) => console.log(e.clientX, e.clientY)}
  onContextMenu={(e) => { e.preventDefault(); console.log("Right clicked"); }}
>
  Hover or click me
</div>
```

### ⌨️ Keyboard Events

```jsx
function SearchBox() {
  function handleKeyDown(event) {
    if (event.key === "Enter") {
      console.log("Search triggered!");
    }
    if (event.key === "Escape") {
      console.log("Search cancelled!");
    }
  }

  return <input onKeyDown={handleKeyDown} placeholder="Search..." />;
}
```

### 📋 Form Events

```jsx
function MyForm() {
  function handleSubmit(event) {
    event.preventDefault(); // VERY IMPORTANT in React forms
    console.log("Form submitted!");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 🔤 Input / Change Events

```jsx
function MyInput() {
  function handleChange(event) {
    console.log(event.target.value); // Fires on every keystroke
  }

  function handleBlur() {
    console.log("Input lost focus");
  }

  function handleFocus() {
    console.log("Input focused");
  }

  return (
    <input
      onChange={handleChange}
      onBlur={handleBlur}
      onFocus={handleFocus}
    />
  );
}
```

---

## Event Handling with Hooks

### `useState` + Events

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

### Handling Input with `useState`

```jsx
import { useState } from "react";

function NameForm() {
  const [name, setName] = useState("");

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <p>Hello, {name || "stranger"}!</p>
    </div>
  );
}
```

### Multiple Inputs with One Handler

```jsx
import { useState } from "react";

function RegisterForm() {
  const [formData, setFormData] = useState({
    username: "",
    email: "",
    password: "",
  });

  function handleChange(event) {
    const { name, value } = event.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value,       // Dynamic key using input's name attribute
    }));
  }

  return (
    <form>
      <input name="username" value={formData.username} onChange={handleChange} placeholder="Username" />
      <input name="email"    value={formData.email}    onChange={handleChange} placeholder="Email" />
      <input name="password" value={formData.password} onChange={handleChange} placeholder="Password" type="password" />
      <pre>{JSON.stringify(formData, null, 2)}</pre>
    </form>
  );
}
```

### `useCallback` — Memoize Event Handlers

```jsx
import { useState, useCallback } from "react";

function ParentComponent() {
  const [count, setCount] = useState(0);

  // Without useCallback: new function created on every render
  // With useCallback: function only recreated when dependencies change
  const handleClick = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []); // Empty deps = created once

  return <ChildButton onClick={handleClick} />;
}
```

> ✅ Use `useCallback` when passing handlers to memoized child components (`React.memo`) to prevent unnecessary re-renders.

---

## Two-Way Binding (Controlled Components)

In React, form elements are **controlled** when their value is driven by state.

```jsx
import { useState } from "react";

function ControlledForm() {
  const [text, setText] = useState("");
  const [selected, setSelected] = useState("banana");
  const [checked, setChecked] = useState(false);

  return (
    <div>
      {/* Text Input */}
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />

      {/* Select Dropdown */}
      <select value={selected} onChange={(e) => setSelected(e.target.value)}>
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="mango">Mango</option>
      </select>

      {/* Checkbox */}
      <input
        type="checkbox"
        checked={checked}
        onChange={(e) => setChecked(e.target.checked)}
      />

      <p>Text: {text}</p>
      <p>Selected: {selected}</p>
      <p>Checked: {String(checked)}</p>
    </div>
  );
}
```

---

## preventDefault & stopPropagation

### `preventDefault()` — Stop Default Browser Behavior

```jsx
// Prevent link navigation
<a href="https://google.com" onClick={(e) => e.preventDefault()}>
  Won't Navigate
</a>

// Prevent form reload on submit
<form onSubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
  ...
</form>
```

### `stopPropagation()` — Stop Event Bubbling

```jsx
function App() {
  return (
    <div onClick={() => console.log("DIV clicked")}>
      <button
        onClick={(e) => {
          e.stopPropagation(); // DIV's handler will NOT fire
          console.log("BUTTON clicked");
        }}
      >
        Click
      </button>
    </div>
  );
}

// Without stopPropagation: "BUTTON clicked" then "DIV clicked"
// With stopPropagation:    "BUTTON clicked" only
```

---

## Event Delegation in React

React handles event delegation **automatically** — all events are attached to the root element (`#root`), not individual DOM nodes.

You rarely need manual delegation in React, but here's how to handle dynamic lists cleanly:

```jsx
function TaskList() {
  const tasks = ["Task A", "Task B", "Task C"];

  function handleAction(event) {
    const action = event.target.dataset.action;
    const index  = event.target.dataset.index;

    if (action === "delete") console.log("Delete task", index);
    if (action === "edit")   console.log("Edit task", index);
  }

  return (
    <ul onClick={handleAction}>
      {tasks.map((task, i) => (
        <li key={i}>
          {task}
          <button data-action="edit"   data-index={i}>Edit</button>
          <button data-action="delete" data-index={i}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## useRef for Direct DOM Events

Use `useRef` when you need to attach event listeners directly to DOM elements (e.g., global keyboard shortcuts, third-party integrations).

```jsx
import { useRef, useEffect } from "react";

function KeyboardShortcut() {
  const boxRef = useRef(null);

  useEffect(() => {
    const element = boxRef.current;

    function handleKeyDown(e) {
      if (e.key === "Enter") console.log("Enter pressed on box!");
    }

    element.addEventListener("keydown", handleKeyDown);

    // ✅ Always clean up to avoid memory leaks
    return () => {
      element.removeEventListener("keydown", handleKeyDown);
    };
  }, []);

  return <div ref={boxRef} tabIndex={0} style={{ padding: 20, border: "1px solid" }}>Focus me and press Enter</div>;
}
```

---

## Custom Event Hooks

Reusable hooks for common event patterns:

### `useKeyPress` Hook

```jsx
import { useState, useEffect } from "react";

function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = useState(false);

  useEffect(() => {
    function downHandler({ key }) {
      if (key === targetKey) setKeyPressed(true);
    }

    function upHandler({ key }) {
      if (key === targetKey) setKeyPressed(false);
    }

    window.addEventListener("keydown", downHandler);
    window.addEventListener("keyup", upHandler);

    return () => {
      window.removeEventListener("keydown", downHandler);
      window.removeEventListener("keyup", upHandler);
    };
  }, [targetKey]);

  return keyPressed;
}

// Usage
function App() {
  const enterPressed = useKeyPress("Enter");
  return <p>Enter key is {enterPressed ? "pressed" : "not pressed"}</p>;
}
```

### `useWindowResize` Hook

```jsx
import { useState, useEffect } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    }

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}

// Usage
function App() {
  const { width, height } = useWindowSize();
  return <p>Window: {width} x {height}</p>;
}
```

### `useClickOutside` Hook

```jsx
import { useEffect } from "react";

function useClickOutside(ref, callback) {
  useEffect(() => {
    function handleClick(event) {
      if (ref.current && !ref.current.contains(event.target)) {
        callback();
      }
    }

    document.addEventListener("mousedown", handleClick);
    return () => document.removeEventListener("mousedown", handleClick);
  }, [ref, callback]);
}

// Usage (e.g., close modal on outside click)
function Modal({ onClose }) {
  const modalRef = useRef(null);
  useClickOutside(modalRef, onClose);

  return <div ref={modalRef} className="modal">Modal Content</div>;
}
```

---


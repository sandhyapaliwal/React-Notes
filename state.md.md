# ⚛️ React.js State 


## 📚 Table of Contents

- [What is State?](#-what-is-state)
- [useState Hook](#-usestate-hook)
- [Updating State](#-updating-state)
- [State with Objects](#-state-with-objects)
- [State with Arrays](#-state-with-arrays)
- [useReducer Hook](#-usereducer-hook)
- [Lifting State Up](#-lifting-state-up)
- [Context API (Global State)](#-context-api-global-state)
- [Common Mistakes](#-common-mistakes)
- [Best Practices](#-best-practices)

---

## 🔍 What is State?

**State** is a built-in React object that stores data or information about a component. When state changes, the component **re-renders** automatically.

```
State = Data that changes over time and affects what is shown on the screen
```

### State vs Props

| Feature     | State                        | Props                        |
|-------------|------------------------------|------------------------------|
| Ownership   | Owned by the component       | Passed from parent           |
| Mutable     | ✅ Yes (via setter function)  | ❌ No (read-only)             |
| Triggers re-render | ✅ Yes               | ✅ Yes (when parent changes)  |

---

## 🪝 useState Hook

`useState` is the most common way to add state in a **functional component**.

### Syntax

```jsx
const [state, setState] = useState(initialValue);
```

### Basic Example

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0); // initial value = 0

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

### With String State

```jsx
function Greeting() {
  const [name, setName] = useState("World");

  return (
    <div>
      <h1>Hello, {name}!</h1>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
    </div>
  );
}
```

### With Boolean State

```jsx
function Toggle() {
  const [isVisible, setIsVisible] = useState(false);

  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        {isVisible ? "Hide" : "Show"}
      </button>
      {isVisible && <p>Now you see me! 👋</p>}
    </div>
  );
}
```

---

## 🔄 Updating State

### ⚠️ Never Mutate State Directly

```jsx
// ❌ WRONG — do not mutate directly
count = count + 1;

// ✅ CORRECT — use setter function
setCount(count + 1);
```

### Functional Updates (Recommended for derived state)

When the new state depends on the previous state, use the **functional form**:

```jsx
// ❌ May cause bugs in async situations
setCount(count + 1);

// ✅ Always uses the latest state
setCount(prevCount => prevCount + 1);
```

### Batched Updates

React 18+ automatically **batches** multiple `setState` calls into a single re-render:

```jsx
function handleClick() {
  setCount(c => c + 1);  // \
  setName("Alice");       //  → Only ONE re-render
  setActive(true);        // /
}
```

---

## 🗂️ State with Objects

When state is an **object**, always spread the previous state to avoid overwriting other fields.

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: "",
    age: 0,
    email: "",
  });

  const handleChange = (field, value) => {
    setUser(prev => ({
      ...prev,        // keep all existing fields
      [field]: value  // update only this field
    }));
  };

  return (
    <form>
      <input
        placeholder="Name"
        onChange={(e) => handleChange("name", e.target.value)}
      />
      <input
        placeholder="Age"
        type="number"
        onChange={(e) => handleChange("age", e.target.value)}
      />
      <input
        placeholder="Email"
        onChange={(e) => handleChange("email", e.target.value)}
      />
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </form>
  );
}
```

---

## 📋 State with Arrays

Never mutate arrays. Use **immutable methods** like `map`, `filter`, and spread (`...`).

### Add an Item

```jsx
const [items, setItems] = useState([]);

const addItem = (newItem) => {
  setItems(prev => [...prev, newItem]);
};
```

### Remove an Item

```jsx
const removeItem = (id) => {
  setItems(prev => prev.filter(item => item.id !== id));
};
```

### Update an Item

```jsx
const updateItem = (id, updatedData) => {
  setItems(prev =>
    prev.map(item => item.id === id ? { ...item, ...updatedData } : item)
  );
};
```

### Full Todo Example

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState("");

  const addTodo = () => {
    if (!input.trim()) return;
    setTodos(prev => [...prev, { id: Date.now(), text: input, done: false }]);
    setInput("");
  };

  const toggleTodo = (id) => {
    setTodos(prev =>
      prev.map(t => t.id === id ? { ...t, done: !t.done } : t)
    );
  };

  const deleteTodo = (id) => {
    setTodos(prev => prev.filter(t => t.id !== id));
  };

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.done ? "line-through" : "none" }}
              onClick={() => toggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>❌</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## ⚙️ useReducer Hook

`useReducer` is better than `useState` when state logic is **complex** or has multiple sub-values.

### Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### Counter with useReducer

```jsx
import { useReducer } from "react";

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT": return { count: state.count + 1 };
    case "DECREMENT": return { count: state.count - 1 };
    case "RESET":     return { count: 0 };
    default:          return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
    </div>
  );
}
```

### When to use `useReducer` vs `useState`

| Situation                          | Use         |
|------------------------------------|-------------|
| Simple value (number, string, bool)| `useState`  |
| Multiple related values            | `useReducer`|
| Complex update logic               | `useReducer`|
| Next state depends on previous     | Both work, but `useReducer` is cleaner |

---

## ⬆️ Lifting State Up

When **multiple components** need to share the same state, move it to their **closest common parent**.

```jsx
// ✅ Parent owns the state, children receive it via props

function Parent() {
  const [value, setValue] = useState("");

  return (
    <div>
      <ChildInput value={value} onChange={setValue} />
      <ChildDisplay value={value} />
    </div>
  );
}

function ChildInput({ value, onChange }) {
  return (
    <input value={value} onChange={e => onChange(e.target.value)} />
  );
}

function ChildDisplay({ value }) {
  return <p>You typed: {value}</p>;
}
```

---

## 🌍 Context API (Global State)

Use **Context API** to share state across many components without prop drilling.

```jsx
import { createContext, useContext, useState } from "react";

// 1. Create context
const ThemeContext = createContext();

// 2. Provide context
function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Page />
    </ThemeContext.Provider>
  );
}

// 3. Consume context anywhere
function Page() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div style={{ background: theme === "light" ? "#fff" : "#333" }}>
      <p>Current theme: {theme}</p>
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
    </div>
  );
}
```

---

## ❌ Common Mistakes

### 1. Mutating State Directly

```jsx
// ❌ Wrong
user.name = "Alice";
setUser(user);

// ✅ Correct
setUser({ ...user, name: "Alice" });
```

### 2. Setting State in Render (Infinite Loop)

```jsx
// ❌ Causes infinite re-render
function Bad() {
  const [count, setCount] = useState(0);
  setCount(1); // ← called during render!
  return <div>{count}</div>;
}

// ✅ Use useEffect for side effects
function Good() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setCount(1);
  }, []); // only runs once
  return <div>{count}</div>;
}
```

### 3. Stale State in Closures

```jsx
// ❌ Uses stale 'count' from closure
setTimeout(() => {
  setCount(count + 1);
}, 1000);

// ✅ Always uses latest state
setTimeout(() => {
  setCount(prev => prev + 1);
}, 1000);
```

---


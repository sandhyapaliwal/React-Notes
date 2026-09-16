# Parent-Child Communication in React

React components are organized in a tree, and data typically flows in one direction — from parent to child. But real applications need communication in both directions: parents passing data down, and children sending data/events back up. 

## Table of Contents

- [Overview: How Data Flows in React](#overview-how-data-flows-in-react)
- [1. Parent to Child: Passing Props](#1-parent-to-child-passing-props)
- [2. Child to Parent: Callback Functions](#2-child-to-parent-callback-functions)
- [3. Passing Data with Events](#3-passing-data-with-events)
- [4. Sibling Communication (via a Common Parent)](#4-sibling-communication-via-a-common-parent)
- [5. Passing Multiple Values](#5-passing-multiple-values)
- [6. Children Prop and Composition](#6-children-prop-and-composition)
- [7. Ref-Based Communication (useRef + forwardRef)](#7-ref-based-communication-useref--forwardref)
- [8. Context API for Deeply Nested Trees](#8-context-api-for-deeply-nested-trees)

## Overview: How Data Flows in React

React follows a **unidirectional (one-way) data flow**: state lives in a component, and is passed down to children via **props**. Children cannot directly modify a parent's state — instead, the parent passes down a **function** as a prop, and the child calls that function to notify the parent something happened. This keeps data flow predictable and easy to debug.

```
Parent (owns state)
   │
   ├── passes data down as props
   │
   ▼
Child (receives props, calls callback props to notify parent)
```

## 1. Parent to Child: Passing Props

The simplest form of communication — a parent passes values to a child as **props**.

```jsx
function Parent() {
  const username = "Sandhya";

  return <Child name={username} />;
}

function Child({ name }) {
  return <h2>Hello, {name}!</h2>;
}
```

Props are **read-only** in the child — the child should never reassign or mutate them directly.

## 2. Child to Parent: Callback Functions

Since a child can't modify the parent's state directly, the parent passes a **function** as a prop. The child calls this function (usually in response to an event) to send data back up.

```jsx
function Parent() {
  const [message, setMessage] = useState("");

  const handleChildMessage = (text) => {
    setMessage(text);
  };

  return (
    <div>
      <p>Message from child: {message}</p>
      <Child onSend={handleChildMessage} />
    </div>
  );
}

function Child({ onSend }) {
  return (
    <button onClick={() => onSend("Hello from Child!")}>
      Send Message to Parent
    </button>
  );
}
```

This is the most common pattern for child-to-parent communication and works for any depth of nesting (as long as props are passed down the chain).

## 3. Passing Data with Events

Callback functions often need to pass along data from the event itself (e.g., an input's value).

```jsx
function Parent() {
  const [inputValue, setInputValue] = useState("");

  return (
    <div>
      <p>You typed: {inputValue}</p>
      <ChildInput onChange={(value) => setInputValue(value)} />
    </div>
  );
}

function ChildInput({ onChange }) {
  return (
    <input
      type="text"
      onChange={(e) => onChange(e.target.value)}
      placeholder="Type something..."
    />
  );
}
```

## 4. Sibling Communication (via a Common Parent)

React has no direct sibling-to-sibling channel. Two children that need to share data must **lift state up** to their common parent, which then passes it down to both.

```jsx
function Parent() {
  const [sharedValue, setSharedValue] = useState("");

  return (
    <div>
      <ChildA onUpdate={setSharedValue} />
      <ChildB value={sharedValue} />
    </div>
  );
}

function ChildA({ onUpdate }) {
  return <button onClick={() => onUpdate("Updated by A")}>Update</button>;
}

function ChildB({ value }) {
  return <p>ChildB sees: {value}</p>;
}
```

`ChildA` sends data up to `Parent` via a callback, and `Parent` passes it down to `ChildB` as a prop — this is often called **"lifting state up."**

## 5. Passing Multiple Values

You can pass multiple arguments through a callback, or bundle them into an object.

```jsx
function Parent() {
  const handleSubmit = (name, age) => {
    console.log(`${name} is ${age} years old`);
  };

  return <Child onSubmit={handleSubmit} />;
}

function Child({ onSubmit }) {
  const [name, setName] = useState("");
  const [age, setAge] = useState("");

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" />
      <input value={age} onChange={(e) => setAge(e.target.value)} placeholder="Age" />
      <button onClick={() => onSubmit(name, age)}>Submit</button>
    </div>
  );
}
```

For more than a couple of values, prefer passing a single object for clarity:

```jsx
onSubmit({ name, age });
```

## 6. Children Prop and Composition

The special `children` prop lets a parent pass entire elements (not just data) into a child — useful for layout/wrapper components.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function Parent() {
  return (
    <Card>
      <h2>Title</h2>
      <p>Some content inside the card.</p>
    </Card>
  );
}
```

You can go further and pass a **render prop** — a function as `children` — when the child needs to hand data back to whatever is being rendered inside it.

```jsx
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return <div onMouseMove={handleMouseMove}>{children(position)}</div>;
}

function App() {
  return (
    <MouseTracker>
      {(position) => (
        <p>
          Mouse position: {position.x}, {position.y}
        </p>
      )}
    </MouseTracker>
  );
}
```

## 7. Ref-Based Communication (useRef + forwardRef)

Occasionally a parent needs to **imperatively** call a method on a child (e.g., focus an input, trigger a video play). This bypasses the normal props flow using `forwardRef` and `useImperativeHandle`.

```jsx
import { useRef, forwardRef, useImperativeHandle } from "react";

const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
  }));

  return <input ref={inputRef} {...props} />;
});

function Parent() {
  const inputRef = useRef(null);

  return (
    <div>
      <CustomInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus Input</button>
    </div>
  );
}
```

> This pattern should be used sparingly — it's an escape hatch for imperative actions, not a replacement for normal props/callback communication.

## 8. Context API for Deeply Nested Trees

When data needs to reach components many levels deep, passing props through every intermediate component ("prop drilling") gets messy. **Context** lets any descendant read (or update) shared state directly.

```jsx
import { createContext, useContext, useState } from "react";

const UserContext = createContext(null);

function Parent() {
  const [user, setUser] = useState({ name: "Sandhya" });

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}

function Layout() {
  return (
    <div>
      <Header />
    </div>
  );
}

function Header() {
  // No need to pass "user" through Layout as a prop
  const { user } = useContext(UserContext);
  return <h1>Welcome, {user.name}</h1>;
}

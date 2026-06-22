# Lists & Keys in React

A complete guide to rendering lists and using keys effectively in React.

---

## Table of Contents

- [What are Lists in React?](#what-are-lists-in-react)
- [Rendering Lists with `.map()`](#rendering-lists-with-map)
- [Rendering Lists in JSX](#rendering-lists-in-jsx)
- [What are Keys?](#what-are-keys)
- [Why are Keys Important?](#why-are-keys-important)
- [Rules for Keys](#rules-for-keys)
- [Good vs Bad Key Examples](#good-vs-bad-key-examples)
- [Keys with Components](#keys-with-components)
- [Nested Lists](#nested-lists)

---

## What are Lists in React?

In React, **lists** are used to display multiple similar items — like a list of users, products, or messages — by rendering an array of elements dynamically.

Instead of writing each item manually in JSX, React lets you loop through an array and render each item programmatically.

---

## Rendering Lists with `.map()`

The most common way to render a list in React is using the JavaScript `.map()` method.

```jsx
const fruits = ['Apple', 'Banana', 'Mango'];

function FruitList() {
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}
```

**Output:**
- Apple
- Banana
- Mango

---

## Rendering Lists in JSX

You can embed the `.map()` call directly inside JSX using curly braces `{}`.

```jsx
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' },
];

function UserList() {
  return (
    <div>
      <h2>User List</h2>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## What are Keys?

A **key** is a special prop in React that helps it identify which items in a list have changed, been added, or been removed.

```jsx
<li key="unique-id">Item</li>
```

Keys must be passed to the **outermost element** returned inside a `.map()` call.

> Keys are not accessible as props inside the component — they are used internally by React only.

---

## Why are Keys Important?

React uses a **reconciliation algorithm** to efficiently update the DOM. Without keys, React has no way to know which item changed, so it re-renders the entire list — which is slow and can cause bugs.

With keys, React can:

- **Identify** which item was added, removed, or updated
- **Reorder** items without re-rendering everything
- **Preserve component state** across re-renders

**Without key (bad):**
```jsx
items.map((item) => <li>{item.name}</li>)
// ⚠️ Warning: Each child in a list should have a unique "key" prop.
```

**With key (good):**
```jsx
items.map((item) => <li key={item.id}>{item.name}</li>)
// ✅ React can now track each item
```

---

## Rules for Keys

| Rule | Description |
|------|-------------|
| **Must be unique** | Keys must be unique among siblings (not globally) |
| **Must be stable** | Keys should not change between renders |
| **Must be a string or number** | Keys are strings or numbers (React converts numbers to strings) |
| **Don't use index as key (if avoidable)** | Index can cause bugs when list order changes |

---

## Good vs Bad Key Examples

### ✅ Good — Using a unique ID from data

```jsx
const products = [
  { id: 'p1', name: 'Laptop' },
  { id: 'p2', name: 'Phone' },
  { id: 'p3', name: 'Tablet' },
];

function ProductList() {
  return (
    <ul>
      {products.map((product) => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

### ⚠️ Acceptable — Using index (only for static, non-reorderable lists)

```jsx
const staticItems = ['Home', 'About', 'Contact'];

function NavMenu() {
  return (
    <ul>
      {staticItems.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}
```

### ❌ Bad — Using random or unstable keys

```jsx
// BAD: Math.random() creates a new key on every render
items.map((item) => (
  <li key={Math.random()}>{item.name}</li>
))
```

---

## Keys with Components

When you render a **custom component** inside `.map()`, the key goes on the component tag — not inside the component itself.

```jsx
function TodoItem({ task }) {
  return <li>{task.text}</li>;
}

function TodoList() {
  const tasks = [
    { id: 1, text: 'Learn React' },
    { id: 2, text: 'Build a project' },
    { id: 3, text: 'Get a job' },
  ];

  return (
    <ul>
      {tasks.map((task) => (
        <TodoItem key={task.id} task={task} />
        // ✅ key is on <TodoItem>, not inside it
      ))}
    </ul>
  );
}
```

> **Note:** Inside `TodoItem`, `props.key` is NOT accessible. If you need the ID inside the component, pass it separately as a prop: `<TodoItem key={task.id} id={task.id} task={task} />`

---

## Nested Lists

You can render lists inside lists by nesting `.map()` calls. Make sure each level has its own unique keys.

```jsx
const categories = [
  {
    id: 'cat1',
    name: 'Fruits',
    items: ['Apple', 'Banana', 'Mango'],
  },
  {
    id: 'cat2',
    name: 'Vegetables',
    items: ['Carrot', 'Broccoli', 'Spinach'],
  },
];

function CategoryList() {
  return (
    <div>
      {categories.map((category) => (
        <div key={category.id}>
          <h3>{category.name}</h3>
          <ul>
            {category.items.map((item) => (
              <li key={item}>{item}</li>
              // ✅ item name used as key — unique within siblings
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

---

## Common Mistakes

### 1. Forgetting the `key` prop

```jsx
// ❌ Missing key
items.map((item) => <li>{item.name}</li>)

// ✅ Fixed
items.map((item) => <li key={item.id}>{item.name}</li>)
```

### 2. Putting `key` inside the component instead of on it

```jsx
// ❌ Wrong placement
function MyItem({ id, name }) {
  return <li key={id}>{name}</li>; // key here is ignored!
}

// Then in the parent:
items.map((item) => <MyItem id={item.id} name={item.name} />)

// ✅ Correct placement
items.map((item) => <MyItem key={item.id} id={item.id} name={item.name} />)
```

### 3. Using index as key when list can be reordered or filtered

```jsx
// ❌ Problematic if items are reordered/deleted
items.map((item, index) => <li key={index}>{item.name}</li>)

// ✅ Better — use stable unique ID
items.map((item) => <li key={item.id}>{item.name}</li>)
```

### 4. Duplicate keys

```jsx
// ❌ Duplicate keys cause unpredictable behavior
<li key="1">Item A</li>
<li key="1">Item B</li>

// ✅ Each key must be unique among siblings
<li key="item-a">Item A</li>
<li key="item-b">Item B</li>
```

---


# Lifting State Up in React

**Lifting State Up** is a core React pattern where shared state is moved from child components to their closest common parent, so multiple components can read and update the same data in sync. It's not a hook or an API — it's a way of structuring component state to keep your UI consistent.

## Table of Contents

- [The Problem It Solves](#the-problem-it-solves)
- [The Core Idea](#the-core-idea)
- [Example 1: Two Inputs That Must Stay in Sync](#example-1-two-inputs-that-must-stay-in-sync)
- [Example 2: Shared Filter Between List and Search Box](#example-2-shared-filter-between-list-and-search-box)
- [Example 3: Lifting State Across Three Levels](#example-3-lifting-state-across-three-levels)
- [Step-by-Step Process for Lifting State](#step-by-step-process-for-lifting-state)
- [Lifting State Up vs Other State-Sharing Approaches](#lifting-state-up-vs-other-state-sharing-approaches)

## The Problem It Solves

In React, data flows **one way** — from parent to child, via props. Two sibling components have no direct way to communicate or share state, because neither one is the parent of the other.

```
        Parent
        /    \
   ChildA    ChildB
```

If `ChildA` and `ChildB` each keep their own local state for something that's supposed to be shared or kept in sync (like a temperature value shown in two different units, or a filter that affects a shared list), their state will drift out of sync — updating one won't affect the other.

## The Core Idea

To let sibling components share and stay in sync on a piece of state:

1. **Move the state up** to their closest common parent.
2. The parent passes the state **down** as props to both children.
3. The parent also passes **update functions** (callbacks) down as props, so children can request a change — but the actual state lives only in the parent.

This keeps a **single source of truth**: one component owns the state, and everyone else just reads it or asks the owner to change it.

## Example 1: Two Inputs That Must Stay in Sync

A classic example — a Celsius input and a Fahrenheit input that should always reflect the same temperature.

**Before lifting state (broken — each input has its own state):**

```jsx
function CelsiusInput() {
  const [celsius, setCelsius] = useState("");
  return <input value={celsius} onChange={(e) => setCelsius(e.target.value)} />;
}

function FahrenheitInput() {
  const [fahrenheit, setFahrenheit] = useState("");
  return <input value={fahrenheit} onChange={(e) => setFahrenheit(e.target.value)} />;
}
```

Typing in one input never updates the other — they're disconnected.

**After lifting state up to the parent:**

```jsx
import { useState } from "react";

function toFahrenheit(celsius) {
  return celsius === "" ? "" : (celsius * 9) / 5 + 32;
}

function toCelsius(fahrenheit) {
  return fahrenheit === "" ? "" : ((fahrenheit - 32) * 5) / 9;
}

function TemperatureConverter() {
  const [temperature, setTemperature] = useState("");
  const [scale, setScale] = useState("c"); // "c" for Celsius, "f" for Fahrenheit

  const celsius = scale === "f" ? toCelsius(temperature) : temperature;
  const fahrenheit = scale === "c" ? toFahrenheit(temperature) : temperature;

  return (
    <div>
      <TemperatureInput
        label="Celsius"
        value={celsius}
        onChange={(value) => {
          setScale("c");
          setTemperature(value);
        }}
      />
      <TemperatureInput
        label="Fahrenheit"
        value={fahrenheit}
        onChange={(value) => {
          setScale("f");
          setTemperature(value);
        }}
      />
    </div>
  );
}

function TemperatureInput({ label, value, onChange }) {
  return (
    <fieldset>
      <legend>{label}</legend>
      <input value={value} onChange={(e) => onChange(e.target.value)} />
    </fieldset>
  );
}

export default TemperatureConverter;
```

Now `TemperatureConverter` is the single source of truth. Both inputs read from it and call `onChange` to request updates — they no longer manage their own state.

## Example 2: Shared Filter Between List and Search Box

A search box and a list are siblings — the list needs to know what the search box's current value is.

```jsx
import { useState } from "react";

function ProductPage() {
  const [searchTerm, setSearchTerm] = useState("");

  const products = [
    { id: 1, name: "Laptop" },
    { id: 2, name: "Laptop Stand" },
    { id: 3, name: "Mouse" },
    { id: 4, name: "Mechanical Keyboard" },
  ];

  const filtered = products.filter((p) =>
    p.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div>
      <SearchBox searchTerm={searchTerm} onSearchChange={setSearchTerm} />
      <ProductList products={filtered} />
    </div>
  );
}

function SearchBox({ searchTerm, onSearchChange }) {
  return (
    <input
      type="text"
      placeholder="Search products..."
      value={searchTerm}
      onChange={(e) => onSearchChange(e.target.value)}
    />
  );
}

function ProductList({ products }) {
  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}

export default ProductPage;
```

`searchTerm` lives in `ProductPage` (the closest common parent of `SearchBox` and `ProductList`), so both components stay in sync automatically.

## Example 3: Lifting State Across Three Levels

Sometimes the closest common parent isn't the direct parent — state may need to travel up through an intermediate component too.

```jsx
function App() {
  const [cartCount, setCartCount] = useState(0);

  return (
    <div>
      <Header cartCount={cartCount} />
      <ProductGrid onAddToCart={() => setCartCount((c) => c + 1)} />
    </div>
  );
}

function Header({ cartCount }) {
  return (
    <header>
      <NavBar cartCount={cartCount} /> {/* passed through, not owned here */}
    </header>
  );
}

function NavBar({ cartCount }) {
  return <span>🛒 Cart ({cartCount})</span>;
}

function ProductGrid({ onAddToCart }) {
  return (
    <div>
      <ProductCard onAddToCart={onAddToCart} />
    </div>
  );
}

function ProductCard({ onAddToCart }) {
  return <button onClick={onAddToCart}>Add to Cart</button>;
}
```

Here, `App` is the closest common ancestor of `NavBar` (which displays the count) and `ProductCard` (which triggers the update). The state and its updater pass through `Header` and `ProductGrid` as props, even though those components don't use the value themselves — this passing-through is sometimes called **prop drilling**.

## Step-by-Step Process for Lifting State

When you notice two components need to share state:

1. **Identify every component that needs the state** (to read it, or to change it).
2. **Find their closest common parent** in the component tree.
3. **Move the `useState` call up** to that parent.
4. **Pass the state down as props** to the components that need to read it.
5. **Pass a callback function down as a prop** to the components that need to update it (the state's `setter` function, or a wrapper around it).
6. **Remove the now-redundant local state** from the child components.

## Lifting State Up vs Other State-Sharing Approaches

| Approach | When to Use |
|---|---|
| **Lifting State Up** | A small number of components (usually 2–4) close together in the tree need to share state; simplest solution for local coordination. |
| **Context API** | Many components at different depths need the same data (e.g., theme, logged-in user), and passing props through every intermediate level (prop drilling) becomes unwieldy. |
| **State management library** (Redux, Zustand, Jotai, etc.) | App-wide state that's large, frequently updated, or needs to be accessed/modified from many unrelated parts of a large app. |
| **URL/query params** | State that should be shareable via link or survive a refresh (e.g., a selected tab, a search filter). |

Lifting state up is usually the **first, simplest tool to reach for** — Context and state libraries solve the same core problem but are better suited once prop drilling becomes painful across many layers.

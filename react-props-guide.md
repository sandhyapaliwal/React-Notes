#  React Props 

A comprehensive reference for understanding and using **props** in React.

---

## 📌 Table of Contents

- [What are Props?](#what-are-props)
- [Passing Props](#passing-props)
- [Receiving Props](#receiving-props)
- [Default Props](#default-props)
- [PropTypes (Type Checking)](#proptypes-type-checking)
- [Destructuring Props](#destructuring-props)
- [Passing Functions as Props](#passing-functions-as-props)
- [Passing Children as Props](#passing-children-as-props)
- [Spreading Props](#spreading-props)
- [Props vs State](#props-vs-state)
- [Common Patterns](#common-patterns)

---

## What are Props?

**Props** (short for *properties*) are the mechanism React uses to pass data from a **parent component** to a **child component**.

- Props are **read-only** — a child component should never modify its own props.
- Props flow **one way**: parent → child (unidirectional data flow).
- Any valid JavaScript value can be passed as a prop: strings, numbers, arrays, objects, functions, JSX, etc.

```jsx
// Parent passes data → Child receives and displays it
<UserCard name="Alice" age={25} />
```

---

## Passing Props

Props are passed like HTML attributes in JSX.

```jsx
function App() {
  return (
    <Greeting
      name="Bob"
      age={30}
      isLoggedIn={true}
      scores={[90, 85, 92]}
      address={{ city: "Delhi", zip: "110001" }}
    />
  );
}
```

> **Tip:** Strings use `""`, all other values (numbers, booleans, variables, expressions) use `{}`.

---

## Receiving Props

### Functional Component

```jsx
function Greeting(props) {
  return (
    <div>
      <h1>Hello, {props.name}!</h1>
      <p>Age: {props.age}</p>
    </div>
  );
}
```

### Class Component

```jsx
class Greeting extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, {this.props.name}!</h1>
        <p>Age: {this.props.age}</p>
      </div>
    );
  }
}
```

---

## Default Props

Set fallback values when a prop is not provided by the parent.

### Using `defaultProps`

```jsx
function Button({ label, color }) {
  return <button style={{ backgroundColor: color }}>{label}</button>;
}

Button.defaultProps = {
  label: "Click Me",
  color: "blue",
};
```

### Using Default Parameters (Modern Approach)

```jsx
function Button({ label = "Click Me", color = "blue" }) {
  return <button style={{ backgroundColor: color }}>{label}</button>;
}
```

---

## PropTypes (Type Checking)

Use the `prop-types` library to validate the type of props at runtime (development only).

### Installation

```bash
npm install prop-types
```

### Usage

```jsx
import PropTypes from "prop-types";

function UserCard({ name, age, isAdmin }) {
  return (
    <div>
      <p>{name}</p>
      <p>{age}</p>
      {isAdmin && <span>Admin</span>}
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isAdmin: PropTypes.bool,
};
```

### Common PropTypes

| Type | PropTypes Validator |
|------|---------------------|
| String | `PropTypes.string` |
| Number | `PropTypes.number` |
| Boolean | `PropTypes.bool` |
| Function | `PropTypes.func` |
| Array | `PropTypes.array` |
| Object | `PropTypes.object` |
| Any | `PropTypes.any` |
| JSX / Node | `PropTypes.node` |
| React Element | `PropTypes.element` |
| One of values | `PropTypes.oneOf(['a', 'b'])` |
| Array of type | `PropTypes.arrayOf(PropTypes.string)` |
| Shape | `PropTypes.shape({ id: PropTypes.number })` |



## Destructuring Props

Destructuring makes your component cleaner and avoids repeating `props.`.

```jsx
// Without destructuring
function Profile(props) {
  return <p>{props.name} — {props.city}</p>;
}

// With destructuring (recommended)
function Profile({ name, city }) {
  return <p>{name} — {city}</p>;
}
```

### Nested Destructuring

```jsx
function Address({ location: { city, zip } }) {
  return <p>{city} - {zip}</p>;
}

// Usage
<Address location={{ city: "Mumbai", zip: "400001" }} />
```

---

## Passing Functions as Props

Pass functions from parent to child to allow the child to communicate back up (callback pattern).

```jsx
// Parent
function App() {
  const handleClick = (message) => {
    alert(message);
  };

  return <Button onClick={handleClick} />;
}

// Child
function Button({ onClick }) {
  return (
    <button onClick={() => onClick("Hello from Button!")}>
      Click Me
    </button>
  );
}
```

---

## Passing Children as Props

Use the special `children` prop to pass JSX content between a component's opening and closing tags.

```jsx
// Component that uses children
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Usage
<Card title="My Card">
  <p>This is the card content.</p>
  <button>Action</button>
</Card>
```

---

## Spreading Props

Use the spread operator (`...`) to pass all properties of an object as props at once.

```jsx
const userProps = {
  name: "Alice",
  age: 28,
  role: "Developer",
};

// Instead of this:
<UserCard name={userProps.name} age={userProps.age} role={userProps.role} />

// Do this:
<UserCard {...userProps} />
```

> ⚠️ **Use with caution** — spreading unknown props onto DOM elements can cause React warnings.

---

## Props vs State

| Feature | Props | State |
|--------|-------|-------|
| Defined by | Parent component | Component itself |
| Mutable? | ❌ Read-only | ✅ Can be updated |
| Who controls it? | Parent | The component |
| Triggers re-render? | Yes (when parent re-renders) | Yes (when updated via `setState` / `useState`) |
| Purpose | Pass data down | Manage internal data |

---

## Common Patterns

### Conditional Rendering with Props

```jsx
function Alert({ type, message }) {
  return (
    <div className={`alert alert-${type}`}>
      {message}
    </div>
  );
}

<Alert type="success" message="Saved successfully!" />
<Alert type="error" message="Something went wrong." />
```

### Render Props Pattern

```jsx
function DataFetcher({ render }) {
  const data = ["Apple", "Banana", "Cherry"];
  return render(data);
}

// Usage
<DataFetcher render={(items) => (
  <ul>{items.map((item) => <li key={item}>{item}</li>)}</ul>
)} />
```

### Forwarding Props with Rest Operator

```jsx
function Input({ label, ...rest }) {
  return (
    <div>
      <label>{label}</label>
      <input {...rest} />
    </div>
  );
}

<Input label="Email" type="email" placeholder="Enter email" required />
```

---


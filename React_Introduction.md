# React: A JavaScript Library for Building UIs

React is a JavaScript library maintained by Facebook (now Meta) for building interactive user interfaces using reusable components. It uses a declarative approach, making it easier to predict how your UI will look and behave.

## What is React?

React is fundamentally a view library. It helps you manage how your application looks and responds to user interactions. Instead of manually updating the DOM when data changes, you describe what the UI should look like, and React handles the updates efficiently.

The core idea: **UI = f(state)**

Your interface is a function of your application state. When state changes, React re-renders the UI automatically.

## Key Concepts

### Components

Components are the building blocks of React applications. They're JavaScript functions (or classes) that return JSX, which describes what should appear on the screen.

```javascript
function Greeting() {
  return <h1>Hello, React!</h1>;
}
```

Components are reusable. You can use the same component multiple times, with different data:

```javascript
<Greeting />
<Greeting />
<Greeting />
```

### JSX

JSX is a syntax extension that looks like HTML but is actually JavaScript. It lets you write UI markup directly in your code.

```javascript
const element = <div className="card">Welcome to React</div>;
```

This is transformed into JavaScript function calls:

```javascript
const element = React.createElement('div', { className: 'card' }, 'Welcome to React');
```

You can embed JavaScript expressions inside JSX using curly braces:

```javascript
const name = 'Alice';
const greeting = <h1>Hello, {name}!</h1>;
```

### Props

Props (short for properties) are how you pass data from a parent component to a child component. They're like function arguments.

```javascript
function Card({ title, description }) {
  return (
    <div>
      <h2>{title}</h2>
      <p>{description}</p>
    </div>
  );
}

// Usage
<Card title="React Basics" description="Learn the fundamentals" />
```

Props are read-only. A component should never modify the props it receives.

### State

State is data that a component manages internally. When state changes, the component re-renders.

```javascript
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

`useState` is a React Hook that lets you add state to functional components. It returns an array: the current state value and a function to update it.

### The Virtual DOM

React maintains a virtual representation of the UI in memory (the Virtual DOM). When state changes, React:

1. Creates a new Virtual DOM
2. Compares it with the previous version (diffing)
3. Updates only the parts of the real DOM that changed (reconciliation)

This makes React applications fast, even with frequent updates.

## Component Lifecycle (Hooks)

React Hooks let you use state and other features in functional components.

### useState

Manages component state:

```javascript
const [value, setValue] = useState(initialValue);
```

### useEffect

Runs side effects after rendering:

```javascript
useEffect(() => {
  // Code runs after render
  return () => {
    // Optional cleanup
  };
}, [dependencies]);
```

The dependency array controls when the effect runs:
- Empty array `[]`: runs once after initial render
- No array: runs after every render
- Array with values: runs when those values change

### useContext

Access context values across components without prop drilling:

```javascript
const theme = useContext(ThemeContext);
```

### useReducer

Manage complex state with a reducer function:

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

### useRef

Create persistent references that don't cause re-renders:

```javascript
const inputRef = useRef(null);
```

## Component Patterns

### Controlled Components

The React component controls the input value:

```javascript
function Form() {
  const [name, setName] = useState('');

  return (
    <input 
      value={name} 
      onChange={(e) => setName(e.target.value)} 
    />
  );
}
```

### Composition

Build complex UIs by composing simpler components:

```javascript
function App() {
  return (
    <div>
      <Header />
      <MainContent />
      <Footer />
    </div>
  );
}
```

### Conditional Rendering

Show different content based on conditions:

```javascript
function Dashboard({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <Home /> : <Login />}
    </div>
  );
}
```

### Lists and Keys

When rendering lists, provide unique keys:

```javascript
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

Keys help React identify which items have changed. Use a unique identifier from your data, not the array index.

## Handling Events

Attach event handlers to elements:

```javascript
function Button() {
  const handleClick = () => {
    console.log('Clicked!');
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

Common events: `onClick`, `onChange`, `onSubmit`, `onFocus`, `onBlur`, etc.

## Forms

React provides a way to handle form submissions:

```javascript
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitting:', { email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="email" 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
      />
      <input 
        type="password" 
        value={password} 
        onChange={(e) => setPassword(e.target.value)} 
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

## Styling in React

### Inline Styles

```javascript
const styles = {
  container: {
    backgroundColor: '#f0f0f0',
    padding: '20px',
    borderRadius: '8px'
  }
};

function Card() {
  return <div style={styles.container}>Content</div>;
}
```

### CSS Classes

```javascript
import './Card.css';

function Card() {
  return <div className="card">Content</div>;
}
```

### CSS Modules

```javascript
import styles from './Card.module.css';

function Card() {
  return <div className={styles.card}>Content</div>;
}
```

### Tailwind CSS

```javascript
function Card() {
  return <div className="bg-gray-100 p-5 rounded">Content</div>;
}
```

## API Calls

Fetch data from an API using useEffect:

```javascript
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(response => response.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

## Creating a React App

### Using Vite (Recommended)

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

### Using Create React App

```bash
npx create-react-app my-app
cd my-app
npm start
```

## Project Structure

A typical React project looks like this:

```
my-app/
├── public/
│   └── index.html
├── src/
│   ├── components/
│   │   ├── Header.jsx
│   │   ├── Footer.jsx
│   │   └── Card.jsx
│   ├── hooks/
│   │   └── useCustom.js
│   ├── styles/
│   │   └── global.css
│   ├── App.jsx
│   └── main.jsx
├── package.json
└── vite.config.js
```

## Common Mistakes

**1. Mutating State Directly**

```javascript
// ❌ Wrong
state.name = 'John';

// ✅ Correct
setState({ ...state, name: 'John' });
```

**2. Missing Dependencies in useEffect**

```javascript
// ❌ Wrong - effect never runs again if count changes
useEffect(() => {
  console.log(count);
}, []);

// ✅ Correct
useEffect(() => {
  console.log(count);
}, [count]);
```

**3. Using Array Index as Key**

```javascript
// ❌ Wrong
{items.map((item, index) => <li key={index}>{item}</li>)}

// ✅ Correct
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

**4. Forgetting to Return JSX**

```javascript
// ❌ Wrong
function Card() {
  <div>Hello</div>;
}

// ✅ Correct
function Card() {
  return <div>Hello</div>;
}
```

## Performance Optimization

### useMemo

Memoize expensive calculations:

```javascript
const expensiveValue = useMemo(() => {
  return calculateSomethingExpensive(data);
}, [data]);
```

### useCallback

Memoize function references:

```javascript
const handleClick = useCallback(() => {
  doSomething(data);
}, [data]);
```

### React.memo

Prevent unnecessary re-renders of components:

```javascript
const Card = React.memo(function Card({ title }) {
  return <h2>{title}</h2>;
});
```

## State Management Beyond useState

For larger applications, consider:

- **Context API**: Share state across components without prop drilling
- **Redux**: Centralized state management
- **Zustand**: Lightweight state management
- **Recoil**: Experimental state management from Facebook

## Useful Libraries

- **React Router**: Client-side routing
- **Axios**: HTTP requests (alternative to fetch)
- **Formik**: Form management
- **React Query**: Server state management
- **Jest & React Testing Library**: Testing

## Resources to Learn More

- [React Official Documentation](https://react.dev)
- [React GitHub Repository](https://github.com/facebook/react)
- Build projects to solidify your understanding
- Read other people's React code



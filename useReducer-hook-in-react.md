# useReducer Hook 

`useReducer` is a React Hook used for managing more complex component state. It's an alternative to `useState`, especially useful when state logic involves multiple sub-values, or when the next state depends on the previous one in non-trivial ways.

## Table of Contents

- [Why useReducer?](#why-usereducer)
- [Basic Syntax](#basic-syntax)
- [Step-by-Step Example](#step-by-step-example)
- [Using an Initializer Function](#using-an-initializer-function)
- [useReducer + useContext](#usereducer--usecontext)
- [useReducer vs useState](#usereducer-vs-usestate)
- [Common Pitfalls](#common-pitfalls)


---

## Why useReducer?

As components grow, state logic can become messy with multiple `useState` calls and interdependent update logic scattered across event handlers:

```jsx
const [count, setCount] = useState(0);
const [step, setStep] = useState(1);
const [history, setHistory] = useState([]);
```

Updating all three together correctly, from several different places, gets error-prone fast. `useReducer` centralizes that logic into a single function (the **reducer**) that takes the current state and an **action**, and returns the next state — similar to how Redux works.

## Basic Syntax

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducer, initialState);
```

- `reducer` — a function `(state, action) => newState`
- `initialState` — the starting state value
- `state` — the current state
- `dispatch` — a function you call with an action to trigger a state update

## Step-by-Step Example

### 1. Define the Reducer

```jsx
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}
```

### 2. Use useReducer in a Component

```jsx
import { useReducer } from 'react';

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

Instead of directly setting state, components **dispatch actions** describing *what happened*, and the reducer decides *how the state changes*.

### 3. Actions with a Payload

Actions can carry extra data via a `payload`:

```jsx
function todosReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, { id: Date.now(), text: action.payload, done: false }];
    case 'toggle':
      return state.map((todo) =>
        todo.id === action.payload ? { ...todo, done: !todo.done } : todo
      );
    case 'remove':
      return state.filter((todo) => todo.id !== action.payload);
    default:
      return state;
  }
}

function TodoList() {
  const [todos, dispatch] = useReducer(todosReducer, []);
  const [text, setText] = useState('');

  const addTodo = () => {
    dispatch({ type: 'add', payload: text });
    setText('');
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.done ? 'line-through' : 'none' }}
              onClick={() => dispatch({ type: 'toggle', payload: todo.id })}
            >
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'remove', payload: todo.id })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Using an Initializer Function

`useReducer` accepts an optional third argument — a function to lazily compute the initial state. This is useful when the initial state is expensive to compute or depends on props.

```jsx
function init(initialCount) {
  return { count: initialCount };
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(counterReducer, initialCount, init);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

## useReducer + useContext

A very common pattern is combining `useReducer` with `useContext` to create a lightweight global state manager, without pulling in an external library like Redux.

```jsx
// StoreContext.js
import { createContext, useContext, useReducer } from 'react';

const StoreContext = createContext(null);

const initialState = { user: null, theme: 'light' };

function appReducer(state, action) {
  switch (action.type) {
    case 'login':
      return { ...state, user: action.payload };
    case 'logout':
      return { ...state, user: null };
    case 'toggleTheme':
      return { ...state, theme: state.theme === 'light' ? 'dark' : 'light' };
    default:
      return state;
  }
}

export function StoreProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  return (
    <StoreContext.Provider value={{ state, dispatch }}>
      {children}
    </StoreContext.Provider>
  );
}

export function useStore() {
  return useContext(StoreContext);
}
```

Any component wrapped in `<StoreProvider>` can now read state and dispatch actions:

```jsx
function ThemeToggle() {
  const { state, dispatch } = useStore();
  return (
    <button onClick={() => dispatch({ type: 'toggleTheme' })}>
      Current theme: {state.theme}
    </button>
  );
}
```

## useReducer vs useState

| | `useState` | `useReducer` |
|---|---|---|
| Best for | Simple, independent state values | Complex state logic, multiple sub-values, or state transitions that depend on each other |
| Update mechanism | Direct setter call | Dispatch an action object, handled by a reducer function |
| Predictability | Update logic can be scattered across handlers | Update logic centralized in one reducer function |
| Testability | Harder to isolate update logic | Reducer is a pure function, easy to unit test independently |

Rule of thumb: **start with `useState`. Reach for `useReducer` when state updates become interdependent or you find yourself repeating similar update logic across handlers.**

## Common Pitfalls

1. **Mutating state inside the reducer** — Reducers must return a *new* state object, never mutate the existing one directly (e.g., avoid `state.count++`).
2. **Forgetting a `default` case** — Always return the unchanged `state` (or throw) for unrecognized action types, so unexpected actions don't silently break state.
3. **Overcomplicating simple state** — If a single reducer is only handling one or two straightforward updates, plain `useState` is often simpler and clearer.
4. **Putting side effects inside the reducer** — Reducers should be pure functions with no side effects (no API calls, no timers). Side effects belong in `useEffect` or event handlers, triggered based on updated state.




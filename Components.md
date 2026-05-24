# Components in React

## Functional Components

Functional components are JavaScript functions that return JSX.  
They are now the preferred way to write components in modern React.

**Basic example:**
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}
```

Using Hooks, functional components can also manage state and side effects.

**Example with state:**
```jsx
import React, { useState } from "react";

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

export default Counter;
```

## Class Components

Class components are ES6 classes that extend `React.Component`.  
They use lifecycle methods and have a `render()` method.

**Basic example:**
```jsx
import React, { Component } from "react";

class Welcome extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

export default Welcome;
```

**Example with state and event handler:**
```jsx
import React, { Component } from "react";

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>
          Increment
        </button>
      </div>
    );
  }
}

export default Counter;
```

## Key Differences

| Aspect              | Functional Components                    | Class Components                               |
|---------------------|-------------------------------------------|-----------------------------------------------|
| Syntax              | Simple JavaScript functions               | ES6 classes with `render()` method            |
| State (old way)     | Not possible directly                     | `this.state` and `this.setState()`            |
| State (new way)     | Hooks: `useState`, `useReducer`          | `this.state` and `this.setState()`            |
| Lifecycle methods   | `useEffect` and other Hooks              | `componentDidMount`, `componentDidUpdate`, etc. |
| Boilerplate         | Less code, cleaner                       | More boilerplate and verbose                  |
| Refs                | `useRef` Hook                            | `createRef()` or callback refs                |

React useEffect Hook
The useEffect hook lets you run side effects in React function components, such as fetching data, setting up subscriptions, or working with timers and the DOM.

Side effects are anything that affects something outside the function’s scope, like network requests, logging, or updating the browser title.

js
import React, { useEffect } from "react";
js
useEffect(() => {
  // side effect code here
}, [dependencies]);
1. Why useEffect?
Before hooks, side effects were usually handled in class component lifecycle methods like componentDidMount, componentDidUpdate, and componentWillUnmount.

useEffect combines these lifecycles into a single API for function components and makes it easier to manage side effects in a predictable way.

Typical use cases:

Fetching data from an API.

Subscribing/unsubscribing to events (e.g., window or WebSocket).

Setting up timers / intervals.

Updating document title or interacting with non‑React libraries.

2. Basic Syntax
js
useEffect(() => {
  // effect logic
  // runs after render
}, [dependency1, dependency2]);
First argument: a function containing the side effect.

Second argument: optional dependency array that controls when the effect runs.

If you omit the dependency array, the effect runs after every render.

3. Dependency Array Patterns
3.1 No Dependency Array
js
useEffect(() => {
  console.log("Effect runs on every render");
});
Runs after every render (initial + all updates).

Use carefully: can cause performance issues if the effect is heavy.

3.2 Empty Dependency Array []
js
useEffect(() => {
  console.log("Runs once on mount");
}, []);
Runs only once after the initial render (component mount).

Common for one‑time tasks like initial data fetch or setting up a subscription.

3.3 With Dependencies
js
useEffect(() => {
  console.log("Runs when `count` changes");
}, [count]);
Runs on initial render and every time any listed dependency changes.

Good for reacting to specific state or prop changes.

Best practice: Only put actual data dependencies (state/props used inside the effect) in the array, not every variable from the component.

4. Example: Updating Document Title
4.1 Counter with useEffect
js
import React, { useState, useEffect } from "react";

function CounterWithTitle() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Clicked ${count} times`;
  }, [count]); // runs whenever `count` changes

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(prev => prev + 1)}>
        Click me
      </button>
    </div>
  );
}

export default CounterWithTitle;
This component updates the browser tab title each time the user clicks the button, because the effect depends on count.

5. Example: Fetching Data on Mount
js
import React, { useState, useEffect } from "react";

function Todos() {
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchTodos() {
      try {
        const res = await fetch("https://jsonplaceholder.typicode.com/todos?_limit=5");
        const data = await res.json();
        setTodos(data);
      } catch (error) {
        console.error("Error fetching todos:", error);
      } finally {
        setLoading(false);
      }
    }

    fetchTodos();
  }, []); // runs once on mount

  if (loading) return <p>Loading...</p>;

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}

export default Todos;
Here the effect runs only once when the component mounts and performs an asynchronous fetch, then updates state with the result.

6. Cleanup Functions
Some side effects need cleanup to avoid memory leaks or unwanted behavior, such as event listeners, subscriptions, or timers.

You can return a cleanup function from the effect; React calls it before the component unmounts and before re‑running the effect.

6.1 Timer Example
js
import React, { useState, useEffect } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // Cleanup: clear interval when component unmounts
    return () => {
      clearInterval(id);
    };
  }, []); // set up interval once

  return <p>Timer: {seconds}s</p>;
}

export default Timer;
The cleanup function clears the interval and prevents the timer from continuing after the component is removed.

6.2 Event Listener Example
js
import React, { useState, useEffect } from "react";

function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", handleResize);

    // Cleanup to avoid multiple listeners
    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return <p>Window width: {width}px</p>;
}

export default WindowWidth;
Without cleanup, each render could add new listeners and cause bugs; the cleanup keeps it safe and predictable.

7. Common Mistakes and Best Practices
7.1 Forgetting the Dependency Array
If you forget [] or specific dependencies, the effect may run more often than necessary.

This can cause repeated API calls or performance issues.

7.2 Missing Real Dependencies
The effect should include every state/prop it uses in the dependency array.

ESLint’s react-hooks/exhaustive-deps rule helps detect missing dependencies.

7.3 Including Unnecessary Items
Do not include stable functions like setState from useState—they never change and clutter the array.

Prefer useCallback or moving functions outside the component if you need stable identities.

7.4 Side Effects vs Rendering
Use useEffect for side effects, not for deriving values that can be computed directly during render.

Keep your components pure; avoid updating state in ways that cause infinite loops.

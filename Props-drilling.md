# Props Drilling in React

## What is Props Drilling?

**Props drilling** is the process of passing data (props) from a parent component down through multiple levels of child components, even when some intermediate components don't need the data themselves. They simply pass it along to deeper children [web:3][web:4].

```jsx
// Example of Prop Drilling
function App() {
  const [colour] = useState('blue');
  const [language] = useState('en');
  
  return <Menu colour={colour} language={language} />;
}

function Menu(props) {
  return (
    <MenuItem
      colour={props.colour}
      language={props.language}
    />
  );
}

function MenuItem(props) {
  return (
    <div>
      <p>Theme colour: {props.colour}</p>
      <p>Locale: {props.language}</p>
    </div>
  );
}
```

In this example, `Menu` doesn't use `colour` or `language`, but still has to pass them down [web:1].

## Problems with Props Drilling

| Issue | Description |
|-------|-------------|
| **Tight Coupling** | Intermediate components become dependent on props they don't use |
| **Hard to Maintain** | Adding new props requires updating all intermediate components |
| **Code Verbosity** | More boilerplate code to pass props through layers |
| **Reduced Reusability** | Components become harder to reuse in different contexts |

## Solutions to Avoid Props Drilling

### 1. Context API (Most Common)

React's built-in **Context API** allows you to share values across the component tree without explicitly passing props [web:4][web:8]:

```jsx
import React, { createContext, useContext } from 'react';

const UserContext = createContext();

export default function App() {
  const userData = { name: 'Tejas', age: 26 };
  
  return (
    <UserContext.Provider value={userData}>
      <Parent />
    </UserContext.Provider>
  );
}

function Button() {
  const userData = useContext(UserContext);
  return <div>{userData.name}</div>;
}
```

### 2. Use `children` Prop

By passing `children` to a container component, you free it from understanding its children's props [web:2]:

```jsx
function Container({ children }) {
  return <div className="container">{children}</div>;
}

function App() {
  return (
    <Container>
      <Child data={someData} />
    </Container>
  );
}
```

### 3. State Management Libraries

For larger apps, consider global state libraries [web:4]:

- **Redux**
- **Zustand**
- **Recoil**
- **Jotai**

```jsx
// Example with Zustand
import create from 'zustand';

const useStore = create(set => ({
  userData: { name: 'Tejas', age: 26 }
}));

function Button() {
  const userData = useStore(state => state.userData);
  return <div>{userData.name}</div>;
}
```

### 4. Merge Props into Single Object

Combine related props into one object to reduce intermediate component awareness [web:2]:

```jsx
// Instead of passing separately:
<Menu colour={colour} language={language} />

// Pass as single object:
<Menu config={{ colour, language }} />
```

## When is Props Drilling Okay?

✅ **Props drilling is fine when:**
- Building **small apps** or **simple UIs**
- Component tree is **shallow** (1-2 levels)
- Props are used by most components in the chain
- You want to avoid over-engineering [web:4]

❌ **Avoid props drilling when:**
- Component tree is **deep** (3+ levels)
- Many intermediate components don't use the props
- You frequently add/remove props
- Components become hard to maintain



## Conclusion

Props drilling is a natural side-effect of building UIs with nested components in React. While it's acceptable for small apps, as your application grows, solving it early with **Context API** or **global state management** will save you significant maintenance headaches [web:3][web:4].

---


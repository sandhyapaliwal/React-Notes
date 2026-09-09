# useTheme Hook in React

Unlike `useState` or `useEffect`, **`useTheme` is not a built-in React hook**. It's a pattern (usually built with the Context API) — or a hook provided by UI libraries like **styled-components**, **MUI (Material UI)**, or **Chakra UI** — used to read and toggle theme values (colors, spacing, dark/light mode) anywhere in a component tree without prop drilling.


## Table of Contents

- [Why useTheme?](#why-usetheme)
- [Building a Custom useTheme Hook](#building-a-custom-usetheme-hook)
  - [Step 1: Create the Theme Context](#step-1-create-the-theme-context)
  - [Step 2: Create the Provider](#step-2-create-the-provider)
  - [Step 3: Create the useTheme Hook](#step-3-create-the-usetheme-hook)
  - [Step 4: Wrap Your App](#step-4-wrap-your-app)
  - [Step 5: Consume the Theme](#step-5-consume-the-theme)
- [Example: Persisting Theme in localStorage](#example-persisting-theme-in-localstorage)
- [Example: Respecting System Preference](#example-respecting-system-preference)
- [useTheme in Popular Libraries](#usetheme-in-popular-libraries)
 

## Why useTheme?

Without a shared theme mechanism, you'd have to pass theme values (or a `darkMode` boolean) down through every component as props — classic **prop drilling**. A `useTheme` hook backed by Context solves this by:

- Making theme values (colors, fonts, spacing) available to **any component**, at any depth, with one line.
- Centralizing dark/light mode (or multi-theme) logic in one place.
- Keeping components decoupled from *how* the theme is stored or toggled.

## Building a Custom useTheme Hook

### Step 1: Create the Theme Context

```jsx
// ThemeContext.js
import { createContext } from "react";

export const themes = {
  light: {
    background: "#ffffff",
    text: "#1a1a1a",
    primary: "#1f3864",
  },
  dark: {
    background: "#121212",
    text: "#f5f5f5",
    primary: "#7ea8ff",
  },
};

export const ThemeContext = createContext(null);
```

### Step 2: Create the Provider

```jsx
// ThemeProvider.jsx
import { useState, useMemo, useCallback } from "react";
import { ThemeContext, themes } from "./ThemeContext";

export function ThemeProvider({ children }) {
  const [mode, setMode] = useState("light");

  const toggleTheme = useCallback(() => {
    setMode((prev) => (prev === "light" ? "dark" : "light"));
  }, []);

  const value = useMemo(
    () => ({
      mode,
      colors: themes[mode],
      toggleTheme,
    }),
    [mode, toggleTheme]
  );

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}
```

### Step 3: Create the useTheme Hook

```jsx
// useTheme.js
import { useContext } from "react";
import { ThemeContext } from "./ThemeContext";

export function useTheme() {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }

  return context;
}
```

Throwing an error when the context is missing helps catch a common mistake early — using the hook outside the provider — instead of silently returning `undefined`.

### Step 4: Wrap Your App

```jsx
// App.jsx
import { ThemeProvider } from "./ThemeProvider";
import Home from "./Home";

function App() {
  return (
    <ThemeProvider>
      <Home />
    </ThemeProvider>
  );
}

export default App;
```

### Step 5: Consume the Theme

```jsx
// Home.jsx
import { useTheme } from "./useTheme";

function Home() {
  const { mode, colors, toggleTheme } = useTheme();

  return (
    <div style={{ background: colors.background, color: colors.text, padding: "2rem" }}>
      <h1>Current mode: {mode}</h1>
      <button
        onClick={toggleTheme}
        style={{ background: colors.primary, color: "#fff", padding: "0.5rem 1rem" }}
      >
        Toggle Theme
      </button>
    </div>
  );
}

export default Home;
```

## Example: Persisting Theme in localStorage

Combine `useTheme` with `localStorage` so the user's preference survives page reloads.

```jsx
import { useState, useMemo, useCallback, useEffect } from "react";
import { ThemeContext, themes } from "./ThemeContext";

export function ThemeProvider({ children }) {
  const [mode, setMode] = useState(() => localStorage.getItem("theme") || "light");

  useEffect(() => {
    localStorage.setItem("theme", mode);
  }, [mode]);

  const toggleTheme = useCallback(() => {
    setMode((prev) => (prev === "light" ? "dark" : "light"));
  }, []);

  const value = useMemo(() => ({ mode, colors: themes[mode], toggleTheme }), [mode, toggleTheme]);

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}
```

## Example: Respecting System Preference

Default to the user's OS-level dark/light preference using `window.matchMedia`.

```jsx
const getInitialMode = () => {
  const saved = localStorage.getItem("theme");
  if (saved) return saved;

  const prefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
  return prefersDark ? "dark" : "light";
};

// used inside ThemeProvider:
const [mode, setMode] = useState(getInitialMode);
```

## useTheme in Popular Libraries

If your project already uses a styling library, you likely don't need to build your own — a `useTheme` hook is often included.

### styled-components

```bash
npm install styled-components
```

```jsx
import { ThemeProvider, useTheme } from "styled-components";

const theme = {
  colors: { primary: "#1f3864", background: "#ffffff" },
};

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Banner />
    </ThemeProvider>
  );
}

function Banner() {
  const theme = useTheme(); // reads the theme object provided above
  return <div style={{ color: theme.colors.primary }}>Styled with theme</div>;
}


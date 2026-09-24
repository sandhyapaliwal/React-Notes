# Portals in React

A **Portal** in React lets you render a child component into a DOM node that exists **outside** the DOM hierarchy of its parent component — while still keeping it inside the same React component tree (for context, event bubbling, and state).

They're the standard solution for UI elements that need to visually "break out" of their parent's layout — modals, tooltips, dropdowns, toasts, and popovers.

## Table of Contents

- [Why Portals?](#why-portals)
- [The Problem Without Portals](#the-problem-without-portals)
- [Basic Syntax](#basic-syntax)
- [Example 1: A Simple Modal](#example-1-a-simple-modal)
- [Example 2: Reusable Modal Component](#example-2-reusable-modal-component)
- [Example 3: Toast/Notification Portal](#example-3-toastnotification-portal)
- [Event Bubbling Through Portals](#event-bubbling-through-portals)
- [Portals with TypeScript](#portals-with-typescript)


## Why Portals?

Normally, a component's rendered output is nested inside its parent's DOM node, following the same structure as the component tree. This works fine for most UI, but breaks down for elements like modals or tooltips when:

- A parent has `overflow: hidden` or `overflow: auto`, which clips the child visually.
- A parent has a `z-index` or `transform` that creates a new stacking context, causing the child to appear behind other elements even with a high `z-index` of its own.
- You need the element to sit directly under `<body>` for correct positioning (e.g., a modal that should cover the entire viewport).

Portals solve this by rendering the child into a completely different DOM node, chosen by you — while the component still logically belongs to its place in the React tree.

## The Problem Without Portals

```jsx
// Without a portal — the modal is trapped inside .app-container
function App() {
  return (
    <div className="app-container" style={{ overflow: "hidden", position: "relative" }}>
      <Modal>Look at me!</Modal>
    </div>
  );
}
```

If `.app-container` has `overflow: hidden`, the modal gets visually clipped no matter how high its `z-index` is — because it's still a DOM descendant of that container.

## Basic Syntax

```jsx
import { createPortal } from "react-dom";

createPortal(children, domNode);
```

- **`children`** — any renderable React content (elements, strings, fragments).
- **`domNode`** — the actual DOM element to render into (usually obtained via `document.getElementById(...)`).

## Example 1: A Simple Modal

First, add a target node in your HTML file (outside the main app root):

```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
</body>
```

Then create the portal:

```jsx
import { createPortal } from "react-dom";

function Modal({ children }) {
  const modalRoot = document.getElementById("modal-root");
  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">{children}</div>
    </div>,
    modalRoot
  );
}
```

```jsx
function App() {
  return (
    <div className="app-container">
      <h1>My App</h1>
      <Modal>
        <p>This renders outside .app-container in the DOM!</p>
      </Modal>
    </div>
  );
}
```

Even though `<Modal>` is written inside `<App>` in JSX, its actual DOM output lands inside `#modal-root`, completely bypassing any clipping or stacking issues from `.app-container`.

## Example 2: Reusable Modal Component

A more complete, reusable modal with open/close control and an overlay click-to-close.

```jsx
import { createPortal } from "react-dom";
import { useEffect } from "react";

function Modal({ isOpen, onClose, children }) {
  useEffect(() => {
    function handleEscape(e) {
      if (e.key === "Escape") onClose();
    }
    if (isOpen) document.addEventListener("keydown", handleEscape);
    return () => document.removeEventListener("keydown", handleEscape);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div
      className="modal-overlay"
      onClick={onClose}
      style={{
        position: "fixed", inset: 0, background: "rgba(0,0,0,0.5)",
        display: "flex", alignItems: "center", justifyContent: "center",
      }}
    >
      <div
        className="modal-content"
        onClick={(e) => e.stopPropagation()} // prevent overlay close on inner click
        style={{ background: "#fff", padding: "1.5rem", borderRadius: "8px", minWidth: "300px" }}
      >
        {children}
        <button onClick={onClose} style={{ marginTop: "1rem" }}>Close</button>
      </div>
    </div>,
    document.getElementById("modal-root")
  );
}

export default Modal;
```

**Usage:**

```jsx
import { useState } from "react";
import Modal from "./Modal";

function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      <Modal isOpen={isOpen} onClose={() => setIsOpen(false)}>
        <h2>Confirm Action</h2>
        <p>Are you sure you want to proceed?</p>
      </Modal>
    </div>
  );
}
```

## Example 3: Toast/Notification Portal

Portals are also ideal for toast notifications that should always render above everything else, regardless of where they're triggered from in the component tree.

```jsx
import { createPortal } from "react-dom";

function Toast({ message, type = "info" }) {
  return createPortal(
    <div className={`toast toast-${type}`} style={{ position: "fixed", bottom: "20px", right: "20px" }}>
      {message}
    </div>,
    document.getElementById("toast-root")
  );
}
```

```jsx
function SaveButton() {
  const [showToast, setShowToast] = useState(false);

  const handleSave = () => {
    // save logic...
    setShowToast(true);
    setTimeout(() => setShowToast(false), 3000);
  };

  return (
    <>
      <button onClick={handleSave}>Save</button>
      {showToast && <Toast message="Saved successfully!" type="success" />}
    </>
  );
}
```

## Event Bubbling Through Portals

A key detail: even though a portal's DOM node lives elsewhere in the actual document, **React events still bubble up through the React component tree, not the DOM tree**. So an event fired inside a portal will still be caught by an `onClick` handler on a parent component in React's tree — even though, in raw DOM terms, the portal's node isn't a descendant of that parent.

```jsx
function App() {
  return (
    // This onClick will fire even for clicks inside the portal's modal content
    <div onClick={() => console.log("Parent div clicked")}>
      <Modal isOpen={true} onClose={() => {}}>
        <button onClick={() => console.log("Button inside portal clicked")}>
          Click me
        </button>
      </Modal>
    </div>
  );
}

// Clicking the button logs both:
// "Button inside portal clicked"
// "Parent div clicked"
```

This is intentional and matches React's conceptual model — portals change *where* something renders in the DOM, not its logical place in the component tree.

## Portals with TypeScript

```tsx
import { createPortal } from "react-dom";
import { ReactNode } from "react";

interface ModalProps {
  children: ReactNode;
  isOpen: boolean;
  onClose: () => void;
}

function Modal({ children, isOpen, onClose }: ModalProps) {
  if (!isOpen) return null;

  const modalRoot = document.getElementById("modal-root");
  if (!modalRoot) return null; // guard against a missing target node

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      {children}
    </div>,
    modalRoot
  );
}

export default Modal;
```


# React Fragments

A **Fragment** lets you group a list of children without adding an extra node to the DOM. It's one of the simplest but most useful features in React, solving a very common problem: a component must return a single root element, but you don't always want that element to leave a mark in the rendered HTML.

## Table of Contents

- [The Problem Fragments Solve](#the-problem-fragments-solve)
- [Basic Usage](#basic-usage)
- [Shorthand Syntax](#shorthand-syntax)
- [Fragment vs Shorthand: When to Use Which](#fragment-vs-shorthand-when-to-use-which)
- [Example 1: Returning Multiple Elements](#example-1-returning-multiple-elements)
- [Example 2: Fragments in a List (with `key`)](#example-2-fragments-in-a-list-with-key)
- [Example 3: Table Rows Without Wrapper Divs](#example-3-table-rows-without-wrapper-divs)
- [Example 4: Conditional Rendering with Fragments](#example-4-conditional-rendering-with-fragments)
- [Why Not Just Use a `<div>`?](#why-not-just-use-a-div)

## The Problem Fragments Solve

In React, a component's `return` statement must resolve to a single element. Before Fragments existed, the only way to return multiple sibling elements was to wrap them in a container like a `<div>`:

```jsx
// Works, but adds an unnecessary <div> to the DOM
function UserInfo() {
  return (
    <div>
      <h2>Sandhya Paliwal</h2>
      <p>Frontend Developer</p>
    </div>
  );
}
```

That extra `<div>` isn't just noise — it can break CSS layouts (e.g., in a `<table>` or CSS Grid/Flexbox structure) that expect specific parent-child relationships. Fragments let you group elements **without** adding this extra node.

## Basic Usage

```jsx
import { Fragment } from "react";

function UserInfo() {
  return (
    <Fragment>
      <h2>Sandhya Paliwal</h2>
      <p>Frontend Developer</p>
    </Fragment>
  );
}
```

The rendered HTML contains only the `<h2>` and `<p>` — no wrapping element at all.

## Shorthand Syntax

Most of the time, you'll use the shorthand `<>...</>` syntax instead of importing `Fragment` explicitly:

```jsx
function UserInfo() {
  return (
    <>
      <h2>Sandhya Paliwal</h2>
      <p>Frontend Developer</p>
    </>
  );
}
```

This is functionally identical to using `<Fragment>` — it's just less to type and doesn't require an import.

## Fragment vs Shorthand: When to Use Which

| Feature | `<Fragment>` | `<>...</>` (shorthand) |
|---|---|---|
| Import required | Yes — `import { Fragment } from "react"` | No |
| Supports `key` prop | Yes | No |
| Supports other props | Yes (`key` only, in practice) | No |
| Typical use | Rendering lists that need a `key` | Everyday grouping of elements |

**Rule of thumb:** use the shorthand `<>...</>` by default, and switch to the explicit `<Fragment>` only when you need to pass a `key` (e.g., inside a `.map()`).

## Example 1: Returning Multiple Elements

```jsx
function ProfileHeader() {
  return (
    <>
      <img src="/avatar.png" alt="User avatar" />
      <h1>Sandhya Paliwal</h1>
      <p>Frontend Developer specializing in React.js</p>
    </>
  );
}
```

## Example 2: Fragments in a List (with `key`)

When rendering a list of grouped elements with `.map()`, each Fragment needs a unique `key`, which requires the explicit `Fragment` import.

```jsx
import { Fragment } from "react";

const faqs = [
  { id: 1, question: "What is React?", answer: "A JavaScript library for building UIs." },
  { id: 2, question: "What is a Hook?", answer: "A function that lets you use state and other features." },
];

function FAQList() {
  return (
    <dl>
      {faqs.map((faq) => (
        <Fragment key={faq.id}>
          <dt>{faq.question}</dt>
          <dd>{faq.answer}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

The shorthand `<>...</>` **cannot** take a `key`, so this pattern always needs the full `<Fragment>` syntax.

## Example 3: Table Rows Without Wrapper Divs

Fragments are especially useful for HTML elements with strict parent-child rules, like tables — a `<div>` between `<table>` and `<tr>` is invalid HTML and breaks rendering.

```jsx
function TableRowGroup({ item }) {
  return (
    <>
      <tr>
        <td>{item.name}</td>
        <td>{item.price}</td>
      </tr>
      <tr>
        <td colSpan={2}>{item.description}</td>
      </tr>
    </>
  );
}

function ProductTable({ items }) {
  return (
    <table>
      <tbody>
        {items.map((item) => (
          <TableRowGroup key={item.id} item={item} />
        ))}
      </tbody>
    </table>
  );
}
```

## Example 4: Conditional Rendering with Fragments

Fragments are handy when conditionally rendering multiple elements together without introducing a wrapping node.

```jsx
function Notification({ type, message }) {
  return (
    <>
      {type === "error" && (
        <>
          <strong>Error:</strong> <span>{message}</span>
        </>
      )}
      {type === "success" && (
        <>
          <strong>Success:</strong> <span>{message}</span>
        </>
      )}
    </>
  );
}
```

## Why Not Just Use a `<div>`?

A `<div>` seems like an easy fix, but it has real downsides:

- **Breaks CSS layouts** — Flexbox and Grid apply rules to *direct children*; an unwanted `<div>` changes which elements count as direct children.
- **Invalid HTML in certain contexts** — e.g., a `<div>` cannot legally sit between `<table>` and `<tr>`, or inside `<select>` around `<option>`.
- **Unnecessary DOM nodes** — more nodes means a (slightly) larger DOM tree, which can affect performance in large, deeply nested UIs.
- **Loses semantic meaning** — wrapping unrelated elements in a generic `<div>` adds no meaning and can clutter accessibility trees.


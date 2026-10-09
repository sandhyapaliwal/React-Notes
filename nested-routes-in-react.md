# Nested Routes in React Router

Nested routes let you render child components inside a parent layout, based on the URL — so a shared layout (navbar, sidebar, tabs) stays mounted while only the inner content changes as the user navigates. This is one of the core patterns in **React Router v6+**.

## Table of Contents

- [Why Nested Routes?](#why-nested-routes)
- [The Outlet Component](#the-outlet-component)
- [Basic Example](#basic-example)
- [Example 1: Dashboard Layout with Nested Pages](#example-1-dashboard-layout-with-nested-pages)
- [Example 2: Index Routes](#example-2-index-routes)
- [Example 3: Nested Routes with Dynamic Params](#example-3-nested-routes-with-dynamic-params)
- [Example 4: Multiple Levels of Nesting](#example-4-multiple-levels-of-nesting)
- [Example 5: Passing Context Through Outlet](#example-5-passing-context-through-outlet)
- [Defining Nested Routes with createBrowserRouter](#defining-nested-routes-with-createbrowserrouter)

## Why Nested Routes?

Without nesting, every page of a dashboard-style app would need to re-import and re-render the same navbar, sidebar, and footer. Nested routes solve this by letting a parent route render a shared layout once, with child routes filling in just the part of the page that changes.

Benefits:

- **Shared layouts** (navbar, sidebar, tabs) stay mounted across child navigations — no re-render, no flicker.
- **URLs map naturally to UI structure** — `/settings/profile` and `/settings/billing` both live inside a `/settings` layout.
- **Co-located data loading** — each nested route can load only the data it needs (especially with loaders in `createBrowserRouter`).

## The Outlet Component

The key piece that makes nesting work is `<Outlet />` — it's a placeholder that tells React Router where to render the matched child route inside the parent's JSX.

```jsx
import { Outlet } from "react-router-dom";

function Layout() {
  return (
    <div>
      <nav>My Navbar</nav>
      <main>
        <Outlet /> {/* child route renders here */}
      </main>
    </div>
  );
}
```

## Basic Example

```jsx
import { Routes, Route, Outlet, Link } from "react-router-dom";

function Layout() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link> | <Link to="about">About</Link>
      </nav>
      <hr />
      <Outlet />
    </div>
  );
}

function Home() {
  return <h2>Home Page</h2>;
}

function About() {
  return <h2>About Page</h2>;
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
      </Route>
    </Routes>
  );
}

export default App;
```

Here, `Layout` always renders (navbar + `Outlet`), while `Home` or `About` renders inside the `Outlet` depending on the URL.

## Example 1: Dashboard Layout with Nested Pages

```jsx
import { Routes, Route, Outlet, Link } from "react-router-dom";

function DashboardLayout() {
  return (
    <div style={{ display: "flex" }}>
      <aside>
        <Link to="overview">Overview</Link>
        <Link to="reports">Reports</Link>
        <Link to="settings">Settings</Link>
      </aside>
      <section>
        <Outlet />
      </section>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route path="overview" element={<Overview />} />
        <Route path="reports" element={<Reports />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}
```

Visiting `/dashboard/reports` renders `DashboardLayout` (sidebar always visible), with `Reports` filled into the `Outlet`.

## Example 2: Index Routes

An **index route** renders at the parent's own path, when no child segment is specified — useful for a default/landing view inside a layout.

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<Overview />} /> {/* renders at exactly /dashboard */}
  <Route path="reports" element={<Reports />} />
</Route>
```

Visiting `/dashboard` (no further segment) renders `Overview` inside the `Outlet`. Visiting `/dashboard/reports` renders `Reports` instead.

## Example 3: Nested Routes with Dynamic Params

Params defined on a parent route remain accessible to nested children via `useParams`.

```jsx
<Route path="/teams/:teamId" element={<TeamLayout />}>
  <Route index element={<TeamOverview />} />
  <Route path="members" element={<TeamMembers />} />
  <Route path="members/:memberId" element={<MemberDetail />} />
</Route>
```

```jsx
import { useParams } from "react-router-dom";

function MemberDetail() {
  const { teamId, memberId } = useParams();
  // Both teamId and memberId are available here

  return (
    <p>
      Team {teamId}, Member {memberId}
    </p>
  );
}
```

## Example 4: Multiple Levels of Nesting

Routes can nest as deeply as your UI needs — each level adds its own `<Outlet />`.

```jsx
<Route path="/app" element={<AppLayout />}>
  <Route path="projects" element={<ProjectsLayout />}>
    <Route index element={<ProjectsList />} />
    <Route path=":projectId" element={<ProjectLayout />}>
      <Route index element={<ProjectOverview />} />
      <Route path="tasks" element={<ProjectTasks />} />
    </Route>
  </Route>
</Route>
```

For `/app/projects/42/tasks` to render correctly, `AppLayout`, `ProjectsLayout`, and `ProjectLayout` each need their own `<Outlet />` so the next level down has somewhere to render.

## Example 5: Passing Context Through Outlet

`<Outlet />` accepts a `context` prop, letting a parent layout pass data down to whichever child is currently rendered, without prop drilling through route config.

```jsx
function DashboardLayout() {
  const [user, setUser] = useState({ name: "Sandhya" });

  return (
    <div>
      <aside>{/* nav links */}</aside>
      <Outlet context={{ user }} />
    </div>
  );
}
```

```jsx
import { useOutletContext } from "react-router-dom";

function Overview() {
  const { user } = useOutletContext();
  return <h2>Welcome, {user.name}</h2>;
}
```

## Defining Nested Routes with createBrowserRouter

Modern React Router apps (v6.4+) often use `createBrowserRouter` with a `children` array instead of JSX `<Routes>`. Nesting works the same way, but as plain objects:

```jsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/dashboard",
    element: <DashboardLayout />,
    children: [
      { index: true, element: <Overview /> },
      { path: "reports", element: <Reports /> },
      { path: "settings", element: <Settings /> },
    ],
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

This style also supports per-route `loader` and `action` functions for data fetching, which pairs naturally with nested layouts.

# SPA vs MPA in ReactJS




## Single Page Application (SPA)

### What is a SPA?

A Single Page Application is a web application that dynamically rewrites the current page rather than loading entire new pages from the server. It loads one HTML page initially and then uses JavaScript to update the DOM as the user interacts with the app.

### How SPA Works with React

```jsx
// React Router enables SPA navigation
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import Contact from './pages/Contact';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### Characteristics of SPA

| Feature | Description |
|---------|-------------|
| **Initial Load** | Single HTML file with JavaScript bundle |
| **Navigation** | Client-side routing, no full page reload |
| **State Management** | Maintained in memory (Redux, Context API, Zustand) |
| **Server Role** | Primarily serves data via APIs (REST, GraphQL) |
| **URL Changes** | Handled by JavaScript, not server redirects |
| **Assets** | Bundled and loaded once |

### Advantages of SPA

- **Fast Navigation**: No full page reloads; instant transitions between views
- **Rich User Experience**: Smooth, app-like interactions without delays
- **Reduced Server Load**: Server only handles API requests, not page rendering
- **Offline Support**: Can cache resources and work partially offline with service workers
- **Real-time Updates**: Easy to implement live notifications and data synchronization
- **Better for Progressive Web Apps (PWA)**: Native app-like behavior
- **Efficient Data Transfer**: Only data is exchanged, not full HTML pages

### Disadvantages of SPA

- **Large Initial Bundle**: JavaScript bundle size can be significant
- **SEO Challenges**: Search engines may struggle with dynamically rendered content
- **Complex State Management**: Requires careful management of application state
- **Browser Back/Forward Issues**: Needs proper history management
- **Memory Leaks**: Potential for memory leaks if not properly managed
- **JavaScript Dependency**: Requires JavaScript to be enabled; fails gracefully without it
- **Cold Start Time**: Initial page load can be slow

---

## Multi Page Application (MPA)

### What is an MPA?

A Multi Page Application loads a new HTML page from the server for each view or route. Each page is a separate HTML document, and navigation causes a full page refresh with new content from the server.

### How React is Used in MPA

```jsx
// Each page is a separate React component rendered on the server
// Using Next.js with SSR (Server-Side Rendering)
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

export default function HomePage({ data }) {
  return <div>{/* Component content */}</div>;
}
```

### Characteristics of MPA

| Feature | Description |
|---------|-------------|
| **Initial Load** | Multiple HTML files, one per route |
| **Navigation** | Full page reload for each route change |
| **State Management** | Each page has its own state |
| **Server Role** | Renders HTML pages and serves data |
| **URL Changes** | Server handles routing and redirects |
| **Assets** | Loaded per page as needed |

### Advantages of MPA

- **SEO Friendly**: Each page has its own HTML, meta tags, and content (great for search engines)
- **Smaller Initial Payload**: Each page loads only what it needs
- **Better Browser Support**: Works well even with JavaScript disabled
- **Simpler Backend**: Traditional server-side routing is straightforward
- **Independent Pages**: Each page can be optimized independently
- **Scalability**: Easy to scale with multiple micro-frontends
- **Page-level Code Splitting**: JavaScript is split per page naturally

### Disadvantages of MPA

- **Slower Navigation**: Full page reloads cause noticeable delays
- **Poor User Experience**: Flickering, loading states, less app-like feel
- **Server Load**: Server renders every page request
- **Bandwidth**: Full HTML is transferred for each navigation
- **State Loss**: State is lost on each page reload
- **Complex Interactions**: Difficult to maintain state across pages
- **Development Complexity**: Managing shared state across pages is harder




## Key Differences

### Routing & Navigation

**SPA**: Client-side routing using libraries like React Router
```jsx
<Link to="/about">About</Link>
```

**MPA**: Server-side routing
```jsx
<a href="/about">About</a>
```

### State Management

**SPA**: Global state lives in Redux/Context API
```jsx
const user = useSelector(state => state.user);
```

**MPA**: Each page request starts fresh
```jsx
// State is passed as props from server
export async function getServerSideProps() {
  return { props: { user: userData } };
}
```

### Bundle Size

**SPA**: Larger initial JavaScript bundle
**MPA**: Multiple smaller bundles, one per page


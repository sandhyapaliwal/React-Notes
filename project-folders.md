# React Project Structure

A clean, scalable, and professional folder structure for React applications.

## 📂 Folder Descriptions

| Folder | Purpose |
|--------|---------|
| `public/` | Static assets served directly (HTML, favicon, manifest) |
| `src/components/` | Reusable UI components split by `common` and `features` |
| `src/pages/` | Page-level components corresponding to routes |
| `src/hooks/` | Custom React hooks for reusable logic |
| `src/services/` | API calls and external service integrations |
| `src/store/` | State management (Redux slices, store configuration) |
| `src/utils/` | Helper functions, utilities, and constants |
| `src/assets/` | Images, fonts, and global styles |
| `src/routes/` | Route configuration and protected routes |
| `src/context/` | React Context providers for global state |

## 🚀 Best Practices

1. **Feature-based organization**: Group related components, hooks, and services by feature
2. **Co-locate files**: Keep related files (component, styles, tests) in the same folder
3. **Index files**: Use `index.js` for clean imports (`import Button from './components/common/Button'`)
4. **Separate concerns**: Keep components, services, and utilities in distinct folders
5. **Scalability**: Structure supports growth from small to large applications

## 📝 Sample Component Structure

```jsx
// src/components/common/Button/Button.jsx
import React from 'react'
import styles from './Button.module.css'

const Button = ({ children, onClick, variant = 'primary' }) => {
  return (
    <button className={`${styles.button} ${styles[variant]}`} onClick={onClick}>
      {children}
    </button>
  )
}

export default Button
```

## 🔧 Setup Commands

```bash
# Create React app with Vite
npm create vite@latest react-project -- --template react

# Install dependencies
cd react-project
npm install

# Install common packages
npm install react-router-dom @reduxjs/toolkit react-redux axios

# Start development server
npm run dev

# Build for production
npm run build

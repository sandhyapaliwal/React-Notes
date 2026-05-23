# React Project Structure

A clean, scalable, and professional folder structure for React applications.

## 📁 Project Structure
react-project/
├── public/
│ ├── index.html
│ ├── favicon.ico
│ ├── logo192.png
│ ├── logo512.png
│ ├── manifest.json
│ └── robots.txt
│
├── src/
│ ├── components/ # Reusable UI components
│ │ ├── common/ # Generic components (Button, Input, Modal)
│ │ │ ├── Button/
│ │ │ │ ├── Button.jsx
│ │ │ │ ├── Button.module.css
│ │ │ │ └── index.js
│ │ │ └── Input/
│ │ │ ├── Input.jsx
│ │ │ ├── Input.module.css
│ │ │ └── index.js
│ │ └── features/ # Feature-specific components
│ │ ├── Header/
│ │ │ ├── Header.jsx
│ │ │ ├── Header.module.css
│ │ │ └── index.js
│ │ └── Footer/
│ │ ├── Footer.jsx
│ │ ├── Footer.module.css
│ │ └── index.js
│ │
│ ├── pages/ # Page-level components (route components)
│ │ ├── Home/
│ │ │ ├── Home.jsx
│ │ │ ├── Home.module.css
│ │ │ └── index.js
│ │ ├── About/
│ │ │ ├── About.jsx
│ │ │ ├── About.module.css
│ │ │ └── index.js
│ │ └── Contact/
│ │ ├── Contact.jsx
│ │ ├── Contact.module.css
│ │ └── index.js
│ │
│ ├── hooks/ # Custom React hooks
│ │ ├── useAuth.js
│ │ ├── useFetch.js
│ │ └── useLocalStorage.js
│ │
│ ├── services/ # API calls and external services
│ │ ├── api.js
│ │ ├── authService.js
│ │ └── userService.js
│ │
│ ├── store/ # State management (Redux/Zustand/Context)
│ │ ├── slices/
│ │ │ ├── authSlice.js
│ │ │ └── userSlice.js
│ │ ├── store.js
│ │ └── index.js
│ │
│ ├── utils/ # Helper functions and utilities
│ │ ├── formatDate.js
│ │ ├── validate.js
│ │ └── constants.js
│ │
│ ├── assets/ # Images, fonts, styles
│ │ ├── images/
│ │ ├── fonts/
│ │ └── styles/
│ │ ├── global.css
│ │ └── variables.css
│ │
│ ├── routes/ # Route configuration
│ │ ├── index.js
│ │ └── PrivateRoute.jsx
│ │
│ ├── context/ # React Context providers
│ │ ├── AuthContext.jsx
│ │ └── ThemeContext.jsx
│ │
│ ├── App.jsx # Main App component
│ ├── App.css # App styles
│ ├── index.jsx # Entry point
│ ├── index.css # Global styles
│ └── reportWebVitals.js
│
├── .env # Environment variables
├── .env.example # Example environment variables
├── .gitignore # Git ignore rules
├── package.json # Dependencies and scripts
├── package-lock.json # Locked dependencies
├── README.md # Project documentation
├── eslint.config.js # ESLint configuration
├── prettier.config.js # Prettier configuration
└── vite.config.js # Vite configuration (if using Vite)

text

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

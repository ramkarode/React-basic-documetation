# React Context API — Complete Guide
> From Beginner to Advanced | Modern React 18+ Practices | Interview-Ready

---

## Table of Contents

1. [Introduction to State Management](#1-introduction-to-state-management)
2. [Understanding Prop Drilling](#2-understanding-prop-drilling)
3. [Why Context API Exists](#3-why-context-api-exists)
4. [Context API Architecture](#4-context-api-architecture)
5. [Understanding createContext()](#5-understanding-createcontext)
6. [Provider Components](#6-provider-components)
7. [Understanding Children Props](#7-understanding-children-props)
8. [Understanding useContext()](#8-understanding-usecontext)
9. [Custom Hook Pattern](#9-custom-hook-pattern)
10. [Building a Theme Switcher](#10-building-a-theme-switcher-application)
11. [Building an Authentication System](#11-building-an-authentication-system)
12. [Building a Shopping Cart System](#12-building-a-shopping-cart-system)
13. [Provider Composition Pattern](#13-provider-composition-pattern)
14. [Context Folder Structure](#14-context-folder-structure)
15. [Context Module Pattern](#15-context-module-pattern)
16. [Performance Optimization](#16-performance-optimization)
17. [Context API vs Redux Toolkit](#17-context-api-vs-redux-toolkit)
18. [Common Mistakes](#18-common-mistakes)
19. [Legacy Consumer Pattern](#19-legacy-consumer-pattern-brief)
20. [Real-World Architectures](#20-real-world-architectures)
21. [Context API Best Practices](#21-context-api-best-practices)
22. [Interview Questions](#22-interview-questions)
23. [Context API Cheat Sheet](#23-context-api-cheat-sheet)
24. [Complete Production-Ready Project](#24-complete-production-ready-project)

---

## 1. Introduction to State Management

### What is State?

State is data that changes over time and controls what a component renders. When state changes, React re-renders the affected component and its children to reflect the update.

```jsx
// Local state example
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

State is the "memory" of your application — things like:
- Whether a user is logged in
- Items in a shopping cart
- The current theme (dark/light)
- Form input values
- API response data

### Local State

Local state lives inside a single component and is managed with `useState` or `useReducer`. It is not visible to other components unless explicitly passed down.

**When to use local state:**
- Toggle visibility of a dropdown
- Form field values before submission
- Hover/focus states
- Pagination in a self-contained table

```jsx
function SearchBox() {
  const [query, setQuery] = useState('');

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### Global State

Global state is data shared across multiple components, often across different parts of the component tree. Examples include:
- Currently logged-in user
- Application theme
- Language/locale preference
- Shopping cart contents
- Notification messages

### Component Communication

React components communicate through **props** (parent → child) and **callbacks** (child → parent via function props). When unrelated components need the same data, this becomes difficult.

```
Parent
  ├── ChildA  (needs user data)
  └── ChildB
        └── ChildC  (also needs user data)
```

Passing user data from Parent to both ChildA and deep-nested ChildC requires passing it through ChildB even though ChildB doesn't use it — this is **prop drilling**.

### State Sharing Problems

Real-world problem scenario: An e-commerce site where the **Header** shows the cart item count, the **ProductCard** has an "Add to Cart" button, and the **CartDrawer** shows the full cart. All three components need access to the same cart state — but they are siblings or even cousins in the tree.

Without a global state solution, you would be forced to:
1. Lift the cart state to a common ancestor
2. Pass it down through every intermediate component
3. Pass setter functions back up through multiple layers

This is messy, fragile, and hard to maintain.

---

## 2. Understanding Prop Drilling

### What is Prop Drilling?

Prop drilling is the practice of passing data through multiple component layers just to get it where it needs to go — even when intermediate components don't need that data themselves.

### Visual Diagram

```
App (owns userProfile)
 │
 └── Dashboard (passes userProfile down)
       │
       └── Sidebar (passes userProfile down)
             │
             └── UserCard (FINALLY uses userProfile)
```

Code that illustrates this problem:

```jsx
// App.jsx
function App() {
  const [userProfile, setUserProfile] = useState({
    name: 'Rahul Sharma',
    email: 'rahul@example.com',
    role: 'admin',
  });

  return <Dashboard userProfile={userProfile} />;
}

// Dashboard.jsx — doesn't use userProfile, just passes it
function Dashboard({ userProfile }) {
  return <Sidebar userProfile={userProfile} />;
}

// Sidebar.jsx — doesn't use userProfile, just passes it
function Sidebar({ userProfile }) {
  return <UserCard userProfile={userProfile} />;
}

// UserCard.jsx — finally uses it
function UserCard({ userProfile }) {
  return (
    <div>
      <h2>{userProfile.name}</h2>
      <p>{userProfile.email}</p>
    </div>
  );
}
```

Dashboard and Sidebar are polluted with a prop they never use.

### Why It Happens

Prop drilling happens because React's data flow is unidirectional (top-down). The only built-in mechanism for sharing data is passing props. When components that need data are far apart in the tree, all intermediate components must act as couriers.

### Problems in Large Applications

In a large application, prop drilling causes:

| Problem | Impact |
|---|---|
| Prop pollution | Components receive props they never use |
| Tight coupling | Renaming or restructuring breaks many files |
| Hard refactoring | Moving a component breaks the data chain |
| Testing difficulty | Mocks must include irrelevant props |
| Readability | Component signatures become huge |

### Maintenance Challenges

Consider a 5-level-deep component tree. If you add a new field to `userProfile`, you must update the prop type in every intermediate file. If you rename `userProfile` to `currentUser`, you break the chain everywhere. This is unsustainable as teams and codebases grow.

---

## 3. Why Context API Exists

### Motivation Behind Context API

React's Context API was designed specifically to solve prop drilling. It provides a way to share data across the component tree without explicitly passing props at every level.

> "Context provides a way to pass data through the component tree without having to pass props down manually at every level." — React Docs

### Solving Prop Drilling

With Context, any component in the tree can subscribe to shared data directly, regardless of how deep it is:

```
App
 └── (Context.Provider wraps everything)
       ├── Dashboard          ← can read context
       │     └── Sidebar      ← can read context
       │           └── UserCard  ← can read context directly
       └── Header             ← can read context
```

No intermediate passing needed.

### Global Data Sharing

Context is ideal for data that is "global" to a component subtree — data that many components at different levels need to read or update.

### Common Use Cases

**Authentication:**
```jsx
// Any component can check who's logged in
const { user, isLoggedIn, logout } = useAuth();
```

**Theme Switching:**
```jsx
// Any component can read or toggle the theme
const { theme, toggleTheme } = useTheme();
```

**User Preferences:**
```jsx
// Language, timezone, notification settings
const { locale, setLocale } = usePreferences();
```

**Shopping Cart:**
```jsx
// Add to cart from any product component
const { cart, addToCart, removeFromCart } = useCart();
```

---

## 4. Context API Architecture

### Core Building Blocks

The Context API has four key pieces:

| Piece | Purpose |
|---|---|
| `createContext()` | Creates the context object |
| `Provider` | Wraps the component tree and supplies values |
| `children` prop | Allows the Provider to wrap arbitrary components |
| `useContext()` | Reads context value inside any functional component |
| Custom Hook | Wraps `useContext` for a better developer experience |

### Architecture Diagram

```
┌─────────────────────────────────────────────┐
│  createContext()                             │
│  Returns: { Provider, Consumer, _currentValue} │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  <ThemeContext.Provider value={...}>         │
│    ┌─────────────────────────────────────┐   │
│    │  App (children)                      │   │
│    │    ├── Header                        │   │
│    │    │     └── ThemeToggle  ◄──reads   │   │
│    │    └── Main                          │   │
│    │          └── ProductCard  ◄──reads   │   │
│    └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  useContext(ThemeContext)                    │
│  Returns the current value from Provider    │
└─────────────────────────────────────────────┘
```

### Data Flow

1. You call `createContext()` and get back a context object.
2. You wrap part of your component tree with `<YourContext.Provider value={...}>`.
3. Any component inside that tree calls `useContext(YourContext)` to read the value.
4. When the `value` prop on Provider changes, all subscribed components re-render.

---

## 5. Understanding createContext()

### Syntax

```jsx
import { createContext } from 'react';

const ThemeContext = createContext(defaultValue);
```

`createContext` accepts one argument — the **default value**. This value is used only when a component reads the context but has no matching Provider above it in the tree.

### What createContext() Returns

```jsx
const ThemeContext = createContext('light');

// ThemeContext is an object with:
// ThemeContext.Provider  — component used to supply values
// ThemeContext.Consumer  — legacy class component API (rarely used now)
// ThemeContext.displayName — for React DevTools labeling
```

### Default Values in Practice

```jsx
// String default
const ThemeContext = createContext('light');

// Object default (recommended for complex contexts)
const AuthContext = createContext({
  user: null,
  isLoggedIn: false,
  login: () => {},
  logout: () => {},
});

// Null default (when there's always a Provider present)
const CartContext = createContext(null);
```

**Best practice:** Use meaningful default values that match the shape of the real value. This helps with TypeScript typing and makes the context usable even without a Provider in tests.

### Setting displayName

```jsx
const ThemeContext = createContext('light');
ThemeContext.displayName = 'ThemeContext'; // Shown in React DevTools
```

### Multiple Contexts

You can create as many contexts as you need — one per concern:

```jsx
export const AuthContext = createContext(null);
export const ThemeContext = createContext('light');
export const CartContext = createContext(null);
export const NotificationContext = createContext([]);
```

### Best Practices for createContext

- Export the context object so components can import it
- Keep one context per file for clarity
- Use TypeScript generics to type the context value
- Always provide meaningful defaults
- Name contexts clearly: `AuthContext`, not `Context1`

---

## 6. Provider Components

### What Providers Are

A Provider is a React component that wraps a part of your component tree and makes context values available to all descendants. Every context object returned by `createContext()` includes a built-in `Provider` component.

### How Providers Work

The Provider accepts a `value` prop. Whatever you pass as `value` becomes available to all consuming components below it in the tree.

```jsx
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>
```

When `value` changes, every component consuming that context will re-render.

### Building a Custom Provider Component

Rather than using `ThemeContext.Provider` directly in your app root, you wrap it in a dedicated component:

```jsx
import { useState } from 'react';
import { ThemeContext } from './ThemeContext';

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  const value = { theme, toggleTheme };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

export default ThemeProvider;
```

**Line-by-line explanation:**

| Line | Explanation |
|---|---|
| `const [theme, setTheme] = useState('light')` | Local state that holds the current theme value |
| `const toggleTheme = () => {...}` | Function to switch between light and dark |
| `const value = { theme, toggleTheme }` | Object bundling state and updater into the context value |
| `<ThemeContext.Provider value={value}>` | The actual React context provider with the value |
| `{children}` | Renders whatever components are passed inside the Provider |

### Context Scope

A Provider's value is only available to components that are **children** (direct or nested) of that Provider. Siblings or ancestors cannot consume it.

```jsx
// ✅ Can use ThemeContext — inside ThemeProvider
<ThemeProvider>
  <App />  {/* ✅ */}
</ThemeProvider>

// ❌ Cannot use ThemeContext — outside the Provider
<Header /> {/* ❌ — has no Provider above it */}
<ThemeProvider>
  ...
</ThemeProvider>
```

### Provider Hierarchy and Overriding

You can nest the same Provider to override values for a subtree:

```jsx
<ThemeContext.Provider value="light">
  <App />  {/* gets "light" */}
  <ThemeContext.Provider value="dark">
    <AdminPanel />  {/* gets "dark" */}
  </ThemeContext.Provider>
</ThemeContext.Provider>
```

The innermost Provider wins for its subtree.

### Passing Values

Keep the value object stable using `useMemo` to prevent unnecessary re-renders:

```jsx
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const value = useMemo(
    () => ({ theme, toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light') }),
    [theme]
  );

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

---

## 7. Understanding Children Props

### What are Children Props?

In React, `children` is a special prop that represents the content passed between a component's opening and closing tags.

```jsx
<ThemeProvider>
  <App />  {/* This is the children prop */}
</ThemeProvider>
```

Inside `ThemeProvider`, `props.children` (or destructured as `{ children }`) refers to `<App />`.

### Why Providers Use children

Providers use the `children` pattern because they need to wrap arbitrary parts of your app. The Provider itself doesn't know or care what its children are — it just renders them inside the context boundary.

```jsx
// ThemeProvider doesn't know about App, Dashboard, or anything else
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme }}>
      {children}  {/* Renders whatever was passed in */}
    </ThemeContext.Provider>
  );
};
```

### The Composition Pattern

This pattern is known as **component composition** — building complex UIs from simpler, reusable pieces.

```jsx
// Wrapping a whole app
<AuthProvider>
  <App />
</AuthProvider>

// Wrapping a specific section
<CartProvider>
  <CheckoutPage />
</CartProvider>

// Wrapping a single component
<ThemeProvider>
  <Dashboard />
</ThemeProvider>
```

### Diagram: Children as a Slot

```
<ThemeProvider>         ← Provider component
  ┌─────────────┐
  │  {children} │  ← slot where App renders
  │  ┌───────┐  │
  │  │  App  │  │
  │  └───────┘  │
  └─────────────┘
</ThemeProvider>
```

### children Can Be Anything

```jsx
// Single component
<ThemeProvider><App /></ThemeProvider>

// Multiple children
<ThemeProvider>
  <Header />
  <Main />
  <Footer />
</ThemeProvider>

// Deeply nested
<ThemeProvider>
  <Router>
    <Layout>
      <App />
    </Layout>
  </Router>
</ThemeProvider>
```

---

## 8. Understanding useContext()

### Syntax

```jsx
import { useContext } from 'react';

const value = useContext(SomeContext);
```

`useContext` takes a **context object** (returned by `createContext`) and returns the current value for that context. It always reads from the nearest matching Provider above the component in the tree.

### How useContext Works Internally

When you call `useContext(ThemeContext)`:

1. React traverses up the component tree from the calling component.
2. It finds the nearest `ThemeContext.Provider`.
3. It returns that Provider's `value` prop.
4. If no Provider is found, it returns the default value from `createContext(defaultValue)`.
5. The component subscribes — when that Provider's `value` changes, this component re-renders.

### Benefits Over Consumer Pattern

| Feature | `useContext` | `Context.Consumer` |
|---|---|---|
| Syntax | Clean, single line | Nested render prop |
| Works with hooks | Yes | No |
| Readability | High | Low |
| TypeScript support | Excellent | Verbose |
| Modern React | Yes | Legacy |

### Practical Examples

**Reading a theme:**
```jsx
import { useContext } from 'react';
import { ThemeContext } from '../context/ThemeContext';

function Button({ label }) {
  const { theme } = useContext(ThemeContext);

  return (
    <button
      style={{
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333',
      }}
    >
      {label}
    </button>
  );
}
```

**Reading auth state:**
```jsx
function ProfilePage() {
  const { user, isLoggedIn } = useContext(AuthContext);

  if (!isLoggedIn) return <p>Please log in.</p>;

  return <h1>Welcome, {user.name}!</h1>;
}
```

**Using cart data:**
```jsx
function CartIcon() {
  const { cart } = useContext(CartContext);

  return (
    <div>
      🛒 <span>{cart.length}</span>
    </div>
  );
}
```

### useContext and Re-renders

A component using `useContext` will re-render every time the context value changes. This is important for performance — if a context value changes frequently, components that only need a small part of it still re-render. This is solved with context splitting (covered in Chapter 16).

---

## 9. Custom Hook Pattern

### Why Custom Hooks Are Preferred

Calling `useContext(SomeContext)` directly in every component works, but it has drawbacks:

- Every consumer must import both the hook AND the context object
- No place to add validation (e.g., checking the context is inside a Provider)
- Logic cannot be shared across consumers

Custom hooks fix all three problems.

### Building Custom Hooks

**useAuth:**
```jsx
// hooks/useAuth.js
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

export const useAuth = () => {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }

  return context;
};
```

**useTheme:**
```jsx
// hooks/useTheme.js
import { useContext } from 'react';
import { ThemeContext } from '../context/ThemeContext';

export const useTheme = () => {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }

  return context;
};
```

**useCart:**
```jsx
// hooks/useCart.js
import { useContext } from 'react';
import { CartContext } from '../context/CartContext';

export const useCart = () => {
  const context = useContext(CartContext);

  if (!context) {
    throw new Error('useCart must be used within a CartProvider');
  }

  return context;
};
```

### Using Custom Hooks in Components

```jsx
// Before custom hook (messy)
import { useContext } from 'react';
import { AuthContext } from '../../context/AuthContext';

function Header() {
  const { user } = useContext(AuthContext);
  return <div>{user?.name}</div>;
}

// After custom hook (clean)
import { useAuth } from '../../hooks/useAuth';

function Header() {
  const { user } = useAuth();
  return <div>{user?.name}</div>;
}
```

### Benefits Summary

**Cleaner Components:** Components only import and call `useAuth()` — no need to know the implementation details.

**Reusability:** The same hook can be used in dozens of components without duplication.

**Validation:** The `if (!context) throw new Error(...)` check ensures the hook is always used inside its Provider, catching bugs early in development.

**Better Developer Experience:** Autocomplete works better, errors are descriptive, and refactoring is easier.

**Encapsulation:** You can add logging, memoization, or derived values inside the hook without changing any consumer components.

---

## 10. Building a Theme Switcher Application

### Project Structure

```
src/
├── context/
│   └── ThemeContext.js
├── hooks/
│   └── useTheme.js
├── components/
│   ├── Navbar.jsx
│   ├── ThemeToggle.jsx
│   └── Card.jsx
└── App.jsx
```

### ThemeContext.js

```jsx
import { createContext } from 'react';

export const ThemeContext = createContext(null);
ThemeContext.displayName = 'ThemeContext';
```

### ThemeProvider.jsx

```jsx
import { useState, useMemo, useCallback } from 'react';
import { ThemeContext } from './ThemeContext';

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = useCallback(() => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  }, []);

  const value = useMemo(
    () => ({ theme, toggleTheme, isDark: theme === 'dark' }),
    [theme, toggleTheme]
  );

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

### useTheme.js

```jsx
import { useContext } from 'react';
import { ThemeContext } from '../context/ThemeContext';

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be inside ThemeProvider');
  return context;
};
```

### ThemeToggle.jsx

```jsx
import { useTheme } from '../hooks/useTheme';

export const ThemeToggle = () => {
  const { isDark, toggleTheme } = useTheme();

  return (
    <button
      onClick={toggleTheme}
      style={{
        padding: '8px 16px',
        borderRadius: '20px',
        cursor: 'pointer',
        background: isDark ? '#f0f0f0' : '#1a1a2e',
        color: isDark ? '#1a1a2e' : '#f0f0f0',
        border: 'none',
        fontWeight: 'bold',
      }}
    >
      {isDark ? '☀️ Light Mode' : '🌙 Dark Mode'}
    </button>
  );
};
```

### Navbar.jsx

```jsx
import { useTheme } from '../hooks/useTheme';
import { ThemeToggle } from './ThemeToggle';

export const Navbar = () => {
  const { isDark } = useTheme();

  return (
    <nav
      style={{
        padding: '16px 24px',
        display: 'flex',
        justifyContent: 'space-between',
        alignItems: 'center',
        background: isDark ? '#1a1a2e' : '#ffffff',
        color: isDark ? '#ffffff' : '#1a1a2e',
        boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
      }}
    >
      <span style={{ fontWeight: 'bold', fontSize: '20px' }}>MyApp</span>
      <ThemeToggle />
    </nav>
  );
};
```

### Card.jsx

```jsx
import { useTheme } from '../hooks/useTheme';

export const Card = ({ title, description }) => {
  const { isDark } = useTheme();

  return (
    <div
      style={{
        padding: '24px',
        borderRadius: '12px',
        background: isDark ? '#16213e' : '#f8f9fa',
        color: isDark ? '#e0e0e0' : '#333',
        boxShadow: isDark
          ? '0 4px 20px rgba(0,0,0,0.4)'
          : '0 4px 20px rgba(0,0,0,0.08)',
        margin: '16px 0',
        transition: 'all 0.3s ease',
      }}
    >
      <h2 style={{ margin: '0 0 8px' }}>{title}</h2>
      <p style={{ margin: 0, opacity: 0.8 }}>{description}</p>
    </div>
  );
};
```

### App.jsx

```jsx
import { ThemeProvider } from './context/ThemeProvider';
import { Navbar } from './components/Navbar';
import { Card } from './components/Card';

function App() {
  return (
    <ThemeProvider>
      <Navbar />
      <main style={{ maxWidth: '800px', margin: '40px auto', padding: '0 24px' }}>
        <Card title="Welcome" description="This app supports dark and light mode using React Context API." />
        <Card title="Theme Switching" description="Click the toggle in the navbar to switch themes instantly." />
      </main>
    </ThemeProvider>
  );
}

export default App;
```

---

## 11. Building an Authentication System

### Project Structure

```
src/
├── context/
│   └── AuthContext.js
├── providers/
│   └── AuthProvider.jsx
├── hooks/
│   └── useAuth.js
└── components/
    ├── LoginForm.jsx
    ├── UserProfile.jsx
    └── ProtectedRoute.jsx
```

### AuthContext.js

```jsx
import { createContext } from 'react';

export const AuthContext = createContext(null);
AuthContext.displayName = 'AuthContext';
```

### AuthProvider.jsx

```jsx
import { useState, useEffect, useMemo, useCallback } from 'react';
import { AuthContext } from '../context/AuthContext';

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(() => localStorage.getItem('authToken'));
  const [loading, setLoading] = useState(true);

  // Restore session on mount
  useEffect(() => {
    const savedUser = localStorage.getItem('authUser');
    if (token && savedUser) {
      setUser(JSON.parse(savedUser));
    }
    setLoading(false);
  }, [token]);

  const login = useCallback(async (email, password) => {
    // In production: replace with real API call
    const mockUser = { id: 1, name: 'Priya Patel', email, role: 'user' };
    const mockToken = 'mock-jwt-token-' + Date.now();

    setUser(mockUser);
    setToken(mockToken);
    localStorage.setItem('authToken', mockToken);
    localStorage.setItem('authUser', JSON.stringify(mockUser));

    return mockUser;
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    setToken(null);
    localStorage.removeItem('authToken');
    localStorage.removeItem('authUser');
  }, []);

  const value = useMemo(
    () => ({
      user,
      token,
      isLoggedIn: !!user,
      isAdmin: user?.role === 'admin',
      loading,
      login,
      logout,
    }),
    [user, token, loading, login, logout]
  );

  return (
    <AuthContext.Provider value={value}>
      {!loading && children}
    </AuthContext.Provider>
  );
};
```

### useAuth.js

```jsx
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
};
```

### LoginForm.jsx

```jsx
import { useState } from 'react';
import { useAuth } from '../hooks/useAuth';

export const LoginForm = () => {
  const { login } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    try {
      await login(email, password);
    } catch {
      setError('Invalid credentials. Please try again.');
    }
  };

  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: '400px', margin: 'auto' }}>
      <h2>Login</h2>
      {error && <p style={{ color: 'red' }}>{error}</p>}
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
        style={{ display: 'block', width: '100%', marginBottom: '12px', padding: '8px' }}
      />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        required
        style={{ display: 'block', width: '100%', marginBottom: '12px', padding: '8px' }}
      />
      <button type="submit" style={{ padding: '10px 20px', cursor: 'pointer' }}>
        Login
      </button>
    </form>
  );
};
```

### UserProfile.jsx

```jsx
import { useAuth } from '../hooks/useAuth';

export const UserProfile = () => {
  const { user, logout, isAdmin } = useAuth();

  return (
    <div style={{ padding: '24px' }}>
      <h2>Welcome, {user.name}!</h2>
      <p>Email: {user.email}</p>
      <p>Role: {user.role}</p>
      {isAdmin && <span style={{ color: 'green' }}>✅ Admin Access</span>}
      <br />
      <button onClick={logout} style={{ marginTop: '16px', padding: '8px 16px' }}>
        Logout
      </button>
    </div>
  );
};
```

### ProtectedRoute.jsx

```jsx
import { useAuth } from '../hooks/useAuth';

export const ProtectedRoute = ({ children, requiredRole }) => {
  const { isLoggedIn, user } = useAuth();

  if (!isLoggedIn) {
    return <p>You must be logged in to view this page.</p>;
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <p>Access denied. You don't have permission to view this.</p>;
  }

  return children;
};

// Usage:
// <ProtectedRoute>
//   <Dashboard />
// </ProtectedRoute>
//
// <ProtectedRoute requiredRole="admin">
//   <AdminPanel />
// </ProtectedRoute>
```

---

## 12. Building a Shopping Cart System

### CartContext.js

```jsx
import { createContext } from 'react';

export const CartContext = createContext(null);
CartContext.displayName = 'CartContext';
```

### CartProvider.jsx

```jsx
import { useReducer, useMemo, useCallback } from 'react';
import { CartContext } from '../context/CartContext';

// Reducer for predictable state updates
const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM': {
      const exists = state.items.find((item) => item.id === action.payload.id);
      if (exists) {
        return {
          ...state,
          items: state.items.map((item) =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
        };
      }
      return { ...state, items: [...state.items, { ...action.payload, quantity: 1 }] };
    }

    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter((item) => item.id !== action.payload),
      };

    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map((item) =>
          item.id === action.payload.id
            ? { ...item, quantity: Math.max(0, action.payload.quantity) }
            : item
        ).filter((item) => item.quantity > 0),
      };

    case 'CLEAR_CART':
      return { ...state, items: [] };

    default:
      return state;
  }
};

const initialState = { items: [] };

export const CartProvider = ({ children }) => {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const addToCart = useCallback((product) => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  }, []);

  const removeFromCart = useCallback((productId) => {
    dispatch({ type: 'REMOVE_ITEM', payload: productId });
  }, []);

  const updateQuantity = useCallback((productId, quantity) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id: productId, quantity } });
  }, []);

  const clearCart = useCallback(() => {
    dispatch({ type: 'CLEAR_CART' });
  }, []);

  const total = useMemo(
    () =>
      state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    [state.items]
  );

  const itemCount = useMemo(
    () => state.items.reduce((sum, item) => sum + item.quantity, 0),
    [state.items]
  );

  const value = useMemo(
    () => ({
      items: state.items,
      total,
      itemCount,
      addToCart,
      removeFromCart,
      updateQuantity,
      clearCart,
    }),
    [state.items, total, itemCount, addToCart, removeFromCart, updateQuantity, clearCart]
  );

  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
};
```

### useCart.js

```jsx
import { useContext } from 'react';
import { CartContext } from '../context/CartContext';

export const useCart = () => {
  const context = useContext(CartContext);
  if (!context) throw new Error('useCart must be used within CartProvider');
  return context;
};
```

### ProductCard.jsx

```jsx
import { useCart } from '../hooks/useCart';

export const ProductCard = ({ product }) => {
  const { addToCart } = useCart();

  return (
    <div style={{ border: '1px solid #eee', borderRadius: '8px', padding: '16px' }}>
      <h3>{product.name}</h3>
      <p>₹{product.price}</p>
      <button onClick={() => addToCart(product)}>Add to Cart</button>
    </div>
  );
};
```

### CartSummary.jsx

```jsx
import { useCart } from '../hooks/useCart';

export const CartSummary = () => {
  const { items, total, itemCount, removeFromCart, updateQuantity, clearCart } = useCart();

  if (items.length === 0) return <p>Your cart is empty.</p>;

  return (
    <div style={{ padding: '24px' }}>
      <h2>Cart ({itemCount} items)</h2>
      {items.map((item) => (
        <div key={item.id} style={{ display: 'flex', gap: '16px', marginBottom: '12px', alignItems: 'center' }}>
          <span style={{ flex: 1 }}>{item.name}</span>
          <span>₹{item.price}</span>
          <input
            type="number"
            value={item.quantity}
            min="1"
            onChange={(e) => updateQuantity(item.id, Number(e.target.value))}
            style={{ width: '60px', padding: '4px' }}
          />
          <button onClick={() => removeFromCart(item.id)}>Remove</button>
        </div>
      ))}
      <hr />
      <h3>Total: ₹{total.toFixed(2)}</h3>
      <button onClick={clearCart} style={{ marginTop: '8px' }}>Clear Cart</button>
    </div>
  );
};
```

---

## 13. Provider Composition Pattern

### What is Provider Composition?

Provider composition is the pattern of nesting multiple context providers to supply different pieces of global state to your app simultaneously.

```jsx
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <CartProvider>
          <NotificationProvider>
            <AppRoutes />
          </NotificationProvider>
        </CartProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}
```

### Why Multiple Contexts?

It is tempting to put all global state in one giant context. But this causes all consumers to re-render whenever **any** part of the state changes — even if they only use one field. Separate contexts give you granular re-render control.

### Composition Diagram

```
<AuthProvider>          ← user, login, logout
  <ThemeProvider>       ← theme, toggleTheme
    <CartProvider>      ← cart, addToCart, total
      <NotifProvider>   ← notifications, addNotif
        <App />         ← has access to ALL 4 contexts
      </NotifProvider>
    </CartProvider>
  </ThemeProvider>
</AuthProvider>
```

Any component in the tree can read from any of the 4 contexts independently.

### AppProviders Wrapper — Clean Composition

Instead of nesting in `main.jsx` or `App.jsx`, create a dedicated wrapper:

```jsx
// providers/AppProviders.jsx
import { AuthProvider } from './AuthProvider';
import { ThemeProvider } from './ThemeProvider';
import { CartProvider } from './CartProvider';
import { NotificationProvider } from './NotificationProvider';

export const AppProviders = ({ children }) => (
  <AuthProvider>
    <ThemeProvider>
      <CartProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </CartProvider>
    </ThemeProvider>
  </AuthProvider>
);

// main.jsx
import { AppProviders } from './providers/AppProviders';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <AppProviders>
      <App />
    </AppProviders>
  </React.StrictMode>
);
```

### Order Matters

The order of nesting can matter when providers depend on each other. For example, `CartProvider` might use the logged-in user to fetch their cart — so `AuthProvider` should be the outermost:

```jsx
// ✅ Auth wraps Cart — CartProvider can use useAuth()
<AuthProvider>
  <CartProvider>  {/* can access AuthContext internally */}
    ...
  </CartProvider>
</AuthProvider>

// ❌ Cart wraps Auth — CartProvider cannot use AuthContext
<CartProvider>
  <AuthProvider>
    ...
  </AuthProvider>
</CartProvider>
```

### Context Separation Benefits

- **Performance:** Each context re-renders only its own consumers
- **Maintainability:** Each context has a single responsibility
- **Testability:** Mock just the context you need for a given test
- **Team-Friendly:** Different developers can own different contexts

---

## 14. Context Folder Structure

Good structure is what separates hobby projects from production codebases.

### Small Applications

```
src/
├── context/
│   ├── ThemeContext.jsx     ← createContext + Provider + hook all in one file
│   └── AuthContext.jsx
└── components/
    └── ...
```

For small apps, everything can live in one file per context.

### Medium Applications

```
src/
├── context/
│   ├── AuthContext.js       ← just the createContext call
│   ├── ThemeContext.js
│   └── CartContext.js
├── providers/
│   ├── AuthProvider.jsx     ← Provider component with state logic
│   ├── ThemeProvider.jsx
│   └── CartProvider.jsx
├── hooks/
│   ├── useAuth.js           ← custom hook wrapping useContext
│   ├── useTheme.js
│   └── useCart.js
└── components/
    └── ...
```

### Enterprise Applications

```
src/
├── contexts/
│   ├── auth/
│   │   ├── AuthContext.js
│   │   ├── AuthProvider.jsx
│   │   ├── useAuth.js
│   │   ├── authReducer.js
│   │   └── index.js
│   ├── theme/
│   │   ├── ThemeContext.js
│   │   ├── ThemeProvider.jsx
│   │   ├── useTheme.js
│   │   └── index.js
│   └── cart/
│       ├── CartContext.js
│       ├── CartProvider.jsx
│       ├── useCart.js
│       ├── cartReducer.js
│       └── index.js
├── providers/
│   └── AppProviders.jsx     ← composes all providers
├── services/
│   ├── authService.js       ← API calls separated from Context
│   └── cartService.js
├── hooks/                   ← any additional custom hooks
└── components/
    └── ...
```

### Best Practices for Structure

- Keep one context per feature (auth, theme, cart, notifications)
- Use `index.js` files to create clean public APIs
- Separate state logic (reducer) from the Provider component
- Keep API calls in `services/` — not inside Providers

---

## 15. Context Module Pattern

The Context Module Pattern splits a single context feature into well-defined, separately exportable files, then re-exports them from an `index.js`. This makes imports clean and the module easy to test independently.

### Folder

```
contexts/auth/
├── AuthContext.js      ← createContext
├── AuthProvider.jsx    ← Provider component
├── useAuth.js          ← custom hook
├── authReducer.js      ← state management logic
└── index.js            ← public API
```

### authReducer.js

```jsx
export const authReducer = (state, action) => {
  switch (action.type) {
    case 'LOGIN':
      return { ...state, user: action.payload.user, token: action.payload.token, isLoggedIn: true };
    case 'LOGOUT':
      return { user: null, token: null, isLoggedIn: false };
    case 'UPDATE_USER':
      return { ...state, user: { ...state.user, ...action.payload } };
    default:
      return state;
  }
};

export const initialAuthState = {
  user: null,
  token: null,
  isLoggedIn: false,
};
```

### AuthProvider.jsx (using reducer)

```jsx
import { useReducer, useMemo, useCallback, useEffect } from 'react';
import { AuthContext } from './AuthContext';
import { authReducer, initialAuthState } from './authReducer';

export const AuthProvider = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialAuthState);

  useEffect(() => {
    const token = localStorage.getItem('authToken');
    const user = localStorage.getItem('authUser');
    if (token && user) {
      dispatch({ type: 'LOGIN', payload: { user: JSON.parse(user), token } });
    }
  }, []);

  const login = useCallback((user, token) => {
    localStorage.setItem('authToken', token);
    localStorage.setItem('authUser', JSON.stringify(user));
    dispatch({ type: 'LOGIN', payload: { user, token } });
  }, []);

  const logout = useCallback(() => {
    localStorage.removeItem('authToken');
    localStorage.removeItem('authUser');
    dispatch({ type: 'LOGOUT' });
  }, []);

  const value = useMemo(
    () => ({ ...state, login, logout }),
    [state, login, logout]
  );

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};
```

### index.js

```jsx
// contexts/auth/index.js
export { AuthContext } from './AuthContext';
export { AuthProvider } from './AuthProvider';
export { useAuth } from './useAuth';
```

### Usage from anywhere

```jsx
// Clean single import
import { useAuth, AuthProvider } from '../contexts/auth';
```

---

## 16. Performance Optimization

### The Problem: Unnecessary Re-renders

Every time a Provider's `value` prop changes, all consumers re-render. If the value is a new object literal on every render, this happens even when the data hasn't actually changed.

```jsx
// ❌ BAD — new object on every render triggers all consumers
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>  {/* new obj every render! */}
      {children}
    </ThemeContext.Provider>
  );
};
```

### Fix 1: useMemo for the Context Value

```jsx
// ✅ GOOD — value only changes when theme actually changes
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const value = useMemo(
    () => ({ theme, setTheme }),
    [theme]  // Only recreate when theme changes
  );

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

### Fix 2: useCallback for Functions

```jsx
// ✅ Stable function reference — won't trigger re-renders
const login = useCallback(async (email, password) => {
  // login logic
}, []); // empty deps = created once
```

### Fix 3: Context Splitting

Split one large context into multiple smaller ones based on how frequently values change.

```jsx
// ❌ BAD — one context for both static user data and dynamic notifications
const AppContext = createContext({ user, notifications, theme });

// ✅ GOOD — separate contexts; user data change won't re-render notification consumers
const UserContext = createContext(null);        // rarely changes
const NotificationContext = createContext([]);  // changes often
const ThemeContext = createContext('light');    // changes on toggle
```

### Fix 4: React.memo for Consumer Components

Wrap consumer components in `React.memo` to prevent re-renders from parent changes unrelated to context:

```jsx
import { memo } from 'react';
import { useTheme } from '../hooks/useTheme';

const ThemeToggle = memo(() => {
  const { isDark, toggleTheme } = useTheme();
  return <button onClick={toggleTheme}>{isDark ? '☀️' : '🌙'}</button>;
});
```

### Before vs After Optimization

```jsx
// BEFORE — 3 re-renders per toggle (Header, Sidebar, Footer all re-render)
const AppContext = createContext({ user, theme, cart });

// AFTER — only theme consumers re-render on toggle
const UserContext = createContext({ user });
const ThemeContext = createContext({ theme });
const CartContext = createContext({ cart });
```

### Performance Summary

| Technique | When to Use | Impact |
|---|---|---|
| `useMemo` on value | Always in Providers | Prevents unnecessary consumer re-renders |
| `useCallback` on functions | Functions passed in context | Stable refs, better `React.memo` behavior |
| Context splitting | Large contexts | Only relevant consumers re-render |
| `React.memo` | Frequently rendered consumers | Stops re-renders from parent updates |

---

## 17. Context API vs Redux Toolkit

### Feature Comparison

| Feature | Context API | Redux Toolkit |
|---|---|---|
| Built-in to React | ✅ Yes | ❌ No (separate package) |
| Setup complexity | Low | Medium |
| Boilerplate | Minimal | Some (reducers, slices) |
| DevTools | Basic (React DevTools) | Excellent (Redux DevTools) |
| Middleware support | No | Yes (thunks, sagas) |
| Performance at scale | Requires manual optimization | Optimized by default (selectors) |
| Time-travel debugging | No | Yes |
| Learning curve | Low | Medium |
| Bundle size | Zero (built-in) | ~47kb (redux + react-redux + rtk) |
| Async handling | Manual (useEffect, fetch) | RTK Query (built-in) |
| Best for | Small–Medium apps | Medium–Large apps |

### When to Choose Context API

- Small to medium applications (< 10 shared states)
- Simple global state like theme, locale, auth
- Teams new to React state management
- Projects where bundle size is critical
- No need for advanced debugging or middleware

### When to Choose Redux Toolkit

- Large, complex applications with many slices of state
- Need for Redux DevTools (time travel, action history)
- Complex async data fetching (RTK Query)
- Enterprise teams familiar with Redux patterns
- State that needs to be serialized/persisted/shared across tabs
- Need for middleware (logging, analytics)

### Code Comparison: Auth State

**Context API:**
```jsx
export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const login = useCallback(async (credentials) => {
    const data = await api.login(credentials);
    setUser(data.user);
  }, []);
  const value = useMemo(() => ({ user, login }), [user, login]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

**Redux Toolkit:**
```jsx
// authSlice.js
const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null },
  reducers: {
    setUser: (state, action) => { state.user = action.payload; },
  },
});

// Component
const user = useSelector((state) => state.auth.user);
const dispatch = useDispatch();
dispatch(setUser(data.user));
```

### The Verdict

> Start with Context API. Migrate to Redux Toolkit when you hit pain points: debugging complexity, performance issues, or need for advanced async handling.

---

## 18. Common Mistakes

### Mistake 1: Overusing Context for Local State

```jsx
// ❌ BAD — putting form state in global context
const FormContext = createContext();
// Form state only needs to be in the component, not globally shared

// ✅ GOOD — local state for form
const [formData, setFormData] = useState({ name: '', email: '' });
```

**Rule:** If only one component needs the data, use local state. Use Context only for data shared across multiple components.

### Mistake 2: Large Context Objects

```jsx
// ❌ BAD — one giant context
const AppContext = createContext({
  user, theme, cart, notifications, permissions, preferences, language
});
// Every component re-renders when anything changes

// ✅ GOOD — split by concern
const AuthContext = createContext({ user, permissions });
const ThemeContext = createContext({ theme });
const CartContext = createContext({ cart });
```

### Mistake 3: Creating New Objects on Every Render

```jsx
// ❌ BAD
<MyContext.Provider value={{ count, setCount }}>
  {/* new object literal every render — all consumers re-render */}

// ✅ GOOD
const value = useMemo(() => ({ count, setCount }), [count]);
<MyContext.Provider value={value}>
```

### Mistake 4: Not Using Custom Hooks

```jsx
// ❌ BAD — importing context object everywhere
import { useContext } from 'react';
import { AuthContext } from '../../context/AuthContext';
const { user } = useContext(AuthContext);

// ✅ GOOD — custom hook with validation
import { useAuth } from '../../hooks/useAuth';
const { user } = useAuth();
```

### Mistake 5: Incorrect Provider Placement

```jsx
// ❌ BAD — Provider inside the component that also consumes it
function Dashboard() {
  return (
    <ThemeProvider>
      <Dashboard />  {/* infinite loop! */}
    </ThemeProvider>
  );
}

// ✅ GOOD — Provider wraps consumers from the outside
function App() {
  return (
    <ThemeProvider>
      <Dashboard />
    </ThemeProvider>
  );
}
```

### Mistake 6: Putting Functions That Change in Context Without useCallback

```jsx
// ❌ BAD — new function reference on every render
const value = { login: async (creds) => { ... } };

// ✅ GOOD — stable reference
const login = useCallback(async (creds) => { ... }, []);
const value = useMemo(() => ({ login }), [login]);
```

---

## 19. Legacy Consumer Pattern (Brief)

### Context.Consumer

Before React Hooks (React 16.3–16.7), context was consumed using a render prop pattern called `Context.Consumer`:

```jsx
// Legacy — avoid in new code
<ThemeContext.Consumer>
  {(value) => (
    <div style={{ background: value.theme === 'dark' ? '#333' : '#fff' }}>
      Content here
    </div>
  )}
</ThemeContext.Consumer>
```

### Why useContext Replaced It

| Aspect | Consumer | useContext |
|---|---|---|
| Syntax | Render prop (verbose) | Single hook call (clean) |
| Nesting | Gets deeply nested | Flat |
| Multiple contexts | Deeply nested | Multiple hook calls |
| Works in functions | Yes | Yes |
| Works in class components | Yes | No |

```jsx
// Old way — 3 contexts = 3 levels of nesting
<AuthContext.Consumer>
  {({ user }) => (
    <ThemeContext.Consumer>
      {({ theme }) => (
        <CartContext.Consumer>
          {({ cart }) => <Dashboard user={user} theme={theme} cart={cart} />}
        </CartContext.Consumer>
      )}
    </ThemeContext.Consumer>
  )}
</AuthContext.Consumer>

// New way — clean and flat
function Dashboard() {
  const { user } = useAuth();
  const { theme } = useTheme();
  const { cart } = useCart();
  return <div>...</div>;
}
```

`Context.Consumer` is still valid and works fine — it's the preferred approach in **class components** since hooks cannot be used in them. For all functional components, use `useContext` (via custom hooks).

---

## 20. Real-World Architectures

### LMS (Learning Management System) Platform

```
src/
├── contexts/
│   ├── auth/              ← student/instructor login, roles
│   │   ├── AuthContext.js
│   │   ├── AuthProvider.jsx
│   │   └── useAuth.js
│   ├── courses/           ← enrolled courses, progress
│   │   ├── CourseContext.js
│   │   ├── CourseProvider.jsx
│   │   └── useCourses.js
│   └── theme/             ← UI theme preferences
│       ├── ThemeContext.js
│       ├── ThemeProvider.jsx
│       └── useTheme.js
├── providers/
│   └── AppProviders.jsx
└── components/
    ├── CourseCard/
    ├── VideoPlayer/
    └── ProgressBar/
```

**Contexts in use:**
- `AuthContext` — current user, role (student/instructor/admin), permissions
- `CourseContext` — enrolled courses, current lesson, completion progress
- `ThemeContext` — light/dark mode

### CRM (Customer Relationship Management) System

```
src/
├── contexts/
│   ├── user/              ← agent identity, team, permissions
│   ├── notifications/     ← real-time alerts, toasts
│   └── permissions/       ← RBAC (role-based access control)
├── providers/
│   └── AppProviders.jsx
└── components/
    ├── LeadCard/
    ├── NotifBell/
    └── PermissionGate/
```

**PermissionGate example:**
```jsx
export const PermissionGate = ({ permission, children }) => {
  const { hasPermission } = usePermissions();
  return hasPermission(permission) ? children : null;
};

// Usage:
<PermissionGate permission="delete_lead">
  <DeleteButton />
</PermissionGate>
```

### E-Commerce Application

```
src/
├── contexts/
│   ├── auth/              ← login, session, user profile
│   ├── cart/              ← items, quantities, total
│   ├── wishlist/          ← saved products
│   └── preferences/       ← language, currency, region
├── providers/
│   └── AppProviders.jsx
└── components/
    ├── ProductCard/
    ├── CartDrawer/
    └── WishlistButton/
```

**Wishlist Context example:**
```jsx
export const WishlistProvider = ({ children }) => {
  const [wishlist, setWishlist] = useState([]);

  const addToWishlist = useCallback((product) => {
    setWishlist((prev) =>
      prev.some((p) => p.id === product.id) ? prev : [...prev, product]
    );
  }, []);

  const removeFromWishlist = useCallback((productId) => {
    setWishlist((prev) => prev.filter((p) => p.id !== productId));
  }, []);

  const isInWishlist = useCallback(
    (productId) => wishlist.some((p) => p.id === productId),
    [wishlist]
  );

  const value = useMemo(
    () => ({ wishlist, addToWishlist, removeFromWishlist, isInWishlist }),
    [wishlist, addToWishlist, removeFromWishlist, isInWishlist]
  );

  return (
    <WishlistContext.Provider value={value}>
      {children}
    </WishlistContext.Provider>
  );
};
```

---

## 21. Context API Best Practices

### Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Context object | `PascalCase` + `Context` | `AuthContext`, `ThemeContext` |
| Provider component | `PascalCase` + `Provider` | `AuthProvider`, `CartProvider` |
| Custom hook | `use` + `PascalCase` | `useAuth`, `useTheme`, `useCart` |
| Context file | `PascalCase` + `Context.js` | `AuthContext.js` |
| Provider file | `PascalCase` + `Provider.jsx` | `AuthProvider.jsx` |
| Hook file | `use` + `PascalCase` + `.js` | `useAuth.js` |

### Context Separation

- One context per domain/concern
- Never mix unrelated data in one context
- Ask: "Would a change to this data affect the same set of components as a change to that data?" If no, split them.

### Custom Hooks Are Mandatory

Always expose context through a custom hook, never raw `useContext`:
```jsx
// Every context should have a matching hook
export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be within AuthProvider');
  return ctx;
};
```

### Provider Organization

Keep providers organized and composed at the app root:
```jsx
// providers/AppProviders.jsx — single place to add/remove providers
export const AppProviders = ({ children }) => (
  <AuthProvider>
    <ThemeProvider>
      <CartProvider>
        {children}
      </CartProvider>
    </ThemeProvider>
  </AuthProvider>
);
```

### Scalable Architecture

- Use the **Context Module Pattern** for each context (its own folder)
- Separate state logic into reducers
- Keep API calls in service files
- Export only what consumers need from `index.js`
- Document the shape of each context value with TypeScript or JSDoc

---

## 22. Interview Questions

### Beginner Level (40 Questions)

**Q1. What is the React Context API?**
A: Context API is a built-in React feature that provides a way to share values between components without passing props manually at every level. It solves the prop drilling problem.

**Q2. What is prop drilling?**
A: Prop drilling is passing data through intermediate components that don't need the data, just to deliver it to a deeply nested component that does.

**Q3. What does createContext() return?**
A: It returns a context object containing a `Provider` component, a `Consumer` component (legacy), and a `_currentValue` property. The main parts you use are `Provider` and `useContext`.

**Q4. What is the purpose of the Provider component?**
A: Provider wraps part of the component tree and makes context values available to all child components. It accepts a `value` prop that consumers read.

**Q5. What is useContext()?**
A: `useContext` is a React hook that accepts a context object and returns the current context value provided by the nearest matching Provider above in the tree.

**Q6. What happens when no Provider is found for useContext?**
A: React returns the default value specified in `createContext(defaultValue)`.

**Q7. Can you use Context without a Provider?**
A: Yes, but consumers will receive the default value passed to `createContext()` instead of a dynamic value.

**Q8. What is the children prop?**
A: `children` is a special React prop that represents the JSX content passed between a component's opening and closing tags. Providers use it to render whatever components are wrapped inside them.

**Q9. Why do we wrap Providers around components?**
A: To make context values available to all descendants without requiring prop passing at each level.

**Q10. What is a custom hook in the context of Context API?**
A: A custom hook (e.g., `useAuth`, `useTheme`) wraps `useContext` and provides a cleaner API with optional validation to ensure the hook is used inside its Provider.

**Q11. How do you create a theme context?**
A: `const ThemeContext = createContext('light');`

**Q12. How many contexts can you have in a React app?**
A: As many as you need — one per domain/concern is the recommended pattern.

**Q13. Can Providers be nested?**
A: Yes. You can nest multiple providers. The innermost Provider for a given context wins for its subtree.

**Q14. Does useContext cause re-renders?**
A: Yes. Every time the Provider's value changes, all components consuming that context via useContext will re-render.

**Q15. Is Context API a replacement for Redux?**
A: For simple global state, yes. For complex apps requiring middleware, time-travel debugging, or advanced async handling, Redux Toolkit is more appropriate.

**Q16. What is the difference between local state and context state?**
A: Local state (useState) is private to a component. Context state is shared across multiple components in a subtree.

**Q17. When should you NOT use Context API?**
A: When data is only needed by a single component (use local state), or when state changes are very frequent and affect many consumers (use Redux or Zustand for better performance).

**Q18. Can you update context value from a child component?**
A: Yes, by passing a setter function (or a `dispatch` function) through the context value. Child components call the function to update state in the Provider.

**Q19. What is the displayName property on a context?**
A: It sets the label shown in React DevTools, making it easier to identify contexts: `ThemeContext.displayName = 'ThemeContext'`.

**Q20. What is the default value of createContext() if you don't pass one?**
A: `undefined`.

**Q21. Can you have multiple Providers for the same context?**
A: Yes. Each subtree gets the value from its nearest Provider ancestor.

**Q22. How do you share a function through context?**
A: Include it in the value object: `<MyContext.Provider value={{ doSomething }}>`

**Q23. What does the Provider's value prop accept?**
A: Any JavaScript value — string, number, object, array, function, or a combination.

**Q24. What is the relationship between Context and state?**
A: Context is the distribution mechanism. State (useState/useReducer) is the storage. You use state in the Provider, then pass it through context to consumers.

**Q25. Can a component consume multiple contexts?**
A: Yes. Call `useContext` (or the custom hooks) multiple times: `const { user } = useAuth(); const { theme } = useTheme();`

**Q26. Does Context work with class components?**
A: Yes, using `Context.Consumer` render prop or `static contextType = MyContext`.

**Q27. What is the Consumer component?**
A: `Context.Consumer` is the legacy render-prop API for reading context in class components. In function components, always use `useContext` instead.

**Q28. Are contexts re-created on every render?**
A: The context object itself is static (created once). The value inside it changes when the Provider's value prop changes.

**Q29. What is a Provider wrapper component?**
A: A custom component (e.g., `ThemeProvider`) that encapsulates state logic and renders `<ThemeContext.Provider value={...}>`. It hides implementation details from consumers.

**Q30. How does Context relate to component re-renders?**
A: When a Provider's value changes, React re-renders all components that subscribe to that context via useContext.

**Q31. What is the best way to import a context custom hook?**
A: From a dedicated hooks file: `import { useAuth } from '../hooks/useAuth';`

**Q32. Can you pass children as a function?**
A: Yes, that's the render prop pattern. But for context, passing children as JSX is standard.

**Q33. Is Context synchronous or asynchronous?**
A: The context mechanism itself is synchronous. Async operations (API calls) can be triggered inside the Provider, and state updates happen after they resolve.

**Q34. What happens if you forget to wrap a component in its Provider?**
A: `useContext` returns the default value. If you use a custom hook with a null check (`if (!context) throw new Error(...)`), you'll get a helpful error message.

**Q35. How do you test a component that uses context?**
A: Wrap the component in its Provider inside the test: `render(<ThemeProvider><MyComponent /></ThemeProvider>)`.

**Q36. Can context replace all prop passing?**
A: No. Props are still appropriate for data specific to individual instances of a component. Context is for truly shared/global data.

**Q37. Does React re-render intermediate components when context value changes?**
A: No. Only components that directly call `useContext` for that context will re-render. Intermediate components that don't consume the context are not affected.

**Q38. Can you read context inside event handlers?**
A: Yes. If the hook is called at the top of the component, the returned value is available anywhere in the function body, including event handlers and effects.

**Q39. Can you read context inside useEffect?**
A: Yes. Context values read by the component are available inside `useEffect`. Include them in the dependency array if needed.

**Q40. What is Provider Composition?**
A: Nesting multiple Provider components to supply different contexts to an application simultaneously.

---

### Intermediate Level (30 Questions)

**Q1. Why use useMemo on the context value?**
A: To prevent creating a new object reference on every render. Without `useMemo`, the value object is new on every render (even with identical data), causing all consumers to re-render unnecessarily.

**Q2. What is context splitting and why is it important?**
A: Splitting one large context into multiple smaller ones by domain. Consumers only re-render when the specific context they use changes, not when any part of a monolithic context changes.

**Q3. What is the Context Module Pattern?**
A: A pattern that organizes each context feature into its own folder with separate files for the context object, Provider component, custom hook, and optional reducer — all re-exported from an index.js.

**Q4. When would you use useReducer instead of useState in a Provider?**
A: When state logic is complex (multiple sub-values, multiple types of updates, complex transitions). `useReducer` makes state transitions explicit and predictable.

**Q5. How would you add TypeScript types to a context?**
A:
```tsx
type AuthContextType = {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
};
const AuthContext = createContext<AuthContextType | null>(null);
```

**Q6. What is the difference between React.memo and useMemo in the context of Context API?**
A: `useMemo` stabilizes the context value object inside the Provider. `React.memo` prevents consumer components from re-rendering due to changes in their parent (unrelated to context changes).

**Q7. How does Provider ordering affect nested contexts?**
A: If one Provider depends on another context (e.g., CartProvider reads the logged-in user), the dependency must be an ancestor. Outer Providers are available to inner Provider logic.

**Q8. What is the risk of putting fast-changing state in context?**
A: All consumers re-render on every change. Putting rapidly updating state (e.g., mouse position, scroll position) in context would cause performance issues across the app.

**Q9. How do you persist context state across page refreshes?**
A: Read from `localStorage` in the initial state (lazy initializer of `useState` or initial `useEffect`), and write to `localStorage` on state changes.

**Q10. What is a selector pattern and how does it relate to Context?**
A: Selectors derive specific values from state. In Context, you can create derived values with `useMemo` in the Provider instead of having consumers compute them. In Redux, selectors prevent re-renders more efficiently than Context can.

**Q11. How would you handle loading states in an AuthProvider?**
A: Include a `loading` boolean in state. Set it `true` initially, `false` after restoring session from localStorage. Conditionally render children: `{!loading && children}`.

**Q12. Can a Provider render null or conditionally render children?**
A: Yes. This is useful for loading screens: `{loading ? <Spinner /> : children}`.

**Q13. What is the difference between `value` prop mutation and replacement?**
A: Context compares the `value` prop by reference. Mutating an object without creating a new one won't trigger re-renders. Always create new objects/arrays to update context properly.

**Q14. How would you implement a notification system with Context?**
A: State of `notifications` array in Provider, with `addNotification` and `removeNotification` functions. Each notification has an id, message, type, and optional auto-dismiss timeout.

**Q15. How do you share derived data through context?**
A: Compute it inside the Provider (using `useMemo`) and include it in the value: `const itemCount = useMemo(() => items.reduce(...), [items]);`

**Q16. What is wrong with calling useContext inside a conditional?**
A: Hooks must be called unconditionally at the top level of a function component. React relies on hook call order remaining consistent across renders.

**Q17. How do you debounce or throttle context updates?**
A: Use `useMemo` with debounced values, or separate frequently updating state into local component state and only promote to context when the value settles.

**Q18. What are the tradeoffs of the AppProviders pattern?**
A: Pros: single place to manage all providers, clean `main.jsx`. Cons: all providers are always mounted; for lazy-loaded sections, you might want providers closer to where they're used.

**Q19. Can you use Context inside a custom hook that isn't directly a context hook?**
A: Yes. Any custom hook can call `useContext` internally, as long as it's called inside a React function component or another custom hook.

**Q20. How would you implement role-based access control (RBAC) with Context?**
A: Include a `hasPermission(permission)` function in AuthContext that checks `user.permissions` array or role mappings. Use a `PermissionGate` component that renders children only if the permission check passes.

**Q21. What's the difference between exporting the Context vs exporting the hook?**
A: Exporting the Context allows consumers to call `useContext(AuthContext)` directly (bypassing custom hook validation). Exporting only the hook enforces usage through the validated path.

**Q22. How do you test a context Provider?**
A: Create a test wrapper: `const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>` and pass it as the `wrapper` option in `renderHook` or `render`.

**Q23. What is an unstable context value?**
A: A context value that creates a new reference on every render (e.g., an inline object literal), causing all consumers to re-render even when data is unchanged.

**Q24. How does React Context interact with React.StrictMode?**
A: In StrictMode (development only), Providers render twice. This surfaces side effects in Provider logic. Always handle this gracefully (idempotent effects, cleanup functions).

**Q25. How would you reset context state?**
A: Include a `reset` function in the context value that calls `dispatch({ type: 'RESET' })` or sets state back to initial values.

**Q26. Can you use context across different React roots?**
A: No. Context is scoped to a single React root. For cross-root sharing, you'd need external state (localStorage, Zustand atoms, Redux store).

**Q27. What happens when a Provider unmounts?**
A: Consumers that subscribed to it continue to have the last value until they re-render inside a new Provider or receive the default value.

**Q28. How do you prevent a context consumer from re-rendering when only an unrelated field changes?**
A: Split the context, or use a library like `use-context-selector` that supports selector-based subscriptions.

**Q29. How do you add optimistic updates in a cart context?**
A: Update local state immediately (optimistic), then call the API. If the API fails, revert state to the previous value using error handling in the `catch` block.

**Q30. What is the overhead of deeply nested Providers?**
A: Minimal. Provider components themselves are lightweight. The cost is in consumers re-rendering, not in the Provider hierarchy depth.

---

### Advanced Level (20 Questions)

**Q1. How does React's reconciler decide which context consumers to update?**
A: React uses reference equality (`Object.is`) to compare old and new context values. If the reference is different (even if the data is logically the same), all consumers of that context are scheduled for re-render.

**Q2. How does use-context-selector solve the re-render problem?**
A: `use-context-selector` (by Daishi Kato) allows consumers to subscribe to a specific slice of context using a selector function. The component only re-renders when the selected slice changes, similar to Redux's `useSelector`.

**Q3. What is context propagation and how does React optimize it?**
A: When a Provider's value changes, React performs a "context propagation" pass. It traverses children looking for context subscribers, bailing out at `React.memo` boundaries only if the memoized component doesn't subscribe to the changed context.

**Q4. Explain the "context value is stale" problem in closures.**
A: If a function captured from context closes over an old value (from when the component last rendered), it may use stale data. Using `useCallback` with proper dependencies or `useRef` to always hold the latest value prevents this.

**Q5. How would you implement a multi-tenant context architecture?**
A: Each tenant gets its own Provider tree, configured with tenant-specific data. Components are written generically to consume the nearest Provider, which supplies tenant-specific values. This enables full isolation without duplicating components.

**Q6. What is the atom model (Recoil/Jotai) and how does it differ from Context?**
A: Atom-based state management stores individual units of state (atoms) that components subscribe to independently. Unlike Context, changing one atom only re-renders components that subscribe to that specific atom — not all consumers of a shared Provider.

**Q7. How do you implement context with server-side rendering (SSR)?**
A: Initialize context state from server-fetched data (passed as props or via serialized JSON in the HTML). Providers use these values as initial state. Hydration must match — use identical initial values on client and server.

**Q8. How would you implement a plugin system using Context?**
A: A root `PluginContext` holds a registry of plugins (functions/components). Provider consumers can register plugins using a `registerPlugin` function from context. Host components render registered plugins dynamically.

**Q9. How does React's batching affect context updates in React 18?**
A: React 18 introduced automatic batching for all updates (not just event handlers). Multiple `setState` calls inside async functions, setTimeout, or native event handlers are now batched, reducing the number of context propagation passes.

**Q10. What is the concurrent features impact on Context?**
A: With React 18's concurrent rendering, a component may render with different context values during interruptions. This is generally safe but means you should not rely on context reads being synchronized across an entire render.

**Q11. How would you architect context for a micro-frontend environment?**
A: Each micro-frontend manages its own context internally. Cross-micro-frontend state is shared through a host-level context, custom events, or a shared state management library (Zustand) rather than React Context (which can't cross module boundaries).

**Q12. What is the difference between context and a pub-sub system?**
A: Context is pull-based — components subscribe by calling `useContext` and receive re-renders. Pub-sub is push-based — publishers emit events, and subscribers receive them. Context is simpler but less decoupled than pub-sub.

**Q13. How do you create a type-safe context with a non-null assertion?**
A:
```tsx
const AuthContext = createContext<AuthContextType | null>(null);
export const useAuth = (): AuthContextType => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth outside AuthProvider');
  return ctx; // TypeScript now knows ctx is AuthContextType
};
```

**Q14. How would you implement undo/redo using Context and useReducer?**
A: Keep a history stack in the reducer state: `{ past: [...], present: {...}, future: [...] }`. `UNDO` pops from past into present; `REDO` pops from future into present. Expose `undo`, `redo`, `canUndo`, `canRedo` through context.

**Q15. Explain the render phase vs commit phase in relation to context.**
A: Context subscriptions are evaluated during the render phase. When a Provider's value changes, React queues re-renders for consumers. Those re-renders happen in the next render phase. Side effects (useEffect) run in the commit phase, after all renders.

**Q16. How do you share context state between a React web app and a React Native app?**
A: Extract context logic (Provider, hooks, reducers) into a shared package. Both apps import from the package. The UI rendering differs, but the state logic (which is platform-agnostic JS) is shared.

**Q17. What is context composition vs context inheritance?**
A: Composition: multiple independent contexts, each with a single responsibility, composed together. Inheritance: one context extending another's value (not a React concept, but can be simulated by passing parent context value into a child Provider). Composition is always preferred.

**Q18. How do you implement optimistic updates with rollback in a Context-based cart?**
A: Save previous state before updating. Apply the optimistic change immediately. Trigger the API call. On success, confirm the state (no change needed). On failure, call `setState(previousState)` to rollback, and surface the error to the user.

**Q19. How does React DevTools visualize context?**
A: In React DevTools, you can inspect any component and see its context values under the "hooks" section. Named contexts (with `displayName` set) are labeled clearly. Value changes are highlighted during re-renders.

**Q20. When is it appropriate to use Context API alongside Redux Toolkit?**
A: When some state is truly global/app-level (use Redux), but certain UI state needs to be scoped to a subtree without polluting the global Redux store (use Context). For example, Redux for server data + auth; Context for theme, locale, or feature-flag scoped to a page.

---

## 23. Context API Cheat Sheet

### createContext

```jsx
import { createContext } from 'react';

// Basic
const ThemeContext = createContext('light');

// Object default
const AuthContext = createContext({ user: null, isLoggedIn: false });

// Null (always wrapped in Provider)
const CartContext = createContext(null);

// With displayName
ThemeContext.displayName = 'ThemeContext';
```

### Provider

```jsx
// Pattern 1: Inline (small apps)
<ThemeContext.Provider value={{ theme, setTheme }}>
  {children}
</ThemeContext.Provider>

// Pattern 2: Wrapper component (recommended)
export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

### children props

```jsx
// Provider accepts children
const MyProvider = ({ children }) => (
  <MyContext.Provider value={...}>{children}</MyContext.Provider>
);

// Usage
<MyProvider>
  <App />
</MyProvider>
```

### useContext

```jsx
import { useContext } from 'react';

// Direct (not recommended for production)
const { theme } = useContext(ThemeContext);

// Via custom hook (recommended)
const { theme } = useTheme();
```

### Custom Hooks

```jsx
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be within AuthProvider');
  return context;
};
```

### Provider Composition

```jsx
// AppProviders.jsx
export const AppProviders = ({ children }) => (
  <AuthProvider>
    <ThemeProvider>
      <CartProvider>
        {children}
      </CartProvider>
    </ThemeProvider>
  </AuthProvider>
);

// main.jsx
<AppProviders>
  <App />
</AppProviders>
```

### Performance

```jsx
// Memoize value
const value = useMemo(() => ({ theme, toggleTheme }), [theme, toggleTheme]);

// Memoize functions
const toggleTheme = useCallback(() => setTheme(t => t === 'light' ? 'dark' : 'light'), []);

// Memo consumer
const MyConsumer = memo(() => {
  const { theme } = useTheme();
  return <div className={theme}>...</div>;
});
```

---

## 24. Complete Production-Ready Project

### Project: ShopSphere — Mini E-Commerce App

**Features:** Authentication, Theme Switching, Notifications, Shopping Cart, Multiple Contexts, Provider Composition

### Folder Structure

```
src/
├── contexts/
│   ├── auth/
│   │   ├── AuthContext.js
│   │   ├── AuthProvider.jsx
│   │   ├── useAuth.js
│   │   └── index.js
│   ├── theme/
│   │   ├── ThemeContext.js
│   │   ├── ThemeProvider.jsx
│   │   ├── useTheme.js
│   │   └── index.js
│   ├── cart/
│   │   ├── CartContext.js
│   │   ├── CartProvider.jsx
│   │   ├── useCart.js
│   │   ├── cartReducer.js
│   │   └── index.js
│   └── notifications/
│       ├── NotificationContext.js
│       ├── NotificationProvider.jsx
│       ├── useNotifications.js
│       └── index.js
├── providers/
│   └── AppProviders.jsx
├── components/
│   ├── Navbar/
│   │   └── Navbar.jsx
│   ├── ProductCard/
│   │   └── ProductCard.jsx
│   ├── CartDrawer/
│   │   └── CartDrawer.jsx
│   ├── NotificationList/
│   │   └── NotificationList.jsx
│   └── ProtectedRoute/
│       └── ProtectedRoute.jsx
├── pages/
│   ├── HomePage.jsx
│   ├── LoginPage.jsx
│   └── CartPage.jsx
└── App.jsx
```

### NotificationContext.js

```jsx
import { createContext } from 'react';
export const NotificationContext = createContext(null);
NotificationContext.displayName = 'NotificationContext';
```

### NotificationProvider.jsx

```jsx
import { useState, useCallback, useMemo } from 'react';
import { NotificationContext } from './NotificationContext';

let nextId = 1;

export const NotificationProvider = ({ children }) => {
  const [notifications, setNotifications] = useState([]);

  const addNotification = useCallback((message, type = 'info') => {
    const id = nextId++;
    setNotifications((prev) => [...prev, { id, message, type }]);
    setTimeout(() => {
      setNotifications((prev) => prev.filter((n) => n.id !== id));
    }, 3000);
  }, []);

  const removeNotification = useCallback((id) => {
    setNotifications((prev) => prev.filter((n) => n.id !== id));
  }, []);

  const value = useMemo(
    () => ({ notifications, addNotification, removeNotification }),
    [notifications, addNotification, removeNotification]
  );

  return (
    <NotificationContext.Provider value={value}>
      {children}
    </NotificationContext.Provider>
  );
};
```

### useNotifications.js

```jsx
import { useContext } from 'react';
import { NotificationContext } from './NotificationContext';

export const useNotifications = () => {
  const context = useContext(NotificationContext);
  if (!context) throw new Error('useNotifications must be within NotificationProvider');
  return context;
};
```

### AppProviders.jsx

```jsx
import { AuthProvider } from '../contexts/auth';
import { ThemeProvider } from '../contexts/theme';
import { CartProvider } from '../contexts/cart';
import { NotificationProvider } from '../contexts/notifications';

export const AppProviders = ({ children }) => (
  <AuthProvider>
    <ThemeProvider>
      <CartProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </CartProvider>
    </ThemeProvider>
  </AuthProvider>
);
```

### Navbar.jsx

```jsx
import { useAuth } from '../../contexts/auth';
import { useTheme } from '../../contexts/theme';
import { useCart } from '../../contexts/cart';

export const Navbar = () => {
  const { isLoggedIn, user, logout } = useAuth();
  const { isDark, toggleTheme } = useTheme();
  const { itemCount } = useCart();

  const bg = isDark ? '#1a1a2e' : '#ffffff';
  const color = isDark ? '#ffffff' : '#1a1a2e';

  return (
    <nav style={{ padding: '16px 32px', background: bg, color, display: 'flex', justifyContent: 'space-between', alignItems: 'center', boxShadow: '0 2px 8px rgba(0,0,0,0.15)' }}>
      <strong style={{ fontSize: '22px' }}>🛍️ ShopSphere</strong>
      <div style={{ display: 'flex', gap: '16px', alignItems: 'center' }}>
        <button onClick={toggleTheme} style={{ background: 'none', border: '1px solid currentColor', borderRadius: '20px', padding: '6px 14px', cursor: 'pointer', color }}>
          {isDark ? '☀️ Light' : '🌙 Dark'}
        </button>
        <span>🛒 {itemCount}</span>
        {isLoggedIn ? (
          <>
            <span>Hi, {user.name.split(' ')[0]}!</span>
            <button onClick={logout} style={{ background: '#e74c3c', color: '#fff', border: 'none', borderRadius: '6px', padding: '6px 14px', cursor: 'pointer' }}>
              Logout
            </button>
          </>
        ) : (
          <span style={{ opacity: 0.7 }}>Not logged in</span>
        )}
      </div>
    </nav>
  );
};
```

### ProductCard.jsx

```jsx
import { useCart } from '../../contexts/cart';
import { useNotifications } from '../../contexts/notifications';
import { useTheme } from '../../contexts/theme';

export const ProductCard = ({ product }) => {
  const { addToCart } = useCart();
  const { addNotification } = useNotifications();
  const { isDark } = useTheme();

  const handleAdd = () => {
    addToCart(product);
    addNotification(`${product.name} added to cart!`, 'success');
  };

  return (
    <div style={{
      border: isDark ? '1px solid #333' : '1px solid #eee',
      borderRadius: '12px',
      padding: '20px',
      background: isDark ? '#16213e' : '#fff',
      color: isDark ? '#e0e0e0' : '#333',
      boxShadow: '0 2px 12px rgba(0,0,0,0.1)',
    }}>
      <h3 style={{ margin: '0 0 8px' }}>{product.name}</h3>
      <p style={{ margin: '0 0 4px', opacity: 0.7 }}>{product.description}</p>
      <p style={{ fontWeight: 'bold', color: isDark ? '#64ffda' : '#2ecc71', margin: '8px 0 16px' }}>
        ₹{product.price}
      </p>
      <button
        onClick={handleAdd}
        style={{
          background: '#2ecc71', color: '#fff', border: 'none',
          borderRadius: '8px', padding: '10px 20px', cursor: 'pointer',
          fontWeight: 'bold', width: '100%',
        }}
      >
        Add to Cart
      </button>
    </div>
  );
};
```

### NotificationList.jsx

```jsx
import { useNotifications } from '../../contexts/notifications';

const typeColors = { info: '#3498db', success: '#2ecc71', warning: '#f39c12', error: '#e74c3c' };

export const NotificationList = () => {
  const { notifications, removeNotification } = useNotifications();

  return (
    <div style={{ position: 'fixed', top: '80px', right: '24px', zIndex: 1000, display: 'flex', flexDirection: 'column', gap: '8px' }}>
      {notifications.map((n) => (
        <div
          key={n.id}
          style={{
            background: typeColors[n.type] || '#333',
            color: '#fff',
            padding: '12px 16px',
            borderRadius: '8px',
            boxShadow: '0 4px 12px rgba(0,0,0,0.2)',
            display: 'flex',
            justifyContent: 'space-between',
            gap: '16px',
            minWidth: '280px',
          }}
        >
          <span>{n.message}</span>
          <button
            onClick={() => removeNotification(n.id)}
            style={{ background: 'none', border: 'none', color: '#fff', cursor: 'pointer', fontWeight: 'bold' }}
          >
            ✕
          </button>
        </div>
      ))}
    </div>
  );
};
```

### HomePage.jsx

```jsx
import { useAuth } from '../contexts/auth';
import { ProductCard } from '../components/ProductCard/ProductCard';
import { LoginPage } from './LoginPage';

const PRODUCTS = [
  { id: 1, name: 'React Mastery Course', description: 'Master React from basics to advanced', price: 999 },
  { id: 2, name: 'TypeScript Handbook', description: 'Complete TypeScript for React developers', price: 799 },
  { id: 3, name: 'Node.js Backend Bootcamp', description: 'Build production-ready APIs', price: 1199 },
  { id: 4, name: 'UI/UX Design Fundamentals', description: 'Design beautiful, usable interfaces', price: 699 },
];

export const HomePage = () => {
  const { isLoggedIn } = useAuth();

  if (!isLoggedIn) return <LoginPage />;

  return (
    <main style={{ maxWidth: '900px', margin: '40px auto', padding: '0 24px' }}>
      <h1 style={{ marginBottom: '32px' }}>Featured Products</h1>
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(auto-fill, minmax(200px, 1fr))', gap: '24px' }}>
        {PRODUCTS.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </main>
  );
};
```

### App.jsx

```jsx
import { AppProviders } from './providers/AppProviders';
import { Navbar } from './components/Navbar/Navbar';
import { NotificationList } from './components/NotificationList/NotificationList';
import { HomePage } from './pages/HomePage';

function App() {
  return (
    <AppProviders>
      <Navbar />
      <NotificationList />
      <HomePage />
    </AppProviders>
  );
}

export default App;
```

---

## Summary

React Context API is one of the most powerful tools in the React ecosystem. Here's what you've learned:

| Chapter | Key Takeaway |
|---|---|
| 1–2 | State management problems that Context solves |
| 3–4 | Why Context exists and its architecture |
| 5–8 | Core APIs: createContext, Provider, children, useContext |
| 9 | Custom hooks are the right way to expose context |
| 10–12 | Real implementations: Theme, Auth, Cart |
| 13–15 | Composition, folder structure, module pattern |
| 16 | Performance: useMemo, useCallback, context splitting |
| 17 | Context vs Redux: choose the right tool |
| 18–19 | Common mistakes and legacy patterns |
| 20–21 | Real-world architectures and best practices |
| 22 | 90 interview questions with answers |
| 23 | Quick reference cheat sheet |
| 24 | Production-ready project: ShopSphere |

> **Next Steps:** Practice by building the ShopSphere app from scratch. Then extend it by adding TypeScript, a Wishlist context, and a Preferences context. Once you're comfortable, explore Zustand and Redux Toolkit to understand when to reach beyond Context API.

---

*Guide written for React 18+ | Modern patterns only | Interview-Ready*

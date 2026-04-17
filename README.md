# 02 — JSX in React (Deep Dive)

> **Course:** React — From Beginner to Production  
> **Topic:** JSX (JavaScript XML)  
> **Level:** Beginner → Advanced  
> **Prerequisites:** Basic HTML, Basic JavaScript (variables, functions, arrays, objects)

---

## Table of Contents

1. [What is JSX?](#1-what-is-jsx)
2. [Why Does JSX Exist?](#2-why-does-jsx-exist)
3. [How JSX Works Internally](#3-how-jsx-works-internally)
4. [JSX Syntax — Complete Guide](#4-jsx-syntax--complete-guide)
5. [Expressions in JSX](#5-expressions-in-jsx)
6. [JSX vs HTML — Key Differences](#6-jsx-vs-html--key-differences)
7. [Code Examples (Beginner → Advanced)](#7-code-examples-beginner--advanced)
8. [Real-World Use Cases](#8-real-world-use-cases)
9. [Best Practices](#9-best-practices)
10. [Common Mistakes](#10-common-mistakes)
11. [Performance Considerations](#11-performance-considerations)
12. [JSX vs Alternatives](#12-jsx-vs-alternatives)
13. [Interview Questions](#13-interview-questions)
14. [Practice Tasks](#14-practice-tasks)
15. [Summary](#15-summary)

---

## 1. What is JSX?

### Simple Explanation

JSX stands for **JavaScript XML**. It is a **syntax extension** for JavaScript that lets you write HTML-like code directly inside your JavaScript files.

Think of JSX as a **shorthand** or **template language** that makes it easy to describe what the UI should look like, using a syntax that feels like HTML.

```jsx
// This is JSX
const element = <h1>Hello, World!</h1>;
```

At first glance, this looks like you're storing HTML inside a JavaScript variable — and that's exactly the *feeling* JSX is designed to give you. But under the hood, it's 100% JavaScript.

### Technical Explanation

JSX is **syntactic sugar** over `React.createElement()` calls. When your code is processed by a tool like **Babel**, every JSX expression is transformed into a plain JavaScript function call that creates a React element (a JavaScript object describing the UI).

JSX is **not** a separate language. It is **not** a string. It is **not** HTML. It is a syntax extension that must be compiled/transpiled before browsers can run it.

---

## 2. Why Does JSX Exist?

### The Problem Before JSX

Before JSX, creating UI elements in React required calling `React.createElement()` directly. Here is what building even a simple card UI looked like:

```javascript
// Without JSX — Plain React.createElement calls
const element = React.createElement(
  'div',
  { className: 'card' },
  React.createElement(
    'h2',
    { className: 'card-title' },
    'Welcome to React'
  ),
  React.createElement(
    'p',
    { className: 'card-body' },
    'This is a paragraph inside the card.'
  ),
  React.createElement(
    'button',
    { onClick: handleClick, className: 'btn' },
    'Click Me'
  )
);
```

This is **verbose**, **difficult to read**, and **hard to maintain**. Visualizing the structure of the UI from this code is extremely challenging.

### The Solution JSX Provides

JSX lets you write the same structure in a way that closely mirrors the final HTML output:

```jsx
// With JSX — Clean, readable, and maintainable
const element = (
  <div className="card">
    <h2 className="card-title">Welcome to React</h2>
    <p className="card-body">This is a paragraph inside the card.</p>
    <button onClick={handleClick} className="btn">Click Me</button>
  </div>
);
```

### Why JSX is a Core Design Choice

| Reason | Explanation |
|--------|-------------|
| **Readability** | Developers can visually map JSX to the HTML output |
| **Familiar Syntax** | Web developers already know HTML; JSX lowers the learning curve |
| **Power of JavaScript** | Unlike pure HTML templates, JSX lets you embed full JS logic |
| **Compile-Time Error Checking** | Babel catches JSX errors at build time, not at runtime |
| **Tooling Support** | JSX has excellent editor support (IntelliSense, linting, formatting) |

---

## 3. How JSX Works Internally

Understanding how JSX is processed helps you debug errors and understand React's rendering model.

### Step 1: You Write JSX

```jsx
const element = <h1 className="title">Hello</h1>;
```

### Step 2: Babel Transforms JSX to JavaScript

Babel (or the TypeScript compiler) converts your JSX into `React.createElement()` calls:

```javascript
// What Babel outputs (React 17 and before)
const element = React.createElement(
  'h1',               // type — the element tag or component
  { className: 'title' }, // props — an object of attributes
  'Hello'             // children — content inside the element
);
```

### Step 3: React.createElement Returns a Plain JavaScript Object

`React.createElement()` does **not** create a real DOM node. It creates a **lightweight JavaScript object** called a **React element** (also informally called a "virtual DOM node"):

```javascript
// The React element object produced
{
  type: 'h1',
  props: {
    className: 'title',
    children: 'Hello'
  },
  key: null,
  ref: null,
  $$typeof: Symbol(react.element)
}
```

### Step 4: React Renders the Element to the Real DOM

React takes these JavaScript objects, compares them to the previous render (using the **reconciliation algorithm** / **virtual DOM diffing**), and makes the minimum number of actual DOM updates necessary.

```
JSX → Babel → React.createElement() → React Element (JS Object) → Virtual DOM → Real DOM
```

### The New JSX Transform (React 17+)

Since React 17, you no longer need to `import React from 'react'` in every file that uses JSX. The new JSX transform automatically imports the JSX runtime:

```jsx
// React 17+ — No need to import React
// Babel transforms JSX using the new jsx() function from 'react/jsx-runtime'

// Your code:
const element = <h1>Hello</h1>;

// Babel output (React 17+ new transform):
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { children: 'Hello' });
```

This is why modern React projects don't require `import React from 'react'` at the top of every file.

---

## 4. JSX Syntax — Complete Guide

### 4.1 Basic Element

```jsx
const heading = <h1>Hello World</h1>;
```

### 4.2 Self-Closing Tags

In JSX, **all tags must be closed**. HTML elements that don't have children must use self-closing syntax.

```jsx
// ✅ Correct — Self-closing
const input = <input type="text" />;
const image = <img src="logo.png" alt="Logo" />;
const lineBreak = <br />;

// ❌ Wrong — These will cause errors
const input = <input type="text">   // Error: JSX element is missing closing tag
```

### 4.3 Wrapping in a Parent Element

JSX expressions must have **exactly one root element**. You cannot return two sibling elements without a wrapper.

```jsx
// ❌ Error — Two root elements
return (
  <h1>Title</h1>
  <p>Paragraph</p>
);

// ✅ Option 1 — Wrap in a div
return (
  <div>
    <h1>Title</h1>
    <p>Paragraph</p>
  </div>
);

// ✅ Option 2 — Use React.Fragment (no extra DOM node)
return (
  <React.Fragment>
    <h1>Title</h1>
    <p>Paragraph</p>
  </React.Fragment>
);

// ✅ Option 3 — Shorthand Fragment syntax (most common)
return (
  <>
    <h1>Title</h1>
    <p>Paragraph</p>
  </>
);
```

> **When to prefer Fragment:** Use `<>...</>` when you don't want an extra `div` to appear in the DOM — useful for table rows, list items, or when a parent div would break CSS layouts.

### 4.4 className Instead of class

Since `class` is a reserved keyword in JavaScript, JSX uses `className`:

```jsx
// ✅ Correct
<div className="container">Content</div>

// ❌ Wrong — Will still work in some setups but triggers a warning
<div class="container">Content</div>
```

### 4.5 htmlFor Instead of for

Similarly, the HTML `for` attribute (used in `<label>`) becomes `htmlFor` in JSX:

```jsx
// ✅ Correct
<label htmlFor="email">Email Address</label>
<input id="email" type="email" />

// ❌ Wrong
<label for="email">Email Address</label>
```

### 4.6 Inline Styles

In JSX, inline styles are written as a **JavaScript object** (not a CSS string), and property names use **camelCase**:

```jsx
// ✅ Correct — Object with camelCase properties
<div style={{ backgroundColor: 'blue', fontSize: '16px', marginTop: '10px' }}>
  Styled content
</div>

// ❌ Wrong — CSS string syntax does not work in JSX
<div style="background-color: blue; font-size: 16px;">
  This will throw an error
</div>
```

> **The double curly braces `{{ }}`:** The outer `{}` is the JSX expression container. The inner `{}` is the JavaScript object literal. They are two different things.

### 4.7 Comments in JSX

Comments inside JSX must be wrapped in curly braces and use the `/* */` syntax:

```jsx
const element = (
  <div>
    {/* This is a valid JSX comment */}
    <p>Hello</p>
    {/* 
      Multi-line comment
      is also valid 
    */}
  </div>
);

// ❌ This will NOT work — HTML comments are not valid in JSX
<div>
  <!-- This will cause an error -->
</div>
```

### 4.8 Boolean Attributes

In HTML, you write `<input disabled>`. In JSX, boolean attributes should be explicitly set:

```jsx
// ✅ Correct approaches
<input disabled={true} />
<input disabled />          // Shorthand — value defaults to true
<input disabled={false} />  // Explicitly not disabled

// These are equivalent:
<button disabled />
<button disabled={true} />
```

### 4.9 Spreading Props

You can spread a JavaScript object as props using the spread operator:

```jsx
const buttonProps = {
  type: 'submit',
  className: 'btn-primary',
  disabled: false
};

// ✅ Spread all props at once
<button {...buttonProps}>Submit</button>

// This is equivalent to:
<button type="submit" className="btn-primary" disabled={false}>Submit</button>
```

---

## 5. Expressions in JSX

JSX is powerful because you can embed **any valid JavaScript expression** inside `{}` curly braces.

### 5.1 Variables

```jsx
const name = 'Rahul';
const age = 25;

const element = (
  <p>
    My name is {name} and I am {age} years old.
  </p>
);
// Output: My name is Rahul and I am 25 years old.
```

### 5.2 Arithmetic

```jsx
const price = 100;
const discount = 20;

<p>You pay: ₹{price - discount}</p>
// Output: You pay: ₹80
```

### 5.3 Function Calls

```jsx
function formatName(user) {
  return `${user.firstName} ${user.lastName}`;
}

const user = { firstName: 'Priya', lastName: 'Sharma' };

<h1>Welcome, {formatName(user)}!</h1>
// Output: Welcome, Priya Sharma!
```

### 5.4 Ternary Operator (Conditional Rendering)

```jsx
const isLoggedIn = true;

<div>
  {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in.</p>}
</div>
```

### 5.5 Logical AND (&&) Operator

The `&&` operator is used to conditionally render something only when a condition is true:

```jsx
const hasNotifications = true;
const notificationCount = 5;

<div>
  {hasNotifications && <span className="badge">{notificationCount}</span>}
</div>
```

> **Important Gotcha with `&&`:** If the left side evaluates to `0` (number zero), React will render `0` instead of nothing. This is a common bug!

```jsx
// ❌ Bug — If items.length is 0, React renders "0" on screen
{items.length && <ItemList items={items} />}

// ✅ Fix — Convert to a proper boolean
{items.length > 0 && <ItemList items={items} />}
// or
{!!items.length && <ItemList items={items} />}
```

### 5.6 Rendering Lists with .map()

```jsx
const fruits = ['Apple', 'Banana', 'Mango'];

<ul>
  {fruits.map((fruit, index) => (
    <li key={index}>{fruit}</li>
  ))}
</ul>
```

> **The `key` prop is required** when rendering lists. React uses it to identify which items changed, were added, or removed. We'll cover this in depth in the Lists & Keys module.

### 5.7 What You CANNOT Put in JSX Expressions

Not every JavaScript construct can be used inside `{}`. Only **expressions** (things that evaluate to a value) are allowed — **not statements**.

```jsx
// ❌ if/else statements — NOT allowed directly in JSX
<div>
  {if (isLoggedIn) { return <p>Hi</p> }}  // Syntax Error
</div>

// ✅ Use ternary instead
<div>
  {isLoggedIn ? <p>Hi</p> : <p>Login</p>}
</div>

// ❌ for loops — NOT allowed
<div>
  {for (let i = 0; i < 3; i++) { ... }}  // Syntax Error
</div>

// ✅ Use .map() instead
<div>
  {[1, 2, 3].map(i => <p key={i}>{i}</p>)}
</div>
```

---

## 6. JSX vs HTML — Key Differences

This is a critical reference table for anyone coming from an HTML background:

| Feature | HTML | JSX |
|---------|------|-----|
| Attribute: class | `class="..."` | `className="..."` |
| Attribute: for | `for="..."` | `htmlFor="..."` |
| Inline styles | `style="color: red"` (string) | `style={{ color: 'red' }}` (object) |
| Self-closing tags | `<img>`, `<br>`, `<input>` | `<img />`, `<br />`, `<input />` |
| Comments | `<!-- comment -->` | `{/* comment */}` |
| Event names | `onclick`, `onchange` | `onClick`, `onChange` (camelCase) |
| Boolean attributes | `disabled` | `disabled` or `disabled={true}` |
| Root element | Multiple roots allowed | Single root required |
| Custom data attrs | `data-id="1"` | `data-id="1"` (same — works fine) |
| Tab index | `tabindex="1"` | `tabIndex="1"` |
| aria attributes | `aria-label="..."` | `aria-label="..."` (same — works fine) |

---

## 7. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Simple JSX Element

```jsx
// File: SimpleGreeting.jsx

function SimpleGreeting() {
  const name = 'Ananya';
  const currentHour = new Date().getHours();
  const greeting = currentHour < 12 ? 'Good Morning' : 'Good Evening';

  return (
    <div className="greeting-card">
      <h1>{greeting}, {name}! 👋</h1>
      <p>Welcome to your React dashboard.</p>
    </div>
  );
}

export default SimpleGreeting;
```

**What this demonstrates:**
- JSX element structure
- Embedding variables inside JSX
- Using `className`
- Using ternary operator for logic

---

### Example 2 — Intermediate: Dynamic List with Conditional Rendering

```jsx
// File: ProductList.jsx

const products = [
  { id: 1, name: 'Wireless Headphones', price: 2999, inStock: true },
  { id: 2, name: 'Mechanical Keyboard', price: 4500, inStock: false },
  { id: 3, name: 'USB-C Hub', price: 1299, inStock: true },
];

function ProductList() {
  return (
    <div className="product-list">
      <h2>Our Products</h2>

      {products.length === 0 ? (
        <p className="empty-message">No products available right now.</p>
      ) : (
        <ul>
          {products.map((product) => (
            <li key={product.id} className="product-item">
              <span className="product-name">{product.name}</span>
              <span className="product-price">₹{product.price.toLocaleString()}</span>
              {product.inStock ? (
                <span className="badge badge--green">In Stock</span>
              ) : (
                <span className="badge badge--red">Out of Stock</span>
              )}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default ProductList;
```

**What this demonstrates:**
- Rendering a list using `.map()`
- Using `key` prop
- Nested conditional rendering with ternary
- Dynamic `className` patterns

---

### Example 3 — Intermediate: JSX with Event Handlers & Styling

```jsx
// File: AlertButton.jsx

function AlertButton({ label, variant = 'primary', onClick }) {
  const baseStyle = {
    padding: '10px 20px',
    borderRadius: '6px',
    border: 'none',
    cursor: 'pointer',
    fontWeight: '600',
    fontSize: '14px',
  };

  const variants = {
    primary: { backgroundColor: '#4f46e5', color: '#ffffff' },
    danger: { backgroundColor: '#ef4444', color: '#ffffff' },
    ghost: { backgroundColor: 'transparent', color: '#4f46e5', border: '1px solid #4f46e5' },
  };

  const computedStyle = { ...baseStyle, ...variants[variant] };

  return (
    <button style={computedStyle} onClick={onClick}>
      {label}
    </button>
  );
}

// Usage
function App() {
  return (
    <div style={{ display: 'flex', gap: '12px', padding: '20px' }}>
      <AlertButton label="Save Changes" variant="primary" onClick={() => alert('Saved!')} />
      <AlertButton label="Delete" variant="danger" onClick={() => alert('Deleted!')} />
      <AlertButton label="Cancel" variant="ghost" onClick={() => alert('Cancelled!')} />
    </div>
  );
}
```

**What this demonstrates:**
- JSX props with default values
- Dynamic inline styles via computed objects
- Event handler props
- Component composition

---

### Example 4 — Advanced: Real-World Dashboard Card Component

```jsx
// File: components/DashboardCard/DashboardCard.jsx

/**
 * DashboardCard — A reusable metric card for admin dashboards
 *
 * Props:
 *  - title: string
 *  - value: string | number
 *  - icon: React element
 *  - trend: 'up' | 'down' | null
 *  - trendValue: string (e.g., "+12%")
 *  - isLoading: boolean
 */

function DashboardCard({ title, value, icon, trend, trendValue, isLoading = false }) {
  const trendStyle = {
    up: { color: '#16a34a', label: '▲' },
    down: { color: '#dc2626', label: '▼' },
  };

  if (isLoading) {
    return (
      <div className="dashboard-card dashboard-card--skeleton">
        <div className="skeleton skeleton--title" />
        <div className="skeleton skeleton--value" />
        <div className="skeleton skeleton--trend" />
      </div>
    );
  }

  return (
    <div className="dashboard-card">
      <div className="dashboard-card__header">
        <span className="dashboard-card__title">{title}</span>
        {icon && <span className="dashboard-card__icon">{icon}</span>}
      </div>

      <div className="dashboard-card__body">
        <span className="dashboard-card__value">{value}</span>

        {trend && trendValue && (
          <span
            className="dashboard-card__trend"
            style={{ color: trendStyle[trend].color }}
            aria-label={`Trend: ${trend === 'up' ? 'increase' : 'decrease'} of ${trendValue}`}
          >
            {trendStyle[trend].label} {trendValue}
          </span>
        )}
      </div>
    </div>
  );
}

// Usage in a parent component
function AdminDashboard() {
  const stats = [
    { title: 'Total Revenue', value: '₹4,29,850', trend: 'up', trendValue: '+18%' },
    { title: 'New Users', value: '1,284', trend: 'up', trendValue: '+7%' },
    { title: 'Refunds', value: '₹12,400', trend: 'down', trendValue: '-3%' },
    { title: 'Active Sessions', value: '342', trend: null, trendValue: null },
  ];

  return (
    <section className="dashboard">
      <h1 className="dashboard__heading">Analytics Overview</h1>
      <div className="dashboard__grid">
        {stats.map((stat, idx) => (
          <DashboardCard key={idx} {...stat} />
        ))}
      </div>
    </section>
  );
}

export default AdminDashboard;
```

**What this demonstrates:**
- Prop-driven, reusable JSX component
- Conditional rendering (loading state / skeleton)
- Spreading props (`{...stat}`)
- Accessibility via `aria-label`
- Real-world dashboard patterns

---

## 8. Real-World Use Cases

### 8.1 Conditional UI Based on Authentication

```jsx
function Navbar({ user }) {
  return (
    <nav className="navbar">
      <a href="/" className="navbar__logo">MyApp</a>

      <div className="navbar__actions">
        {user ? (
          <>
            <span>Hello, {user.name}</span>
            <button onClick={logout}>Logout</button>
          </>
        ) : (
          <>
            <a href="/login">Login</a>
            <a href="/signup" className="btn-primary">Sign Up</a>
          </>
        )}
      </div>
    </nav>
  );
}
```

### 8.2 Rendering API Data

```jsx
function UserDirectory({ users, isLoading, error }) {
  if (isLoading) return <p>Loading users...</p>;
  if (error) return <p className="error">Failed to load: {error.message}</p>;
  if (!users || users.length === 0) return <p>No users found.</p>;

  return (
    <div className="user-grid">
      {users.map((user) => (
        <div key={user.id} className="user-card">
          <img src={user.avatar} alt={`${user.name}'s avatar`} />
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
}
```

### 8.3 Form Elements

```jsx
function LoginForm({ onSubmit }) {
  return (
    <form onSubmit={onSubmit} className="login-form" noValidate>
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          name="email"
          placeholder="you@example.com"
          autoComplete="email"
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          name="password"
          placeholder="Enter password"
          minLength={8}
          required
        />
      </div>

      <button type="submit" className="btn btn-primary">
        Sign In
      </button>
    </form>
  );
}
```

---

## 9. Best Practices

### 9.1 Keep JSX Clean — Extract Complex Logic

```jsx
// ❌ Bad — Logic clutters JSX
function OrderStatus({ order }) {
  return (
    <p style={{ color: order.status === 'delivered' ? 'green' : order.status === 'cancelled' ? 'red' : 'orange' }}>
      {order.status === 'delivered' ? '✅ Delivered' : order.status === 'cancelled' ? '❌ Cancelled' : '⏳ Pending'}
    </p>
  );
}

// ✅ Good — Extract logic to helper functions
function getOrderStatusConfig(status) {
  const config = {
    delivered: { color: 'green', label: '✅ Delivered' },
    cancelled: { color: 'red', label: '❌ Cancelled' },
    pending: { color: 'orange', label: '⏳ Pending' },
  };
  return config[status] ?? { color: 'gray', label: 'Unknown' };
}

function OrderStatus({ order }) {
  const { color, label } = getOrderStatusConfig(order.status);
  return <p style={{ color }}>{label}</p>;
}
```

### 9.2 Use Fragments to Avoid Unnecessary DOM Nodes

```jsx
// ❌ Unnecessary div wrapper
function TableRow({ data }) {
  return (
    <div>   {/* This breaks the table structure! */}
      <td>{data.name}</td>
      <td>{data.age}</td>
    </div>
  );
}

// ✅ Fragment — No extra DOM node
function TableRow({ data }) {
  return (
    <>
      <td>{data.name}</td>
      <td>{data.age}</td>
    </>
  );
}
```

### 9.3 Always Use Descriptive `key` Props (Not Index)

```jsx
// ❌ Avoid using index as key when list can reorder or change
{todos.map((todo, index) => (
  <TodoItem key={index} todo={todo} />
))}

// ✅ Use a stable unique identifier
{todos.map((todo) => (
  <TodoItem key={todo.id} todo={todo} />
))}
```

### 9.4 One Component Per File

```
src/
└── components/
    ├── Button/
    │   ├── Button.jsx
    │   ├── Button.module.css
    │   └── index.js
    ├── Card/
    │   ├── Card.jsx
    │   └── index.js
    └── Navbar/
        ├── Navbar.jsx
        └── index.js
```

### 9.5 Add `aria` Attributes for Accessibility

```jsx
// ✅ Screen readers can understand this
<button
  aria-label="Close dialog"
  aria-expanded={isOpen}
  onClick={closeDialog}
>
  ✕
</button>
```

### 9.6 Avoid Deeply Nested JSX — Break Into Sub-components

```jsx
// ❌ Too nested — hard to read
function Page() {
  return (
    <div>
      <div>
        <div>
          <ul>
            {items.map(item => (
              <li>
                <div>
                  <h3>{item.title}</h3>
                  <p>{item.desc}</p>
                </div>
              </li>
            ))}
          </ul>
        </div>
      </div>
    </div>
  );
}

// ✅ Break into smaller components
function ItemCard({ title, desc }) {
  return (
    <div className="item-card">
      <h3>{title}</h3>
      <p>{desc}</p>
    </div>
  );
}

function ItemList({ items }) {
  return (
    <ul className="item-list">
      {items.map(item => (
        <li key={item.id}>
          <ItemCard title={item.title} desc={item.desc} />
        </li>
      ))}
    </ul>
  );
}

function Page() {
  return (
    <div className="page">
      <ItemList items={items} />
    </div>
  );
}
```

---

## 10. Common Mistakes

### Mistake 1: Forgetting to Close Tags

```jsx
// ❌ Error
<input type="text">
<img src="logo.png">

// ✅ Fixed
<input type="text" />
<img src="logo.png" />
```

### Mistake 2: Using `class` Instead of `className`

```jsx
// ❌ Works but triggers warning in development
<div class="container">...</div>

// ✅ Correct
<div className="container">...</div>
```

### Mistake 3: Returning Multiple Root Elements Without a Wrapper

```jsx
// ❌ Syntax Error
return (
  <h1>Title</h1>
  <p>Content</p>
);

// ✅ Correct
return (
  <>
    <h1>Title</h1>
    <p>Content</p>
  </>
);
```

### Mistake 4: The `&&` Zero Bug

```jsx
// ❌ Bug — Renders "0" when cart is empty
{cart.length && <CartBadge count={cart.length} />}

// ✅ Fixed
{cart.length > 0 && <CartBadge count={cart.length} />}
```

### Mistake 5: Inline Functions Creating Performance Issues

```jsx
// ❌ Creates a new function on every render
<button onClick={() => handleDelete(item.id)}>Delete</button>

// ✅ Better for performance-sensitive lists — Use useCallback (covered in hooks module)
const handleDeleteItem = useCallback(() => handleDelete(item.id), [item.id]);
<button onClick={handleDeleteItem}>Delete</button>
```

### Mistake 6: Forgetting `key` Prop in Lists

```jsx
// ❌ Triggers "missing key" warning in dev tools
{users.map(user => <UserCard user={user} />)}

// ✅ Always provide key
{users.map(user => <UserCard key={user.id} user={user} />)}
```

### Mistake 7: Putting Statements in JSX Expressions

```jsx
// ❌ Statements are not allowed inside {}
{if (show) { <Modal /> }}

// ✅ Use ternary
{show ? <Modal /> : null}

// ✅ Or logical AND
{show && <Modal />}
```

### Mistake 8: Using `style` as a String

```jsx
// ❌ Error — style must be an object, not a string
<p style="color: red; font-size: 16px">Text</p>

// ✅ Correct — style as a JavaScript object
<p style={{ color: 'red', fontSize: '16px' }}>Text</p>
```

---

## 11. Performance Considerations

### 11.1 Avoid Creating Objects/Arrays Inline in JSX

Creating new objects or arrays inside the JSX causes **unnecessary re-renders** for child components that receive them as props, because a new reference is created on every render.

```jsx
// ❌ New object created on every render
<Chart options={{ responsive: true, legend: false }} data={chartData} />

// ✅ Define outside the component or memoize
const chartOptions = { responsive: true, legend: false }; // Defined outside

function MyChart() {
  return <Chart options={chartOptions} data={chartData} />;
}
```

### 11.2 Avoid Expensive Computations Directly in JSX

```jsx
// ❌ Recalculates on every render
function ProductList({ products }) {
  return (
    <div>
      <p>Total: ₹{products.reduce((sum, p) => sum + p.price, 0)}</p>
      {/* ... */}
    </div>
  );
}

// ✅ Use useMemo to cache the computed value
function ProductList({ products }) {
  const total = useMemo(
    () => products.reduce((sum, p) => sum + p.price, 0),
    [products]
  );

  return (
    <div>
      <p>Total: ₹{total}</p>
      {/* ... */}
    </div>
  );
}
```

### 11.3 Conditional Rendering vs CSS display:none

```jsx
// ❌ Modal is always in the DOM, just hidden — wastes memory
<Modal style={{ display: isOpen ? 'block' : 'none' }} />

// ✅ Modal is only mounted when needed
{isOpen && <Modal />}
```

---

## 12. JSX vs Alternatives

| Approach | Example | Pros | Cons |
|----------|---------|------|------|
| **JSX** | `<Button color="red">Click</Button>` | Readable, familiar, tool support | Requires build step (Babel) |
| **React.createElement** | `React.createElement(Button, {color:'red'}, 'Click')` | No build step needed | Verbose, hard to read |
| **Template Literals (Lit)** | `` html`<button>Click</button>` `` | No build step | Different ecosystem (web components) |
| **Vue SFC templates** | `<template><button>Click</button></template>` | Clean separation | Vue only, different paradigm |

**Use JSX when:**
- Building React applications (standard and recommended)
- You want full JavaScript power inside templates
- You want excellent TypeScript/IDE integration

**Consider avoiding JSX when:**
- You need to render React on environments with no build tooling (rare)
- You're using a framework that has its own templating system

---

## 13. Interview Questions

### Q1: What is JSX and is it required to use React?

**Answer:** JSX is a syntax extension for JavaScript that allows you to write HTML-like code inside JS files. It is **not required** — you can use `React.createElement()` directly. However, JSX is the standard approach because it is far more readable and maintainable. Babel or the TypeScript compiler transforms JSX into `React.createElement()` calls.

---

### Q2: What is the difference between JSX and HTML?

**Answer:** Key differences include: `className` instead of `class`, `htmlFor` instead of `for`, camelCase event names (`onClick`, `onChange`), inline styles as JavaScript objects, all tags must be self-closed, exactly one root element required, and JavaScript comments use `{/* */}` not `<!-- -->`.

---

### Q3: What does Babel do with JSX?

**Answer:** Babel transforms JSX into `React.createElement()` calls. For example, `<h1 className="title">Hello</h1>` becomes `React.createElement('h1', { className: 'title' }, 'Hello')`. With React 17's new JSX transform, it uses the `jsx()` function from `react/jsx-runtime` instead, which no longer requires importing React at the top of each file.

---

### Q4: Why do we need the `key` prop when rendering lists?

**Answer:** React uses the `key` prop to identify which items in a list have changed, been added, or removed. Without unique and stable keys, React may re-render the wrong components, causing bugs — especially with stateful list items like form inputs. Keys must be unique among siblings and should ideally be stable unique IDs, not array indices.

---

### Q5: Can you put any JavaScript inside JSX `{}` curly braces?

**Answer:** No. Only **JavaScript expressions** can go inside `{}` — not statements. Expressions produce a value: variables, arithmetic, ternary operators, function calls, `.map()`, etc. Statements like `if/else`, `for` loops, `switch`, and `while` loops cannot be used directly. You should use ternary or logical operators for conditionals, and `.map()` for loops.

---

### Q6: What is the difference between `<React.Fragment>` and `<>`?

**Answer:** Both are identical in behavior — they render their children without adding an extra DOM node. The only difference is that `<React.Fragment key={...}>` supports the `key` attribute (useful when mapping over a list of fragments), while `<>` is shorthand syntax that does not support attributes.

---

### Q7: What happens if you use `{0 && <Component />}` in JSX?

**Answer:** This is a common bug. When the left operand of `&&` is `0` (falsy), JavaScript short-circuits and returns `0`. React will then render the number `0` on screen, which is not what you want. The fix is to ensure the left side is a proper boolean: `{items.length > 0 && <Component />}` or `{!!items.length && <Component />}`.

---

### Q8: How do you write inline styles in JSX?

**Answer:** Inline styles in JSX are JavaScript objects — not CSS strings. Property names use camelCase. You use double curly braces: the outer `{}` is the JSX expression delimiter, and the inner `{}` is the object literal. Example: `<div style={{ backgroundColor: 'blue', fontSize: '16px' }}>`.

---

### Q9: What is the spread operator used for in JSX?

**Answer:** The spread operator (`...`) lets you pass all properties of a JavaScript object as props to a component at once. For example, `<Button {...buttonProps} />` is equivalent to writing each property individually. It's useful for forwarding props, composing components, or passing a config object as props.

---

### Q10: What is the new JSX Transform introduced in React 17?

**Answer:** The new JSX Transform allows you to use JSX without importing React at the top of every file. Previously, Babel converted JSX to `React.createElement()`, which required React in scope. The new transform generates imports from `react/jsx-runtime` automatically, so you no longer need `import React from 'react'` in every component file.

---

## 14. Practice Tasks

### Task 1 — Beginner: Profile Card

Build a `ProfileCard` component that accepts props and displays them using JSX.

**Requirements:**
- Accept `name`, `role`, `avatarUrl`, and `bio` as props
- Display the avatar image, name (in a heading), role (in a subheading), and bio (in a paragraph)
- Add a className to each element for potential CSS styling
- Use `htmlFor` and `id` correctly if you add any labels
- Export the component

**Expected output structure:**
```
ProfileCard
├── img (avatar)
├── h2 (name)
├── h4 (role)
└── p (bio)
```

---

### Task 2 — Intermediate: Feature List with Conditional Badges

Build a `FeatureList` component that renders a list of software features.

**Requirements:**
- Define an array of feature objects: `{ id, name, description, isPremium, isNew }`
- Use `.map()` to render each feature as a card
- Show a "Premium" badge (golden) if `isPremium` is true
- Show a "New" badge (green) if `isNew` is true
- If no features are in the array, render a "No features available" message
- Use `React.Fragment` or `<>` to avoid unnecessary wrappers
- Use proper `key` props

---

### Task 3 — Advanced: Reusable Alert Component

Build a flexible, accessible `Alert` component.

**Requirements:**
- Accept props: `type` (`'success' | 'error' | 'warning' | 'info'`), `title`, `message`, `onClose` (optional function), `isDismissible` (boolean)
- Dynamically compute `className` based on the `type` prop (e.g., `alert--success`, `alert--error`)
- Dynamically compute inline icon: success → ✅, error → ❌, warning → ⚠️, info → ℹ️
- If `isDismissible` is true, render a close button (`✕`) that calls `onClose`
- Add proper `role="alert"` attribute for accessibility
- Add `aria-live="polite"` for screen reader support
- Create a parent `App` component that renders multiple alerts with a button to dismiss each

---

## 15. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| What is JSX | Syntax extension — HTML-like code in JavaScript, compiled to `React.createElement()` |
| Why JSX | Makes UI structure readable, maintainable, and leverages full JavaScript power |
| Transformation | Babel converts JSX → `React.createElement()` → React element object |
| Root element | JSX must have exactly one root — use `<>` Fragment to avoid extra DOM nodes |
| className | Use `className` not `class`; `htmlFor` not `for` |
| Inline styles | Object with camelCase properties: `{{ backgroundColor: 'blue' }}` |
| Expressions | Use `{}` for any JS expression — variables, ternary, `.map()`, function calls |
| No statements | Cannot use `if/else`, `for`, `while` directly — use ternary and `.map()` |
| Lists | Always provide a stable, unique `key` prop — avoid using array index |
| `&&` gotcha | `{0 && <C />}` renders `0` — use `{count > 0 && <C />}` instead |
| Spread props | `<Comp {...obj} />` spreads object as individual props |
| New JSX transform | React 17+ — no need to import React just for JSX |

### Important Points to Remember

1. **JSX is not HTML** — It only *looks* like HTML. It compiles to JavaScript.
2. **JSX is not a string** — You cannot do string operations on it.
3. **Every JSX expression is an object** — React elements are plain JS objects describing UI.
4. **One root element rule** — Use Fragments if you don't want extra DOM nodes.
5. **Only expressions in `{}`** — Not statements.
6. **Keys matter** — Use stable unique IDs for list items.
7. **camelCase everything** — Attributes, event names, and style properties all use camelCase in JSX.
8. **Accessibility is part of JSX** — Always add `aria-*` attributes and semantic HTML tags.

---

> **Next Topic:** `03-components-and-props.md` — Building and composing React components, passing data with props, and understanding the component tree.


# 03 — Components & Props in React (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** Components & Props
> **Level:** Beginner → Advanced
> **Prerequisites:** JSX (Module 02), Basic JavaScript (functions, objects, destructuring)

---

## Table of Contents

1. [What is a Component?](#1-what-is-a-component)
2. [Why Components Exist](#2-why-components-exist)
3. [How Components Work Internally](#3-how-components-work-internally)
4. [Types of Components](#4-types-of-components)
5. [What are Props?](#5-what-are-props)
6. [Props — Complete Syntax Guide](#6-props--complete-syntax-guide)
7. [The `children` Prop](#7-the-children-prop)
8. [Prop Types & Default Props](#8-prop-types--default-props)
9. [Component Composition](#9-component-composition)
10. [Code Examples (Beginner → Advanced)](#10-code-examples-beginner--advanced)
11. [Real-World Use Cases](#11-real-world-use-cases)
12. [Best Practices](#12-best-practices)
13. [Common Mistakes](#13-common-mistakes)
14. [Performance Considerations](#14-performance-considerations)
15. [Comparison: Class vs Function Components](#15-comparison-class-vs-function-components)
16. [Interview Questions](#16-interview-questions)
17. [Practice Tasks](#17-practice-tasks)
18. [Summary](#18-summary)

---

## 1. What is a Component?

### Simple Explanation

A **component** is a reusable, self-contained piece of UI. Think of your entire web application as a LEGO set — each individual LEGO brick is a component. You build complex structures by combining small, simple bricks.

Every button, navbar, card, modal, form, and page in a React application is (or should be) a component.

```jsx
// The simplest possible React component
function Hello() {
  return <h1>Hello, World!</h1>;
}
```

### Technical Explanation

A React component is a **JavaScript function** (or class) that:
1. Accepts an optional input object called **props**
2. Returns **React elements** (JSX) describing what should appear on screen

React then takes that description and renders it into the actual DOM.

---

## 2. Why Components Exist

### The Problem: Monolithic HTML

Imagine building a large e-commerce site with plain HTML. Your `index.html` would be thousands of lines long, you'd repeat the same product card HTML dozens of times, and changing the card design means finding and editing every single copy.

```html
<!-- Without components — repeated structure everywhere -->
<div class="product-card">
  <img src="shoes.jpg" alt="Shoes" />
  <h3>Running Shoes</h3>
  <p>₹2,999</p>
  <button>Add to Cart</button>
</div>

<div class="product-card">
  <img src="tshirt.jpg" alt="T-Shirt" />
  <h3>Cotton T-Shirt</h3>
  <p>₹799</p>
  <button>Add to Cart</button>
</div>

<!-- ...50 more just like these -->
```

### The Solution: Components

With components, you define the structure **once** and reuse it with different data:

```jsx
// Define once
function ProductCard({ image, name, price }) {
  return (
    <div className="product-card">
      <img src={image} alt={name} />
      <h3>{name}</h3>
      <p>₹{price}</p>
      <button>Add to Cart</button>
    </div>
  );
}

// Reuse everywhere
<ProductCard image="shoes.jpg" name="Running Shoes" price={2999} />
<ProductCard image="tshirt.jpg" name="Cotton T-Shirt" price={799} />
```

### Benefits of Components

| Benefit | Explanation |
|---------|-------------|
| **Reusability** | Write once, use many times with different data |
| **Separation of Concerns** | Each component handles one piece of the UI |
| **Maintainability** | Change one component, update everywhere it's used |
| **Testability** | Components can be unit tested in isolation |
| **Collaboration** | Teams can work on different components simultaneously |
| **Readability** | `<UserProfile />` is clearer than 50 lines of HTML |

---

## 3. How Components Work Internally

### The Rendering Pipeline

When React encounters a component in JSX, here's what happens:

```
1. React sees <ProductCard name="Shoes" price={2999} />
2. React collects all attributes into a props object: { name: "Shoes", price: 2999 }
3. React calls your function: ProductCard({ name: "Shoes", price: 2999 })
4. Your function returns JSX (a React element tree)
5. React reconciles the returned elements with the current DOM
6. React updates only the parts of the DOM that changed
```

### Components as Pure Functions (Conceptually)

The mental model for a component is a **pure function**:

```
UI = Component(props)
```

Given the same props, a component should always return the same UI. This predictability is what makes React applications easy to reason about.

```jsx
// Pure — same props always produce same output
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Calling it with the same name always gives the same result
Greeting({ name: 'Rahul' }) // → <h1>Hello, Rahul!</h1>
Greeting({ name: 'Rahul' }) // → <h1>Hello, Rahul!</h1> (always)
```

### Component Identity and the Component Tree

Every React application has a **component tree** — a hierarchy of components from the root down to the leaf nodes.

```
App
├── Navbar
│   ├── Logo
│   └── NavLinks
│       ├── NavLink (Home)
│       ├── NavLink (Products)
│       └── NavLink (Contact)
├── Main
│   ├── HeroBanner
│   └── ProductGrid
│       ├── ProductCard
│       ├── ProductCard
│       └── ProductCard
└── Footer
    ├── FooterLinks
    └── Copyright
```

React renders this tree top-down. Data flows **downward** through props. Events flow **upward** through callback functions.

---

## 4. Types of Components

### 4.1 Function Components (Modern Standard)

Function components are plain JavaScript functions. They are the **standard way** to write components in React today.

```jsx
function Welcome({ name }) {
  return <h2>Welcome, {name}!</h2>;
}

// Arrow function syntax — equally valid
const Welcome = ({ name }) => <h2>Welcome, {name}!</h2>;
```

### 4.2 Class Components (Legacy)

Class components were the original way to create stateful components before React Hooks (introduced in React 16.8). You may encounter them in older codebases.

```jsx
import { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h2>Welcome, {this.props.name}!</h2>;
  }
}
```

> **Today's Standard:** Write all new components as **function components**. Class components are considered legacy. React will continue to support them but no new features are being added to them.

### 4.3 Presentational vs Container Components (Pattern)

This is an **architectural pattern**, not a React feature:

| Type | Also Called | Responsibility |
|------|-------------|----------------|
| **Presentational** | Dumb, UI, Stateless | How things *look* — pure JSX with props |
| **Container** | Smart, Stateful | How things *work* — fetches data, manages state |

```jsx
// Presentational — only concerned with display
function UserCard({ name, email, avatarUrl }) {
  return (
    <div className="user-card">
      <img src={avatarUrl} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

// Container — concerned with data and logic
function UserCardContainer({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  if (!user) return <Spinner />;
  return <UserCard name={user.name} email={user.email} avatarUrl={user.avatar} />;
}
```

---

## 5. What are Props?

### Simple Explanation

**Props** (short for *properties*) are the way you pass data **into** a component from its parent. They are to components what arguments are to functions.

```jsx
// Props are like function arguments
function greet(name) {           // regular function
  return `Hello, ${name}!`;
}

function Greet({ name }) {       // React component
  return <p>Hello, {name}!</p>;
}

// Calling them:
greet('Priya');                  // → "Hello, Priya!"
<Greet name="Priya" />          // → <p>Hello, Priya!</p>
```

### The Props Object

When React calls your component, it passes **all the JSX attributes** as a single JavaScript object — the `props` object.

```jsx
// When you write this:
<UserProfile name="Riya" age={24} isAdmin={true} />

// React calls your component with this object:
UserProfile({
  name: "Riya",
  age: 24,
  isAdmin: true
})
```

### Props are Read-Only (CRITICAL RULE)

Props must **never be modified** inside a component. This is one of React's fundamental rules.

```jsx
// ❌ NEVER do this — mutating props is forbidden
function BadComponent(props) {
  props.name = 'Changed'; // This is a violation of React's rules
  return <p>{props.name}</p>;
}

// ✅ Props are read-only — use them as-is
function GoodComponent({ name }) {
  return <p>{name}</p>;
}
```

**Why?** React's rendering model is built on the assumption that props are immutable. Mutating props breaks the data flow and leads to unpredictable bugs.

---

## 6. Props — Complete Syntax Guide

### 6.1 Passing String Props

```jsx
// String — use quotes
<Button label="Click Me" />
<Avatar alt="User profile picture" />
```

### 6.2 Passing Number Props

```jsx
// Numbers — use curly braces
<Rating stars={4} />
<Pagination totalPages={10} currentPage={1} />

// ❌ This passes the string "4", not the number 4
<Rating stars="4" />
```

### 6.3 Passing Boolean Props

```jsx
// Shorthand — just the name means true
<Modal isOpen />            // isOpen={true}
<Input disabled />          // disabled={true}

// Explicit false
<Modal isOpen={false} />
```

### 6.4 Passing Arrays and Objects

```jsx
const tags = ['React', 'JavaScript', 'Frontend'];
const user = { name: 'Arjun', role: 'admin' };

<TagList tags={tags} />
<UserBadge user={user} />
```

### 6.5 Passing Functions as Props (Callbacks)

This is how **child-to-parent communication** works in React:

```jsx
function Parent() {
  function handleDelete(id) {
    console.log('Delete item:', id);
  }

  return <Child onDelete={handleDelete} />;
}

function Child({ onDelete }) {
  return (
    <button onClick={() => onDelete(42)}>
      Delete Item
    </button>
  );
}
```

### 6.6 Passing JSX as Props

You can pass entire JSX elements as prop values:

```jsx
<Card
  header={<h2>Card Title</h2>}
  footer={<button>Submit</button>}
/>
```

### 6.7 Destructuring Props (Recommended)

Instead of accessing `props.name`, destructure directly in the function signature:

```jsx
// ❌ Verbose — using props.xxx every time
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.email}</p>
      <span>{props.role}</span>
    </div>
  );
}

// ✅ Clean — destructure in the parameter
function UserCard({ name, email, role }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <span>{role}</span>
    </div>
  );
}
```

### 6.8 Default Values for Props

```jsx
// Method 1 — Default values in destructuring (recommended)
function Button({ label = 'Click Me', variant = 'primary', size = 'md' }) {
  return (
    <button className={`btn btn--${variant} btn--${size}`}>
      {label}
    </button>
  );
}

// Method 2 — defaultProps (legacy, works but less preferred now)
function Button({ label, variant, size }) {
  return <button className={`btn btn--${variant} btn--${size}`}>{label}</button>;
}

Button.defaultProps = {
  label: 'Click Me',
  variant: 'primary',
  size: 'md',
};
```

### 6.9 Renaming Props During Destructuring

```jsx
// Rename props to avoid conflicts with JavaScript reserved words or for clarity
function Component({ class: className, for: htmlFor, value: inputValue }) {
  return <input className={className} htmlFor={htmlFor} value={inputValue} />;
}
```

### 6.10 Rest Props (Forwarding Unknown Props)

```jsx
// Collect remaining props to forward to a DOM element
function Input({ label, className, ...rest }) {
  return (
    <div className="input-wrapper">
      <label>{label}</label>
      <input className={`input ${className}`} {...rest} />
    </div>
  );
}

// Usage — type, placeholder, onChange, etc. are forwarded automatically
<Input
  label="Email"
  className="custom"
  type="email"
  placeholder="you@example.com"
  onChange={handleChange}
  autoComplete="email"
/>
```

---

## 7. The `children` Prop

### What is `children`?

`children` is a **special built-in prop** that represents whatever JSX is placed **between the opening and closing tags** of a component.

```jsx
// The JSX between the tags becomes props.children
<Card>
  <h2>Hello</h2>
  <p>This is content inside the card.</p>
</Card>
```

Inside `Card`, you access it via `props.children` or by destructuring:

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

### Why `children` is Powerful

The `children` prop enables **layout components** — components that provide structure (padding, borders, backgrounds) but don't care about the specific content inside:

```jsx
// Layout components using children
function PageSection({ title, children }) {
  return (
    <section className="page-section">
      <h2 className="section-title">{title}</h2>
      <div className="section-content">
        {children}
      </div>
    </section>
  );
}

// Usage — PageSection wraps any content
<PageSection title="Our Team">
  <TeamMemberCard name="Ravi" />
  <TeamMemberCard name="Sneha" />
  <TeamMemberCard name="Arjun" />
</PageSection>
```

### Types of `children` Values

```jsx
// 1. String
<Label>Email Address</Label>         // children = "Email Address"

// 2. Single element
<Box><Button>Click</Button></Box>    // children = <Button> element

// 3. Multiple elements
<List>
  <Item>One</Item>
  <Item>Two</Item>
</List>                              // children = array of elements

// 4. Expression
<Display>{count * 2}</Display>       // children = computed value

// 5. Nothing (undefined)
<EmptyBox />                         // children = undefined
```

### Checking if `children` Exists

```jsx
import { Children } from 'react';

function Card({ children, title }) {
  return (
    <div className="card">
      {title && <h3 className="card__title">{title}</h3>}
      {Children.count(children) > 0
        ? <div className="card__body">{children}</div>
        : <p className="card__empty">Nothing to display.</p>
      }
    </div>
  );
}
```

---

## 8. Prop Types & Default Props

### Why Validate Props?

In a large team or codebase, components are used by many developers. PropTypes act as lightweight documentation and **runtime warnings** when wrong prop types are passed.

> For full type safety in production, use **TypeScript** instead of PropTypes.

### Installing PropTypes

```bash
npm install prop-types
```

### Using PropTypes

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email, role, isVerified }) {
  return (
    <div className="user-card">
      <h3>{name} {isVerified && '✅'}</h3>
      <p>{email}</p>
      <span>{role} | Age: {age}</span>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string.isRequired,
  role: PropTypes.oneOf(['admin', 'editor', 'viewer']),
  isVerified: PropTypes.bool,
};

UserCard.defaultProps = {
  role: 'viewer',
  isVerified: false,
};
```

### Common PropTypes

| PropType | Validates |
|----------|-----------|
| `PropTypes.string` | String |
| `PropTypes.number` | Number |
| `PropTypes.bool` | Boolean |
| `PropTypes.func` | Function |
| `PropTypes.array` | Array |
| `PropTypes.object` | Object |
| `PropTypes.node` | Anything renderable (string, number, element, array) |
| `PropTypes.element` | React element |
| `PropTypes.oneOf(['a','b'])` | One of specific values (enum) |
| `PropTypes.oneOfType([...])` | One of multiple types |
| `PropTypes.arrayOf(PropTypes.string)` | Array of strings |
| `PropTypes.shape({ name: PropTypes.string })` | Object with specific shape |
| `PropTypes.isRequired` | Append to any type to make required |

### TypeScript Props (Production Standard)

In modern React with TypeScript, you define props using interfaces or type aliases:

```tsx
// TypeScript — type-safe props
interface UserCardProps {
  name: string;
  age: number;
  email: string;
  role?: 'admin' | 'editor' | 'viewer';  // ? means optional
  isVerified?: boolean;
  onProfileClick: (id: string) => void;
}

function UserCard({
  name,
  age,
  email,
  role = 'viewer',
  isVerified = false,
  onProfileClick
}: UserCardProps) {
  return (
    <div className="user-card" onClick={() => onProfileClick(email)}>
      <h3>{name} {isVerified && '✅'}</h3>
      <p>{email}</p>
      <span>{role} | Age: {age}</span>
    </div>
  );
}
```

---

## 9. Component Composition

### What is Composition?

Composition is the practice of **building complex UIs by combining smaller, simpler components**. It is the primary way to reuse logic and UI in React — preferred over inheritance.

> React's documentation explicitly states: "use composition instead of inheritance to reuse code between components."

### Pattern 1: Simple Composition

```jsx
function Avatar({ src, alt }) {
  return <img className="avatar" src={src} alt={alt} />;
}

function UserInfo({ name, title }) {
  return (
    <div className="user-info">
      <strong>{name}</strong>
      <span>{title}</span>
    </div>
  );
}

// Compose them together
function UserHeader({ user }) {
  return (
    <div className="user-header">
      <Avatar src={user.avatarUrl} alt={user.name} />
      <UserInfo name={user.name} title={user.title} />
    </div>
  );
}
```

### Pattern 2: Containment (Slot Pattern)

Components that act as generic containers using `children` or named slot props:

```jsx
// Generic Dialog with named slots
function Dialog({ title, children, footer }) {
  return (
    <div className="dialog-overlay">
      <div className="dialog">
        <div className="dialog__header">
          <h2>{title}</h2>
        </div>
        <div className="dialog__body">
          {children}
        </div>
        {footer && (
          <div className="dialog__footer">
            {footer}
          </div>
        )}
      </div>
    </div>
  );
}

// Usage with named slots
<Dialog
  title="Confirm Delete"
  footer={
    <>
      <button onClick={cancel}>Cancel</button>
      <button onClick={confirm} className="btn-danger">Delete</button>
    </>
  }
>
  <p>Are you sure you want to delete this item? This cannot be undone.</p>
</Dialog>
```

### Pattern 3: Specialization

Creating specific versions of a generic component:

```jsx
// Generic Button
function Button({ children, variant = 'primary', size = 'md', ...rest }) {
  return (
    <button className={`btn btn--${variant} btn--${size}`} {...rest}>
      {children}
    </button>
  );
}

// Specialized versions
function PrimaryButton(props) {
  return <Button variant="primary" {...props} />;
}

function DangerButton(props) {
  return <Button variant="danger" {...props} />;
}

function IconButton({ icon, label, ...props }) {
  return (
    <Button size="sm" aria-label={label} {...props}>
      {icon}
    </Button>
  );
}

// Usage
<PrimaryButton onClick={save}>Save Changes</PrimaryButton>
<DangerButton onClick={deleteItem}>Delete</DangerButton>
<IconButton icon="🗑️" label="Delete item" onClick={deleteItem} />
```

---

## 10. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Static Component

```jsx
// File: components/WelcomeBanner.jsx

function WelcomeBanner() {
  return (
    <div className="welcome-banner">
      <h1>Welcome to ReactShop 🛍️</h1>
      <p>Discover amazing products at unbeatable prices.</p>
      <a href="/products" className="btn btn-primary">
        Shop Now
      </a>
    </div>
  );
}

export default WelcomeBanner;
```

---

### Example 2 — Beginner: Component with Props

```jsx
// File: components/Badge.jsx

function Badge({ text, color = 'blue' }) {
  const colorMap = {
    blue:   { background: '#dbeafe', color: '#1d4ed8' },
    green:  { background: '#dcfce7', color: '#15803d' },
    red:    { background: '#fee2e2', color: '#b91c1c' },
    yellow: { background: '#fef9c3', color: '#a16207' },
  };

  const style = colorMap[color] || colorMap.blue;

  return (
    <span
      style={{
        ...style,
        padding: '2px 10px',
        borderRadius: '9999px',
        fontSize: '12px',
        fontWeight: 600,
      }}
    >
      {text}
    </span>
  );
}

export default Badge;

// Usage:
// <Badge text="New" color="green" />
// <Badge text="Sale" color="red" />
// <Badge text="Popular" color="yellow" />
```

---

### Example 3 — Intermediate: Product Card with Callback

```jsx
// File: components/ProductCard/ProductCard.jsx

import Badge from '../Badge/Badge';

function ProductCard({ product, onAddToCart, onWishlist }) {
  const {
    id,
    name,
    price,
    originalPrice,
    image,
    badge,
    rating,
    reviewCount,
  } = product;

  const discount = originalPrice
    ? Math.round(((originalPrice - price) / originalPrice) * 100)
    : null;

  return (
    <article className="product-card">
      <div className="product-card__image-wrapper">
        <img src={image} alt={name} className="product-card__image" loading="lazy" />
        {badge && (
          <div className="product-card__badge">
            <Badge text={badge.text} color={badge.color} />
          </div>
        )}
        <button
          className="product-card__wishlist-btn"
          onClick={() => onWishlist(id)}
          aria-label={`Add ${name} to wishlist`}
        >
          ♡
        </button>
      </div>

      <div className="product-card__body">
        <h3 className="product-card__name">{name}</h3>

        <div className="product-card__rating">
          {'★'.repeat(Math.round(rating))}{'☆'.repeat(5 - Math.round(rating))}
          <span className="product-card__review-count">({reviewCount})</span>
        </div>

        <div className="product-card__pricing">
          <span className="product-card__price">₹{price.toLocaleString()}</span>
          {originalPrice && (
            <>
              <span className="product-card__original-price">
                ₹{originalPrice.toLocaleString()}
              </span>
              <span className="product-card__discount">{discount}% off</span>
            </>
          )}
        </div>

        <button
          className="btn btn--primary product-card__add-btn"
          onClick={() => onAddToCart(id)}
        >
          Add to Cart
        </button>
      </div>
    </article>
  );
}

export default ProductCard;
```

---

### Example 4 — Advanced: Compound Component Pattern

```jsx
// File: components/Accordion/Accordion.jsx
// An accordion UI built with component composition

import { useState, createContext, useContext } from 'react';

// Context to share state between Accordion sub-components
const AccordionContext = createContext(null);

function Accordion({ children, defaultOpen = null }) {
  const [openIndex, setOpenIndex] = useState(defaultOpen);

  function toggle(index) {
    setOpenIndex(prev => (prev === index ? null : index));
  }

  return (
    <AccordionContext.Provider value={{ openIndex, toggle }}>
      <div className="accordion">
        {children}
      </div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ children, index }) {
  return (
    <div className="accordion__item">
      {/* Pass the index to children */}
      {React.Children.map(children, child =>
        React.cloneElement(child, { index })
      )}
    </div>
  );
}

function AccordionHeader({ children, index }) {
  const { openIndex, toggle } = useContext(AccordionContext);
  const isOpen = openIndex === index;

  return (
    <button
      className={`accordion__header ${isOpen ? 'accordion__header--active' : ''}`}
      onClick={() => toggle(index)}
      aria-expanded={isOpen}
    >
      {children}
      <span className="accordion__icon">{isOpen ? '▲' : '▼'}</span>
    </button>
  );
}

function AccordionBody({ children, index }) {
  const { openIndex } = useContext(AccordionContext);
  const isOpen = openIndex === index;

  if (!isOpen) return null;

  return (
    <div className="accordion__body" role="region">
      {children}
    </div>
  );
}

// Attach sub-components as properties for clean API
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Body = AccordionBody;

export default Accordion;

// ─── Usage ───────────────────────────────────────────────
function FAQPage() {
  const faqs = [
    { q: 'What is React?', a: 'A JavaScript library for building UIs.' },
    { q: 'Is React free?', a: 'Yes, React is open-source and free to use.' },
    { q: 'What are hooks?', a: 'Functions that let you use state in function components.' },
  ];

  return (
    <Accordion defaultOpen={0}>
      {faqs.map((faq, i) => (
        <Accordion.Item key={i} index={i}>
          <Accordion.Header index={i}>{faq.q}</Accordion.Header>
          <Accordion.Body index={i}><p>{faq.a}</p></Accordion.Body>
        </Accordion.Item>
      ))}
    </Accordion>
  );
}
```

---

## 11. Real-World Use Cases

### 11.1 Navigation Bar

```jsx
// File: components/Navbar/Navbar.jsx

function NavLink({ href, children, isActive }) {
  return (
    <a
      href={href}
      className={`nav-link ${isActive ? 'nav-link--active' : ''}`}
      aria-current={isActive ? 'page' : undefined}
    >
      {children}
    </a>
  );
}

function Navbar({ currentPath, user }) {
  const links = [
    { href: '/', label: 'Home' },
    { href: '/products', label: 'Products' },
    { href: '/about', label: 'About' },
    { href: '/contact', label: 'Contact' },
  ];

  return (
    <header className="navbar">
      <a href="/" className="navbar__brand">MyBrand</a>

      <nav className="navbar__links" aria-label="Main navigation">
        {links.map(link => (
          <NavLink
            key={link.href}
            href={link.href}
            isActive={currentPath === link.href}
          >
            {link.label}
          </NavLink>
        ))}
      </nav>

      <div className="navbar__actions">
        {user
          ? <span className="navbar__user">Hi, {user.firstName}</span>
          : <a href="/login" className="btn btn--outline">Login</a>
        }
      </div>
    </header>
  );
}
```

### 11.2 Reusable Form Input

```jsx
// File: components/FormInput/FormInput.jsx

function FormInput({
  label,
  name,
  type = 'text',
  value,
  onChange,
  error,
  hint,
  required = false,
  ...rest
}) {
  const inputId = `input-${name}`;
  const errorId = `${inputId}-error`;

  return (
    <div className={`form-group ${error ? 'form-group--error' : ''}`}>
      <label htmlFor={inputId} className="form-label">
        {label}
        {required && <span className="form-label__required" aria-hidden="true"> *</span>}
      </label>

      <input
        id={inputId}
        name={name}
        type={type}
        value={value}
        onChange={onChange}
        required={required}
        aria-describedby={error ? errorId : undefined}
        aria-invalid={error ? 'true' : undefined}
        className="form-input"
        {...rest}
      />

      {hint && !error && (
        <p className="form-hint">{hint}</p>
      )}

      {error && (
        <p id={errorId} className="form-error" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}

export default FormInput;
```

---

## 12. Best Practices

### 12.1 Single Responsibility Principle

Each component should do **one thing** only.

```jsx
// ❌ Does too much — displays, fetches, and validates
function UserSection() { /* 200 lines of mixed logic */ }

// ✅ Each component has one job
function UserFetcher({ userId }) { /* only fetches data */ }
function UserDisplay({ user }) { /* only displays data */ }
function UserValidator({ data }) { /* only validates */ }
```

### 12.2 Keep Components Small

A component that requires more than 100–150 lines is likely doing too much and should be broken down.

### 12.3 Name Components Clearly

```jsx
// ❌ Unclear names
function Comp() { ... }
function Card2() { ... }
function MyThing() { ... }

// ✅ Self-documenting names
function InvoiceLineItem() { ... }
function UserAvatarWithBadge() { ... }
function CheckoutSummaryCard() { ... }
```

### 12.4 Start Component Names with a Capital Letter

```jsx
// ❌ React treats lowercase as a HTML tag — will NOT render your component
function myButton() { return <button>Click</button>; }
<myButton />   // React thinks this is a custom HTML element

// ✅ Capital letter — React knows it's a component
function MyButton() { return <button>Click</button>; }
<MyButton />
```

### 12.5 Keep Props Interface Small

```jsx
// ❌ Too many individual props
<UserCard
  firstName="Priya"
  lastName="Sharma"
  email="priya@example.com"
  phone="+91-9876543210"
  role="admin"
  department="Engineering"
  avatarUrl="..."
  joinDate="2023-01-15"
/>

// ✅ Group related data into an object
const user = { firstName: 'Priya', lastName: 'Sharma', email: '...', ... };
<UserCard user={user} />
```

### 12.6 Folder Structure for Components

```
src/
└── components/
    ├── common/              # Reused across entire app
    │   ├── Button/
    │   │   ├── Button.jsx
    │   │   ├── Button.test.jsx
    │   │   ├── Button.module.css
    │   │   └── index.js     # export { default } from './Button'
    │   ├── Badge/
    │   └── FormInput/
    ├── layout/              # Layout-level components
    │   ├── Navbar/
    │   ├── Footer/
    │   └── Sidebar/
    └── features/            # Feature-specific components
        ├── products/
        │   ├── ProductCard/
        │   ├── ProductGrid/
        │   └── ProductFilter/
        └── checkout/
            ├── CartItem/
            └── OrderSummary/
```

---

## 13. Common Mistakes

### Mistake 1: Mutating Props

```jsx
// ❌ Never mutate props
function Profile(props) {
  props.name = props.name.toUpperCase(); // BUG
  return <h1>{props.name}</h1>;
}

// ✅ Create a new local variable
function Profile({ name }) {
  const displayName = name.toUpperCase();
  return <h1>{displayName}</h1>;
}
```

### Mistake 2: Lowercase Component Name

```jsx
// ❌ React renders a DOM element named "header", not your component
function header() { return <div>...</div>; }
<header />

// ✅ Capital letter
function Header() { return <div>...</div>; }
<Header />
```

### Mistake 3: Not Using `key` When Composing Lists of Components

```jsx
// ❌ Missing key
{comments.map(c => <CommentCard comment={c} />)}

// ✅ With key
{comments.map(c => <CommentCard key={c.id} comment={c} />)}
```

### Mistake 4: Prop Drilling Too Deep

```jsx
// ❌ Passing user 4 levels deep — maintenance nightmare
<App user={user}>
  <Dashboard user={user}>
    <Sidebar user={user}>
      <UserMenu user={user} />
    </Sidebar>
  </Dashboard>
</App>

// ✅ Use React Context for deeply shared data (covered in Context module)
```

### Mistake 5: Defining Components Inside Other Components

```jsx
// ❌ Inner component re-created on every render — causes performance issues
function Parent() {
  function Child() {  // New function reference every render
    return <p>I am a child</p>;
  }
  return <Child />;
}

// ✅ Define components at module level
function Child() {
  return <p>I am a child</p>;
}

function Parent() {
  return <Child />;
}
```

### Mistake 6: Passing the Wrong Prop Type

```jsx
// ❌ Passing string "5" instead of number 5
<Rating stars="5" />

// Inside the component, "5" * 2 = 10 works, but "5" + 1 = "51" (string concat)
// This causes subtle bugs

// ✅ Use curly braces for non-string values
<Rating stars={5} />
```

---

## 14. Performance Considerations

### 14.1 React.memo — Prevent Unnecessary Re-renders

By default, when a parent re-renders, all its children re-render too — even if their props haven't changed. `React.memo` prevents this:

```jsx
// Without memo — re-renders whenever parent re-renders
function ProductCard({ product }) {
  return <div>{product.name}</div>;
}

// With memo — only re-renders when `product` prop actually changes
const ProductCard = React.memo(function ProductCard({ product }) {
  return <div>{product.name}</div>;
});
```

> Use `React.memo` when a component is expensive to render AND its parent re-renders frequently but the component's props don't change often.

### 14.2 Stable Callback Props with useCallback

When you pass a function as a prop, a new function reference is created each render. This causes `React.memo` children to re-render even if the logic hasn't changed.

```jsx
// ❌ New function reference every render — breaks React.memo
function Parent() {
  function handleClick() { console.log('clicked'); }
  return <MemoizedChild onClick={handleClick} />;
}

// ✅ Stable reference with useCallback
function Parent() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []); // [] means the function never changes

  return <MemoizedChild onClick={handleClick} />;
}
```

### 14.3 Lazy Loading Components

For large components (e.g., modals, charts, pages), load them only when needed:

```jsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<p>Loading chart...</p>}>
      <HeavyChart />
    </Suspense>
  );
}
```

---

## 15. Comparison: Class vs Function Components

| Feature | Class Component | Function Component |
|---------|-----------------|-------------------|
| Syntax | `class X extends Component` | `function X()` or `const X = () =>` |
| State | `this.state`, `this.setState()` | `useState()` hook |
| Lifecycle | `componentDidMount`, etc. | `useEffect()` hook |
| `this` keyword | Required (and confusing) | Not needed |
| Code length | More verbose | More concise |
| Performance | Slightly heavier | Lighter |
| Hooks support | ❌ Cannot use hooks | ✅ Full hooks support |
| New React features | ❌ No new features | ✅ All new features |
| Recommended today | ❌ Legacy | ✅ Standard |

```jsx
// Same component — class vs function

// Class Component (Legacy)
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// Function Component (Modern — preferred)
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

---

## 16. Interview Questions

### Q1: What is a React component? How is it different from a regular JavaScript function?

**Answer:** A React component is a JavaScript function that accepts a `props` object and returns JSX (React elements) describing UI. The key difference from a regular function is that React components integrate into React's rendering lifecycle — React calls them, manages their output, and updates the DOM based on what they return. Component names must start with a capital letter so React can distinguish them from regular HTML elements.

---

### Q2: What are props in React? Can you modify them inside a component?

**Answer:** Props are read-only inputs passed from a parent component to a child component. They are plain JavaScript objects. You **cannot and must not modify props inside a component** — doing so would violate React's unidirectional data flow and lead to unpredictable behavior. If you need to transform a prop value, create a new local variable based on it.

---

### Q3: What is the difference between props and state?

**Answer:** Props are **external** inputs passed **into** a component by its parent — the component doesn't control them. State is **internal** data managed **by** the component itself using `useState`. Props are read-only; state can be updated. When either changes, React re-renders the component.

---

### Q4: What is the `children` prop? Give a real-world use case.

**Answer:** `children` is a special React prop that represents the JSX content placed between a component's opening and closing tags. A real-world use case is a `Modal` or `Card` layout component — it provides the wrapper structure (overlay, container, close button) while `children` can be anything: forms, messages, confirmation dialogs. This makes the layout component infinitely reusable without needing to know about its content.

---

### Q5: What is component composition and why is it preferred over inheritance?

**Answer:** Composition means building complex UI by combining smaller, simpler components — usually via the `children` prop or prop-based slot patterns. React's team recommends it over inheritance because: (1) it's more flexible — you can compose any combination of components; (2) it avoids the fragile base class problem common in OOP inheritance; (3) it maps naturally to the way JSX is nested and rendered.

---

### Q6: What happens if you define a component inside another component?

**Answer:** Defining a component inside another component creates a **new function reference on every render**. React sees this as a completely new component type each render and will unmount the old one and mount a new one — destroying any internal state and causing performance issues. Always define components at the module (top) level.

---

### Q7: What is `React.memo` and when should you use it?

**Answer:** `React.memo` is a higher-order component that memoizes the rendered output of a function component. It prevents re-renders when the props haven't changed. Use it when: (1) the component renders frequently due to parent re-renders, (2) the component is expensive to render (complex calculations or large DOM trees), and (3) props are typically the same between renders. Don't overuse it — the memoization itself has a cost.

---

### Q8: How do you pass data from a child component to a parent component?

**Answer:** Since props flow only downward (parent → child), child-to-parent communication is achieved by passing a **callback function** as a prop from the parent to the child. When something happens in the child (a button click, form submission), it calls the callback with the relevant data. The parent's function receives the data and can update its state accordingly.

---

### Q9: What is the difference between a controlled and uncontrolled component?

**Answer:** A **controlled component** is one where React manages the form element's value through state — the input's value is always in sync with a state variable, and every change goes through `onChange → setState`. An **uncontrolled component** stores its value in the DOM itself and is accessed via a `ref`. Controlled components are the React-recommended approach for forms as they give you full control over the input data.

---

### Q10: What is "prop drilling" and what problems does it cause?

**Answer:** Prop drilling is the practice of passing props through multiple intermediate components that don't use the data themselves — just to get it to a deeply nested component that does. It causes: (1) tightly coupled components, (2) code that's hard to maintain (changing the prop name requires updating every level), and (3) components polluted with props they don't actually care about. Solutions include React Context, Zustand, or Redux for truly global data.

---

## 17. Practice Tasks

### Task 1 — Beginner: Build a Testimonial Card

Create a `TestimonialCard` component that accepts these props:
- `quote` (string) — the testimonial text
- `authorName` (string)
- `authorTitle` (string, e.g., "CEO at TechCorp")
- `avatarUrl` (string)
- `rating` (number, 1–5)

**Requirements:**
- Display stars based on `rating` (filled: ★, empty: ☆)
- Wrap the component in a styled card container
- Add `aria-label` to the rating display for accessibility
- Export with a default export
- Create a parent `TestimonialList` that renders 3 testimonials from a hardcoded array

---

### Task 2 — Intermediate: Build a Reusable Stats Widget

Build a `StatsGrid` component that displays a row of metric cards.

**Requirements:**
- Accept a `stats` array: `[{ label, value, icon, trend: 'up'|'down'|null, change: string }]`
- Each card shows the label, value, icon, and a colored trend arrow with change value
- ▲ green for 'up', ▼ red for 'down', nothing if null
- If `stats` is empty or undefined, show "No stats available"
- Use `React.memo` on the individual stat card
- Add PropTypes validation for the `stats` prop

---

### Task 3 — Advanced: Compound Tab Component

Build a fully composable `Tabs` component system using the compound component pattern.

**Requirements:**
- Create: `Tabs`, `Tabs.List`, `Tabs.Tab`, `Tabs.Panels`, `Tabs.Panel`
- `Tabs` manages the active tab index via state
- `Tabs.Tab` is a button that activates its corresponding panel when clicked
- `Tabs.Panel` renders only when its index matches the active index
- Active tab should have a visual indicator (border-bottom or background)
- Add keyboard support: ArrowLeft/ArrowRight to navigate between tabs
- Add `aria-selected`, `role="tab"`, `role="tabpanel"`, `aria-controls` for full accessibility
- Allow a `defaultTab` prop on `Tabs`

**Expected usage API:**
```jsx
<Tabs defaultTab={0}>
  <Tabs.List>
    <Tabs.Tab index={0}>Overview</Tabs.Tab>
    <Tabs.Tab index={1}>Details</Tabs.Tab>
    <Tabs.Tab index={2}>Reviews</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel index={0}><OverviewContent /></Tabs.Panel>
    <Tabs.Panel index={1}><DetailsContent /></Tabs.Panel>
    <Tabs.Panel index={2}><ReviewsContent /></Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

---

## 18. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| Component | A JS function that takes props and returns JSX |
| Function component | Modern standard — use for all new code |
| Props | Read-only inputs from parent to child — never mutate |
| Props object | All JSX attributes become one plain JS object |
| Default props | Use destructuring defaults: `{ size = 'md' }` |
| `children` prop | Special prop for JSX content between opening/closing tags |
| Callback props | How child communicates to parent — pass function as prop |
| Composition | Build complex UI from small components — preferred over inheritance |
| PropTypes | Runtime type checking in dev — use TypeScript in production |
| `React.memo` | Memoize component to skip re-renders when props unchanged |
| Naming | Always start with a capital letter |
| Folder structure | One component per folder with its own CSS and test file |

### The Golden Rules of Components

1. **Props flow down, events bubble up** — data goes parent → child, callbacks go child → parent
2. **Never mutate props** — they are read-only
3. **One component, one responsibility** — if it's doing too much, split it
4. **Define components at module level** — never inside another component's render
5. **Name clearly and consistently** — `UserProfileCard` beats `Card2`
6. **Prefer composition** — build complex UI by combining simple pieces

---

> **Next Topic:** `04-state-and-useState.md` — Managing component-level data that changes over time, triggering re-renders, and mastering the useState hook.
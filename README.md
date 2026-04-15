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
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


# 04 — State & useState in React (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** State & the useState Hook
> **Level:** Beginner → Advanced
> **Prerequisites:** Components & Props (Module 03), JSX (Module 02), Basic JavaScript (arrays, objects, spread operator)

---

## Table of Contents

1. [What is State?](#1-what-is-state)
2. [Why State Exists](#2-why-state-exists)
3. [How useState Works Internally](#3-how-usestate-works-internally)
4. [useState — Complete Syntax Guide](#4-usestate--complete-syntax-guide)
5. [Updating State Correctly](#5-updating-state-correctly)
6. [State with Objects](#6-state-with-objects)
7. [State with Arrays](#7-state-with-arrays)
8. [Multiple State Variables](#8-multiple-state-variables)
9. [Derived State](#9-derived-state)
10. [Lifting State Up](#10-lifting-state-up)
11. [Code Examples (Beginner → Advanced)](#11-code-examples-beginner--advanced)
12. [Real-World Use Cases](#12-real-world-use-cases)
13. [Best Practices](#13-best-practices)
14. [Common Mistakes](#14-common-mistakes)
15. [Performance Considerations](#15-performance-considerations)
16. [State vs Props — Full Comparison](#16-state-vs-props--full-comparison)
17. [Interview Questions](#17-interview-questions)
18. [Practice Tasks](#18-practice-tasks)
19. [Summary](#19-summary)

---

## 1. What is State?

### Simple Explanation

**State** is data that a component remembers and can change over time. When state changes, React automatically re-renders the component to reflect the new data on screen.

Think of state like the **memory** of a component. A counter that increases when you click a button, a form input that stores what you're typing, a toggle that switches between light and dark mode — all of these are powered by state.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // count is state

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

Every time the button is clicked, `count` changes, and React re-renders the component with the new value.

### Technical Explanation

State is a **reactive data store** local to a component instance. "Reactive" means that React automatically tracks it — when it changes, React schedules a re-render of that component (and its children). State is stored by React itself, not inside the regular function execution stack, which is why it persists across re-renders.

---

## 2. Why State Exists

### The Problem: Regular Variables Don't Trigger Re-renders

If you try to use a regular JavaScript variable to track changing data, it won't work:

```jsx
// ❌ This does NOT work
function BrokenCounter() {
  let count = 0; // regular JS variable

  function handleClick() {
    count = count + 1;        // value changes...
    console.log(count);       // logs correctly (1, 2, 3...)
    // BUT React has no idea this changed — no re-render happens
  }

  return (
    <div>
      <p>{count}</p>           {/* Always shows 0 */}
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

**Two problems here:**
1. The variable resets to `0` every time the component function runs (every render)
2. React doesn't know the variable changed, so it never re-renders

### The Solution: useState

`useState` solves both problems:
1. React stores the value **outside** the function, so it persists between renders
2. Calling the setter function (`setCount`) **signals React** to re-render

```jsx
// ✅ This works
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // tells React: "value changed, please re-render"
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

---

## 3. How useState Works Internally

### The Hook Call

When you call `useState(initialValue)`, React does the following:

```
First render:
1. React sees useState(0) called for the first time in this component
2. React creates a "state slot" in its internal memory for this component instance
3. React stores 0 in that slot
4. React returns [0, setterFunction] → you destructure as [count, setCount]

Subsequent renders (after setCount is called):
1. React re-renders the component (runs the function again)
2. React sees useState(0) again — but now ignores the initial value (0)
3. React reads the current value from the "state slot" in memory (e.g., 3)
4. React returns [3, setterFunction] → you destructure as [count, setCount]
```

### The Order Rule — Why Hooks Must Not Be Called Conditionally

React identifies each `useState` call by its **order** in the component. It doesn't use the variable name. This is why hooks must always be called in the same order on every render.

```jsx
// React's internal state array (conceptual)
// Component: ShoppingCart
// State slot 0: cartItems → []
// State slot 1: isOpen → false
// State slot 2: couponCode → ''

function ShoppingCart() {
  const [cartItems, setCartItems] = useState([]);   // slot 0
  const [isOpen, setIsOpen] = useState(false);       // slot 1
  const [couponCode, setCouponCode] = useState('');  // slot 2
  // ...
}
```

If the order ever changes (e.g., due to a conditional hook call), React reads the wrong slot and everything breaks.

```jsx
// ❌ NEVER do this — conditional hook call
function BadComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [profile, setProfile] = useState(null); // sometimes slot 0, sometimes missing
  }
  const [count, setCount] = useState(0); // sometimes slot 0, sometimes slot 1 — BROKEN
}

// ✅ Always call hooks at the top level, unconditionally
function GoodComponent({ isLoggedIn }) {
  const [profile, setProfile] = useState(null); // always slot 0
  const [count, setCount] = useState(0);         // always slot 1
  // Use isLoggedIn inside the render logic, not to gate the hook call
}
```

### Batching State Updates (React 18+)

In React 18, multiple `setState` calls inside a single event handler are **batched** into a single re-render for performance:

```jsx
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  function handleReset() {
    setName('');    // does NOT trigger a re-render immediately
    setEmail('');   // does NOT trigger a re-render immediately
    // React batches both and triggers ONE re-render at the end
  }
}
```

Before React 18, batching only happened inside React event handlers. Now it happens everywhere, including `setTimeout`, `Promise` callbacks, and native event listeners.

---

## 4. useState — Complete Syntax Guide

### 4.1 Basic Syntax

```jsx
const [stateValue, setterFunction] = useState(initialValue);
```

- `stateValue` — the current value of state
- `setterFunction` — the function you call to update state
- `initialValue` — the starting value (only used on the first render)

### 4.2 Naming Convention

By convention, the setter is named `set` + the state variable name (camelCase):

```jsx
const [count, setCount] = useState(0);
const [isOpen, setIsOpen] = useState(false);
const [userName, setUserName] = useState('');
const [cartItems, setCartItems] = useState([]);
const [userProfile, setUserProfile] = useState(null);
```

### 4.3 Initial Values — All Types

```jsx
// Number
const [count, setCount] = useState(0);
const [price, setPrice] = useState(99.99);

// String
const [name, setName] = useState('');
const [status, setStatus] = useState('idle'); // 'idle' | 'loading' | 'success' | 'error'

// Boolean
const [isVisible, setIsVisible] = useState(false);
const [isDarkMode, setIsDarkMode] = useState(true);

// Null (before data is loaded)
const [user, setUser] = useState(null);
const [selectedItem, setSelectedItem] = useState(null);

// Array
const [items, setItems] = useState([]);
const [tags, setTags] = useState(['react', 'javascript']);

// Object
const [formData, setFormData] = useState({ name: '', email: '', password: '' });

// Undefined (rare)
const [value, setValue] = useState(undefined);
```

### 4.4 Lazy Initial State

If computing the initial state is **expensive** (e.g., reading from localStorage, complex calculation), pass a **function** instead of a value. This function runs only **once** on mount:

```jsx
// ❌ This runs on EVERY render (wasteful)
const [theme, setTheme] = useState(localStorage.getItem('theme') || 'light');

// ✅ This runs ONLY on first render (efficient)
const [theme, setTheme] = useState(() => {
  return localStorage.getItem('theme') || 'light';
});

// Another example — expensive computation
const [data, setData] = useState(() => {
  return processLargeDataset(rawData); // runs only once
});
```

---

## 5. Updating State Correctly

### 5.1 Direct Update

When the new value doesn't depend on the previous value:

```jsx
// Set to a completely new value
setCount(10);
setName('Priya');
setIsOpen(true);
setUser(null);
```

### 5.2 Functional Update (Updater Function) — CRITICAL

When the new value **depends on the previous value**, always use the **functional form** of the setter. This ensures you're working with the latest state, even inside stale closures:

```jsx
// ❌ Potentially stale — uses the captured value of count
setCount(count + 1);

// ✅ Always correct — receives the guaranteed latest value as argument
setCount(prevCount => prevCount + 1);
```

**Why does this matter?** Consider this example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleTripleClick() {
    // ❌ All three calls see count = 0 (stale closure)
    setCount(count + 1); // sets to 1
    setCount(count + 1); // sets to 1 (not 2!)
    setCount(count + 1); // sets to 1 (not 3!)
    // Result: count is 1, not 3

    // ✅ Each call receives the result of the previous
    setCount(prev => prev + 1); // 0 → 1
    setCount(prev => prev + 1); // 1 → 2
    setCount(prev => prev + 1); // 2 → 3
    // Result: count is 3 ✓
  }

  return <button onClick={handleTripleClick}>{count}</button>;
}
```

### 5.3 Toggle Pattern

```jsx
const [isOpen, setIsOpen] = useState(false);

// ✅ Toggle using functional update
function toggle() {
  setIsOpen(prev => !prev);
}

// Usage
<button onClick={toggle}>{isOpen ? 'Close' : 'Open'}</button>
```

### 5.4 Reset Pattern

```jsx
const INITIAL_FORM = { name: '', email: '', message: '' };

const [form, setForm] = useState(INITIAL_FORM);

function resetForm() {
  setForm(INITIAL_FORM);
}
```

---

## 6. State with Objects

### The Golden Rule: Never Mutate State Directly

React compares state using **reference equality** (`===`). If you mutate an object directly without creating a new one, React won't detect any change and won't re-render.

```jsx
const [user, setUser] = useState({ name: 'Rahul', age: 25, city: 'Mumbai' });

// ❌ WRONG — mutating state directly (reference doesn't change)
function handleBirthday() {
  user.age = user.age + 1;  // Mutating the existing object
  setUser(user);             // React sees same reference → NO re-render
}

// ✅ CORRECT — create a new object with spread operator
function handleBirthday() {
  setUser(prevUser => ({
    ...prevUser,       // copy all existing fields
    age: prevUser.age + 1  // override only what changed
  }));
}
```

### Updating Nested Objects

```jsx
const [profile, setProfile] = useState({
  name: 'Sneha',
  address: {
    city: 'Pune',
    pincode: '411001'
  },
  preferences: {
    theme: 'dark',
    notifications: true
  }
});

// ✅ Update nested field — spread at every level
function updateCity(newCity) {
  setProfile(prev => ({
    ...prev,
    address: {
      ...prev.address,
      city: newCity
    }
  }));
}

function toggleNotifications() {
  setProfile(prev => ({
    ...prev,
    preferences: {
      ...prev.preferences,
      notifications: !prev.preferences.notifications
    }
  }));
}
```

> **Tip:** For deeply nested state, consider using a library like **Immer** (used inside Redux Toolkit) which lets you write "mutating" code that is actually safe:

```jsx
import { useImmer } from 'use-immer'; // npm install use-immer

const [profile, updateProfile] = useImmer({ name: 'Sneha', address: { city: 'Pune' } });

// Looks like mutation, but Immer creates a new object safely
function updateCity(newCity) {
  updateProfile(draft => {
    draft.address.city = newCity; // ✅ Safe with Immer
  });
}
```

---

## 7. State with Arrays

Arrays in state also follow the immutability rule — never use mutating methods (`push`, `pop`, `splice`, `sort` in place). Always return a **new array**.

### 7.1 Adding an Item

```jsx
const [items, setItems] = useState(['Apple', 'Banana']);

// ✅ Add to end
setItems(prev => [...prev, 'Mango']);

// ✅ Add to beginning
setItems(prev => ['Mango', ...prev]);

// ✅ Add at specific index
function insertAt(index, newItem) {
  setItems(prev => [
    ...prev.slice(0, index),
    newItem,
    ...prev.slice(index)
  ]);
}
```

### 7.2 Removing an Item

```jsx
const [items, setItems] = useState([
  { id: 1, name: 'Apple' },
  { id: 2, name: 'Banana' },
  { id: 3, name: 'Mango' },
]);

// ✅ Remove by id using filter
function removeItem(id) {
  setItems(prev => prev.filter(item => item.id !== id));
}
```

### 7.3 Updating an Item

```jsx
// ✅ Update a specific item by id using map
function updateItemName(id, newName) {
  setItems(prev =>
    prev.map(item =>
      item.id === id
        ? { ...item, name: newName }  // new object for the updated item
        : item                         // same reference for unchanged items
    )
  );
}
```

### 7.4 Sorting an Array

```jsx
// ❌ .sort() mutates the original array
setItems(prev => {
  prev.sort((a, b) => a.name.localeCompare(b.name)); // mutates prev!
  return prev;
});

// ✅ Spread first to create a new array, then sort
setItems(prev => [...prev].sort((a, b) => a.name.localeCompare(b.name)));
```

### Complete Array State Example

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build a project', completed: false },
  ]);

  function addTodo(text) {
    const newTodo = { id: Date.now(), text, completed: false };
    setTodos(prev => [...prev, newTodo]);
  }

  function toggleTodo(id) {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }

  function deleteTodo(id) {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }

  return (
    <div>
      <AddTodoForm onAdd={addTodo} />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 8. Multiple State Variables

### When to Use Multiple vs One Object

```jsx
// ✅ Use separate state variables when values change independently
const [username, setUsername] = useState('');
const [email, setEmail] = useState('');
const [isSubmitting, setIsSubmitting] = useState(false);
const [errorMessage, setErrorMessage] = useState(null);

// ✅ Use one object when values are closely related and change together
const [formData, setFormData] = useState({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
});
```

### Handling Form with Object State

```jsx
function RegistrationForm() {
  const [form, setForm] = useState({
    firstName: '',
    lastName: '',
    email: '',
    password: '',
  });

  // Generic handler using input's name attribute
  function handleChange(e) {
    const { name, value } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: value  // computed property key
    }));
  }

  return (
    <form>
      <input name="firstName" value={form.firstName} onChange={handleChange} placeholder="First Name" />
      <input name="lastName"  value={form.lastName}  onChange={handleChange} placeholder="Last Name" />
      <input name="email"     value={form.email}     onChange={handleChange} placeholder="Email" type="email" />
      <input name="password"  value={form.password}  onChange={handleChange} placeholder="Password" type="password" />
    </form>
  );
}
```

---

## 9. Derived State

**Derived state** is data that can be computed from existing state or props. You should **not** store it in state — compute it on every render instead.

```jsx
// ❌ Bad — storing derived data as state
function CartSummary() {
  const [items, setItems] = useState([...]);
  const [total, setTotal] = useState(0);  // WRONG — total is derived from items

  function addItem(item) {
    const newItems = [...items, item];
    setItems(newItems);
    setTotal(newItems.reduce((sum, i) => sum + i.price, 0)); // easy to get out of sync
  }
}

// ✅ Good — compute total during render
function CartSummary() {
  const [items, setItems] = useState([]);

  // Derived — computed fresh every render, always in sync
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const itemCount = items.length;
  const hasItems = items.length > 0;

  return (
    <div>
      <p>{itemCount} items — Total: ₹{total}</p>
      {hasItems && <button>Checkout</button>}
    </div>
  );
}
```

### The Rule

> If a value can be computed from state or props, **don't put it in state**. Compute it during rendering.

Storing derived state creates a **synchronization problem** — two sources of truth that can get out of sync.

---

## 10. Lifting State Up

### The Problem: Shared State Between Siblings

When two sibling components need to share or react to the same data, neither should own the state. Instead, **lift the state up** to the closest common ancestor.

```jsx
// ❌ State in each child — they can't talk to each other
function TemperatureConverter() {
  return (
    <>
      <CelsiusInput />    {/* has its own state */}
      <FahrenheitInput /> {/* has its own state — out of sync! */}
    </>
  );
}

// ✅ State lifted to parent — parent is single source of truth
function TemperatureConverter() {
  const [celsius, setCelsius] = useState(0);

  // Derived state — computed, not stored
  const fahrenheit = (celsius * 9/5) + 32;

  return (
    <>
      <CelsiusInput
        value={celsius}
        onChange={setCelsius}
      />
      <FahrenheitInput
        value={fahrenheit}
        onChange={f => setCelsius((f - 32) * 5/9)}
      />
    </>
  );
}

function CelsiusInput({ value, onChange }) {
  return (
    <input
      type="number"
      value={value}
      onChange={e => onChange(Number(e.target.value))}
      placeholder="Celsius"
    />
  );
}

function FahrenheitInput({ value, onChange }) {
  return (
    <input
      type="number"
      value={value}
      onChange={e => onChange(Number(e.target.value))}
      placeholder="Fahrenheit"
    />
  );
}
```

### When to Lift State

- Two or more components need to reflect the same changing data
- A parent needs to know about changes in a child
- Multiple siblings need to stay in sync

---

## 11. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Like Button

```jsx
// File: components/LikeButton.jsx

import { useState } from 'react';

function LikeButton({ initialCount = 0 }) {
  const [likeCount, setLikeCount] = useState(initialCount);
  const [isLiked, setIsLiked] = useState(false);

  function handleLike() {
    if (isLiked) {
      setLikeCount(prev => prev - 1);
    } else {
      setLikeCount(prev => prev + 1);
    }
    setIsLiked(prev => !prev);
  }

  return (
    <button
      onClick={handleLike}
      aria-pressed={isLiked}
      aria-label={isLiked ? 'Unlike this post' : 'Like this post'}
      style={{
        background: isLiked ? '#ef4444' : 'transparent',
        color: isLiked ? '#fff' : '#6b7280',
        border: '1px solid #e5e7eb',
        borderRadius: '8px',
        padding: '6px 14px',
        cursor: 'pointer',
        display: 'flex',
        alignItems: 'center',
        gap: '6px',
      }}
    >
      {isLiked ? '❤️' : '🤍'} {likeCount}
    </button>
  );
}

export default LikeButton;
```

---

### Example 2 — Intermediate: Multi-Step Form

```jsx
// File: components/MultiStepForm/MultiStepForm.jsx

import { useState } from 'react';

const STEPS = ['Personal Info', 'Contact Details', 'Confirmation'];

function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    city: '',
  });

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  }

  function goNext() {
    setCurrentStep(prev => Math.min(prev + 1, STEPS.length - 1));
  }

  function goBack() {
    setCurrentStep(prev => Math.max(prev - 1, 0));
  }

  function handleSubmit() {
    console.log('Form submitted:', formData);
    alert('Registration complete!');
  }

  return (
    <div className="multi-step-form">
      {/* Progress Bar */}
      <div className="step-indicator">
        {STEPS.map((step, i) => (
          <div
            key={step}
            className={`step ${i <= currentStep ? 'step--active' : ''}`}
          >
            <span className="step__number">{i + 1}</span>
            <span className="step__label">{step}</span>
          </div>
        ))}
      </div>

      {/* Step Content */}
      <div className="step-content">
        {currentStep === 0 && (
          <div>
            <h2>Personal Information</h2>
            <input name="firstName" value={formData.firstName} onChange={handleChange} placeholder="First Name" />
            <input name="lastName"  value={formData.lastName}  onChange={handleChange} placeholder="Last Name" />
          </div>
        )}

        {currentStep === 1 && (
          <div>
            <h2>Contact Details</h2>
            <input name="email" type="email" value={formData.email} onChange={handleChange} placeholder="Email" />
            <input name="phone" type="tel"   value={formData.phone} onChange={handleChange} placeholder="Phone" />
            <input name="city"               value={formData.city}  onChange={handleChange} placeholder="City" />
          </div>
        )}

        {currentStep === 2 && (
          <div>
            <h2>Confirm Your Details</h2>
            <p><strong>Name:</strong> {formData.firstName} {formData.lastName}</p>
            <p><strong>Email:</strong> {formData.email}</p>
            <p><strong>Phone:</strong> {formData.phone}</p>
            <p><strong>City:</strong> {formData.city}</p>
          </div>
        )}
      </div>

      {/* Navigation */}
      <div className="step-navigation">
        {currentStep > 0 && (
          <button onClick={goBack} className="btn btn--outline">← Back</button>
        )}
        {currentStep < STEPS.length - 1 ? (
          <button onClick={goNext} className="btn btn--primary">Next →</button>
        ) : (
          <button onClick={handleSubmit} className="btn btn--success">Submit ✓</button>
        )}
      </div>
    </div>
  );
}

export default MultiStepForm;
```

---

### Example 3 — Advanced: Shopping Cart with Full State Management

```jsx
// File: components/ShoppingCart/ShoppingCart.jsx

import { useState, useMemo } from 'react';

const SAMPLE_PRODUCTS = [
  { id: 1, name: 'Wireless Headphones', price: 2999, image: '🎧' },
  { id: 2, name: 'Mechanical Keyboard', price: 4500, image: '⌨️' },
  { id: 3, name: 'USB-C Hub',           price: 1299, image: '🔌' },
  { id: 4, name: 'Webcam HD',           price: 3499, image: '📷' },
];

function ShoppingCart() {
  const [cartItems, setCartItems] = useState([]);   // { id, name, price, image, quantity }
  const [isCartOpen, setIsCartOpen] = useState(false);
  const [coupon, setCoupon] = useState('');
  const [discount, setDiscount] = useState(0);

  // Derived state — computed from cartItems
  const totalItems = cartItems.reduce((sum, item) => sum + item.quantity, 0);
  const subtotal = cartItems.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const discountAmount = (subtotal * discount) / 100;
  const total = subtotal - discountAmount;

  function addToCart(product) {
    setCartItems(prev => {
      const exists = prev.find(item => item.id === product.id);
      if (exists) {
        // Increment quantity
        return prev.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      // Add new item
      return [...prev, { ...product, quantity: 1 }];
    });
  }

  function removeFromCart(id) {
    setCartItems(prev => prev.filter(item => item.id !== id));
  }

  function updateQuantity(id, delta) {
    setCartItems(prev =>
      prev
        .map(item =>
          item.id === id
            ? { ...item, quantity: item.quantity + delta }
            : item
        )
        .filter(item => item.quantity > 0) // auto-remove if quantity reaches 0
    );
  }

  function applyCoupon() {
    const coupons = { SAVE10: 10, SAVE20: 20, REACT50: 50 };
    const value = coupons[coupon.toUpperCase()];
    if (value) {
      setDiscount(value);
      alert(`Coupon applied! ${value}% off.`);
    } else {
      alert('Invalid coupon code.');
    }
  }

  function clearCart() {
    setCartItems([]);
    setDiscount(0);
    setCoupon('');
  }

  return (
    <div className="shop-layout">
      {/* Product Grid */}
      <section className="product-grid">
        <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
          <h1>Products</h1>
          <button
            className="cart-toggle-btn"
            onClick={() => setIsCartOpen(prev => !prev)}
          >
            🛒 Cart ({totalItems})
          </button>
        </div>

        <div className="products">
          {SAMPLE_PRODUCTS.map(product => (
            <div key={product.id} className="product-card">
              <div className="product-card__emoji">{product.image}</div>
              <h3>{product.name}</h3>
              <p>₹{product.price.toLocaleString()}</p>
              <button
                className="btn btn--primary"
                onClick={() => addToCart(product)}
              >
                Add to Cart
              </button>
            </div>
          ))}
        </div>
      </section>

      {/* Cart Drawer */}
      {isCartOpen && (
        <aside className="cart-drawer">
          <div className="cart-drawer__header">
            <h2>Your Cart ({totalItems} items)</h2>
            <button onClick={() => setIsCartOpen(false)} aria-label="Close cart">✕</button>
          </div>

          {cartItems.length === 0 ? (
            <p className="cart-empty">Your cart is empty.</p>
          ) : (
            <>
              <ul className="cart-items">
                {cartItems.map(item => (
                  <li key={item.id} className="cart-item">
                    <span className="cart-item__emoji">{item.image}</span>
                    <div className="cart-item__info">
                      <strong>{item.name}</strong>
                      <span>₹{item.price.toLocaleString()}</span>
                    </div>
                    <div className="cart-item__quantity">
                      <button onClick={() => updateQuantity(item.id, -1)}>−</button>
                      <span>{item.quantity}</span>
                      <button onClick={() => updateQuantity(item.id, +1)}>+</button>
                    </div>
                    <button
                      className="cart-item__remove"
                      onClick={() => removeFromCart(item.id)}
                      aria-label={`Remove ${item.name}`}
                    >
                      🗑️
                    </button>
                  </li>
                ))}
              </ul>

              <div className="cart-coupon">
                <input
                  type="text"
                  value={coupon}
                  onChange={e => setCoupon(e.target.value)}
                  placeholder="Enter coupon code"
                />
                <button onClick={applyCoupon}>Apply</button>
              </div>

              <div className="cart-summary">
                <div className="cart-summary__row">
                  <span>Subtotal</span>
                  <span>₹{subtotal.toLocaleString()}</span>
                </div>
                {discount > 0 && (
                  <div className="cart-summary__row cart-summary__row--discount">
                    <span>Discount ({discount}%)</span>
                    <span>−₹{discountAmount.toLocaleString()}</span>
                  </div>
                )}
                <div className="cart-summary__row cart-summary__row--total">
                  <strong>Total</strong>
                  <strong>₹{total.toLocaleString()}</strong>
                </div>
              </div>

              <div className="cart-actions">
                <button className="btn btn--outline" onClick={clearCart}>Clear Cart</button>
                <button className="btn btn--primary">Checkout →</button>
              </div>
            </>
          )}
        </aside>
      )}
    </div>
  );
}

export default ShoppingCart;
```

---

## 12. Real-World Use Cases

### 12.1 UI Toggle States

```jsx
function Dropdown({ label, items }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="dropdown">
      <button
        onClick={() => setIsOpen(prev => !prev)}
        aria-haspopup="true"
        aria-expanded={isOpen}
      >
        {label} {isOpen ? '▲' : '▼'}
      </button>
      {isOpen && (
        <ul className="dropdown__menu" role="menu">
          {items.map(item => (
            <li key={item.value} role="menuitem">
              <a href={item.href}>{item.label}</a>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### 12.2 Search and Filter

```jsx
function ProductSearch({ products }) {
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');
  const [sortBy, setSortBy] = useState('name');

  // All derived — no extra state needed
  const filtered = products
    .filter(p => category === 'all' || p.category === category)
    .filter(p => p.name.toLowerCase().includes(query.toLowerCase()))
    .sort((a, b) => {
      if (sortBy === 'price-asc') return a.price - b.price;
      if (sortBy === 'price-desc') return b.price - a.price;
      return a.name.localeCompare(b.name);
    });

  return (
    <div>
      <input
        type="search"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      <select value={category} onChange={e => setCategory(e.target.value)}>
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      <select value={sortBy} onChange={e => setSortBy(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="price-asc">Price: Low to High</option>
        <option value="price-desc">Price: High to Low</option>
      </select>
      <p>{filtered.length} products found</p>
      <ProductGrid products={filtered} />
    </div>
  );
}
```

### 12.3 Async State — Data Fetching Pattern

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch user');
        return res.json();
      })
      .then(data => setUser(data))
      .catch(err => setError(err.message))
      .finally(() => setIsLoading(false));
  }, [userId]);

  if (isLoading) return <Skeleton />;
  if (error)     return <ErrorMessage message={error} />;
  if (!user)     return null;

  return <UserCard user={user} />;
}
```

---

## 13. Best Practices

### 13.1 Name State Clearly

```jsx
// ❌ Unclear
const [data, setData] = useState(null);
const [flag, setFlag] = useState(false);
const [val, setVal] = useState('');

// ✅ Self-documenting
const [currentUser, setCurrentUser] = useState(null);
const [isMenuOpen, setIsMenuOpen] = useState(false);
const [searchQuery, setSearchQuery] = useState('');
```

### 13.2 Keep State as Flat as Possible

```jsx
// ❌ Deeply nested — hard to update
const [state, setState] = useState({
  user: {
    profile: {
      settings: {
        notifications: { email: true, sms: false }
      }
    }
  }
});

// ✅ Flat — easy to update
const [emailNotifications, setEmailNotifications] = useState(true);
const [smsNotifications, setSmsNotifications] = useState(false);
```

### 13.3 Group Related State

```jsx
// ❌ Separate state for related async data — easy to get out of sync
const [user, setUser] = useState(null);
const [userLoading, setUserLoading] = useState(false);
const [userError, setUserError] = useState(null);

// ✅ Grouped as a single object
const [userState, setUserState] = useState({
  data: null,
  isLoading: false,
  error: null,
});

// Update atomically
setUserState({ data: fetchedUser, isLoading: false, error: null });
```

### 13.4 Use Functional Updates When Depending on Previous State

```jsx
// ✅ Always safe — uses the latest state value
setCount(prev => prev + 1);
setItems(prev => [...prev, newItem]);
setIsOpen(prev => !prev);
```

### 13.5 Don't Over-State — Use Derived Values

```jsx
// ❌ Storing derived data
const [total, setTotal] = useState(0); // keep in sync with items manually

// ✅ Compute it
const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
```

### 13.6 Initialize State Outside the Component When Possible

```jsx
// ❌ Object created fresh every render (harmless but slightly wasteful)
function Form() {
  const [data, setData] = useState({ name: '', email: '' });
}

// ✅ Define initial state outside the component
const INITIAL_FORM_DATA = { name: '', email: '' };

function Form() {
  const [data, setData] = useState(INITIAL_FORM_DATA);
  function reset() { setData(INITIAL_FORM_DATA); }
}
```

---

## 14. Common Mistakes

### Mistake 1: Mutating State Directly

```jsx
// ❌ Mutating array state
const [items, setItems] = useState([1, 2, 3]);

function addItem() {
  items.push(4);  // Mutation — React won't detect this change
  setItems(items);
}

// ✅ Create a new array
function addItem() {
  setItems(prev => [...prev, 4]);
}
```

### Mistake 2: Using Stale State in Event Handlers

```jsx
// ❌ Stale closure — count is always the value at the time of the last render
const [count, setCount] = useState(0);

useEffect(() => {
  const interval = setInterval(() => {
    setCount(count + 1); // count is stale inside setInterval
  }, 1000);
  return () => clearInterval(interval);
}, []); // count not in dependency array

// ✅ Use functional update — always gets the latest value
useEffect(() => {
  const interval = setInterval(() => {
    setCount(prev => prev + 1); // No dependency on stale count
  }, 1000);
  return () => clearInterval(interval);
}, []);
```

### Mistake 3: Setting State in Render

```jsx
// ❌ Causes infinite re-render loop
function BrokenComponent() {
  const [count, setCount] = useState(0);
  setCount(1); // Called during render → triggers re-render → called again → infinite loop
  return <p>{count}</p>;
}

// ✅ Only update state in event handlers or effects
function FixedComponent() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(1)}>{count}</button>;
}
```

### Mistake 4: Storing Derived State

```jsx
// ❌ Derived state — total can get out of sync with items
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);

// ✅ Derive total from items
const total = items.reduce((sum, item) => sum + item.price, 0);
```

### Mistake 5: Forgetting Spread When Updating Object State

```jsx
// ❌ Replaces the entire state object with only { name }
const [user, setUser] = useState({ name: 'Rahul', email: 'r@test.com', age: 25 });

setUser({ name: 'Priya' }); // email and age are now gone!

// ✅ Spread existing state, override only what changed
setUser(prev => ({ ...prev, name: 'Priya' }));
```

### Mistake 6: Calling useState Inside Conditions or Loops

```jsx
// ❌ Breaks React's hook order rules
function Bad({ show }) {
  if (show) {
    const [value, setValue] = useState(''); // Conditional hook call — ILLEGAL
  }
}

// ✅ Always call at top level
function Good({ show }) {
  const [value, setValue] = useState('');
  if (!show) return null;
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}
```

### Mistake 7: Expecting State to Update Immediately

```jsx
// ❌ count is still the old value on the next line
function handleClick() {
  setCount(count + 1);
  console.log(count); // Still the old value! State updates are async
}

// ✅ Use the new value directly or use useEffect to react to changes
function handleClick() {
  const newCount = count + 1;
  setCount(newCount);
  console.log(newCount); // ✓ The value you just set
}
```

---

## 15. Performance Considerations

### 15.1 Avoid Unnecessary State

Every state update triggers a re-render. Minimize state to only what truly needs to cause re-renders.

```jsx
// ❌ Storing non-reactive data in state
const [apiUrl] = useState('https://api.example.com'); // never changes

// ✅ Use a constant
const API_URL = 'https://api.example.com';
```

### 15.2 Colocate State

Keep state as **close to where it's used** as possible. Don't lift state higher than necessary.

```jsx
// ❌ tooltip open state in the App root (too high)
function App() {
  const [isTooltipOpen, setIsTooltipOpen] = useState(false); // nobody else needs this
  return <Page isTooltipOpen={isTooltipOpen} setIsTooltipOpen={setIsTooltipOpen} />;
}

// ✅ State lives where it's used
function Tooltip({ content }) {
  const [isOpen, setIsOpen] = useState(false); // local — perfect
  return (
    <div onMouseEnter={() => setIsOpen(true)} onMouseLeave={() => setIsOpen(false)}>
      {isOpen && <div className="tooltip">{content}</div>}
    </div>
  );
}
```

### 15.3 useMemo for Expensive Derived Values

```jsx
function ProductList({ products, searchQuery }) {
  // ❌ Recalculates on every render (even unrelated state changes)
  const filtered = products.filter(p =>
    p.name.toLowerCase().includes(searchQuery.toLowerCase())
  );

  // ✅ Recalculates only when products or searchQuery changes
  const filtered = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(searchQuery.toLowerCase())
    ),
    [products, searchQuery]
  );
}
```

### 15.4 Split State to Prevent Wide Re-renders

```jsx
// ❌ One big state object — any change re-renders everything
const [appState, setAppState] = useState({
  user: null,
  theme: 'light',
  notifications: [],
  cartItems: [],
});

// ✅ Separate state — each change only re-renders what uses that state
const [user, setUser] = useState(null);
const [theme, setTheme] = useState('light');
const [notifications, setNotifications] = useState([]);
const [cartItems, setCartItems] = useState([]);
```

---

## 16. State vs Props — Full Comparison

| Aspect | State | Props |
|--------|-------|-------|
| **Where it lives** | Inside the component | Passed from parent |
| **Who controls it** | The component itself | The parent component |
| **Can it change?** | Yes — via setter function | No — read-only inside component |
| **Triggers re-render?** | Yes — always | Only if parent passes new value |
| **Initial value** | Set in `useState()` | Provided by parent's JSX |
| **When to use** | Data that changes over time | Data component receives from outside |
| **Analogy** | A person's current mood | Instructions given to them |

### A Component Can Have Both

```jsx
// ProductCard has BOTH props (from parent) and state (its own)
function ProductCard({ product, onAddToCart }) {  // ← props
  const [quantity, setQuantity] = useState(1);    // ← state
  const [isWishlisted, setIsWishlisted] = useState(false); // ← state

  return (
    <div>
      <h3>{product.name}</h3>  {/* from props */}
      <p>Qty: {quantity}</p>   {/* from state */}
      <button onClick={() => setQuantity(q => q + 1)}>+</button>
      <button onClick={() => onAddToCart(product.id, quantity)}>Add to Cart</button>
    </div>
  );
}
```

---

## 17. Interview Questions

### Q1: What is state in React and how is it different from a regular variable?

**Answer:** State is a reactive data store managed by React outside the component function. Unlike regular variables that are reset to their initial value on every render, state persists between renders. More importantly, when state changes (via the setter function), React automatically re-renders the component. Regular variables cannot trigger re-renders.

---

### Q2: What does useState return and how do you use it?

**Answer:** `useState(initialValue)` returns an array with exactly two elements: the current state value and a setter function. By convention, you destructure it: `const [value, setValue] = useState(initialValue)`. You read `value` in JSX and call `setValue(newValue)` to update it and trigger a re-render. The initial value is only used on the first render.

---

### Q3: Why should you use the functional form of setState (`prev => prev + 1`)?

**Answer:** When the new state depends on the previous state, the functional form guarantees you're working with the most up-to-date value. Without it, you risk using a stale value captured in a closure — especially when multiple updates happen in the same event handler or inside async callbacks like `setInterval`. For example, `setCount(count + 1)` called three times will only increment once, while `setCount(prev => prev + 1)` called three times will correctly increment three times.

---

### Q4: Why can't you mutate state directly in React?

**Answer:** React uses reference equality (`===`) to detect state changes. If you mutate an object or array in place, the reference stays the same, so React thinks nothing changed and skips the re-render. Additionally, mutation breaks React's ability to compare previous and current state for features like time-travel debugging, concurrent rendering, and React DevTools. You must always create a new value (using spread, `map`, `filter`, etc.) to trigger proper re-renders.

---

### Q5: What is the Rule of Hooks and why does it exist?

**Answer:** The Rule of Hooks states that you must only call hooks at the **top level** of a function component — never inside conditions, loops, or nested functions. This rule exists because React identifies each hook by its **call order**. On every render, React expects the same hooks to be called in the same order. If a hook is conditionally called, the order changes between renders, and React reads values from the wrong state slot, causing bugs.

---

### Q6: What is derived state and why should you avoid storing it in state?

**Answer:** Derived state is any value that can be computed from existing state or props. Storing it as a separate state variable creates a synchronization problem — two sources of truth that can become inconsistent. For example, if you store both `items` and `total` as state, you have to remember to update `total` every time `items` changes. Instead, compute derived values directly during rendering: `const total = items.reduce(...)`. This is always in sync and requires no extra state management.

---

### Q7: What is "lifting state up" and when should you do it?

**Answer:** Lifting state up means moving state to the closest common ancestor when multiple components need to share it. If two sibling components need the same data, neither should own it — the parent should hold the state and pass it down via props, with callback functions for updates. You lift state when: two components need to reflect the same data, a parent needs to react to changes in a child, or siblings need to stay in sync.

---

### Q8: What is lazy initialization in useState?

**Answer:** Lazy initialization is passing a **function** to `useState` instead of a value. The function runs only once on the initial render, not on every re-render. This is a performance optimization for cases where computing the initial state is expensive — like reading from `localStorage`, parsing a large dataset, or complex calculations. Example: `useState(() => JSON.parse(localStorage.getItem('cart')) || [])`.

---

### Q9: What is state batching in React 18?

**Answer:** State batching is React's optimization of grouping multiple `setState` calls into a single re-render. In React 18, batching happens automatically in all contexts — event handlers, `setTimeout`, `Promise` callbacks, and native event listeners. Before React 18, batching only worked inside React event handlers; async code triggered multiple re-renders. With automatic batching, calling `setName('')` and `setEmail('')` in the same function results in one re-render, not two.

---

### Q10: When would you use a single state object vs multiple state variables?

**Answer:** Use **separate state variables** when the values are independent and change at different times — like `isOpen`, `searchQuery`, and `currentUser`. Use a **single state object** when multiple values always change together and are tightly coupled — like a form with multiple fields (`{ name, email, password }`), or an async resource's status (`{ data, isLoading, error }`). Using a single object for unrelated state is an anti-pattern because updating one field causes unnecessary re-renders tied to the other fields.

---

## 18. Practice Tasks

### Task 1 — Beginner: Light/Dark Mode Toggle

Build a `ThemeToggle` component.

**Requirements:**
- Maintain a `isDarkMode` boolean state (default: `false`)
- Display a button showing "🌙 Dark Mode" or "☀️ Light Mode" based on current state
- When toggled, change the `background-color` and `color` of the entire page wrapper div using inline styles
- Show the current theme name as text: "Current: Light" or "Current: Dark"
- Store the user's preference in `localStorage` using lazy initialization

---

### Task 2 — Intermediate: Character Counter with Validation

Build a `SmartTextarea` component.

**Requirements:**
- Accept `maxLength` (number) and `label` (string) as props
- Maintain `text` state (string, default: `''`)
- Display a textarea bound to `text` state (controlled input)
- Show character count: "47 / 280 characters"
- Change counter color:
  - Green: `< 60%` of max used
  - Orange: `60%–90%` used
  - Red: `> 90%` used
- If `maxLength` is exceeded, show an error message and disable a "Submit" button
- Show remaining characters: "233 remaining"
- Add a "Clear" button that resets the text using functional update

---

### Task 3 — Advanced: Kanban Board

Build a mini Kanban board with drag-free state management.

**Requirements:**
- Three columns: "To Do", "In Progress", "Done"
- State: array of tasks `{ id, title, description, column, priority }`
- Add new tasks via a form (title + priority dropdown: low/medium/high)
- Move tasks between columns using "→" and "←" buttons
- Delete tasks with a confirmation step (show "Confirm delete?" with Yes/No buttons)
- Filter tasks by priority using a top-level filter (all/low/medium/high)
- Show task count per column
- Sort tasks within each column by priority (high → medium → low)
- All state management using only `useState` — no external libraries
- Persist board state to `localStorage` using lazy initialization

---

## 19. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| State | Reactive data that triggers re-renders when changed |
| Regular variable | Resets every render, doesn't trigger re-renders |
| useState | Returns `[currentValue, setterFn]` |
| Initial value | Only used on first render — pass function for expensive computation |
| Setter function | Always creates a new render — never mutate directly |
| Functional update | `prev => newValue` — use when new value depends on old |
| Object state | Always spread: `{ ...prev, changed: newValue }` |
| Array state | Use `map`, `filter`, spread — never `push`, `splice`, `sort` in-place |
| Derived state | Compute from state/props — don't store it |
| Lift state up | Move to common ancestor when siblings need to share data |
| Rule of Hooks | Call hooks at top level only — never in conditions or loops |
| Batching | React 18 groups multiple setters into one re-render automatically |

### The Mental Model for State

```
User Action
    ↓
Event Handler calls setter (setCount, setUser, etc.)
    ↓
React schedules a re-render
    ↓
Component function runs again
    ↓
React reads the new state value from its internal store
    ↓
React compares new output with previous output (reconciliation)
    ↓
React updates only the changed parts of the real DOM
    ↓
User sees the update on screen
```

### State Update Rules — Quick Reference

```jsx
// Number/String/Boolean — set directly
setState(newValue);
setState(prev => computeNewValue(prev));

// Object — always spread
setState(prev => ({ ...prev, field: newValue }));

// Array add — spread
setState(prev => [...prev, newItem]);

// Array remove — filter
setState(prev => prev.filter(item => item.id !== targetId));

// Array update — map
setState(prev => prev.map(item => item.id === targetId ? { ...item, field: newValue } : item));

// Toggle — functional update
setState(prev => !prev);
```

---

> **Next Topic:** `05-useEffect-and-lifecycle.md` — Running side effects in function components: data fetching, subscriptions, timers, and understanding the component lifecycle through the useEffect hook.


# 05 — useEffect & Component Lifecycle (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** useEffect Hook & Component Lifecycle
> **Level:** Beginner → Advanced
> **Prerequisites:** State & useState (Module 04), Components & Props (Module 03)

---

## Table of Contents

1. [What is a Side Effect?](#1-what-is-a-side-effect)
2. [What is useEffect?](#2-what-is-useeffect)
3. [The Component Lifecycle](#3-the-component-lifecycle)
4. [How useEffect Works Internally](#4-how-useeffect-works-internally)
5. [useEffect — Complete Syntax Guide](#5-useeffect--complete-syntax-guide)
6. [The Dependency Array — Deep Dive](#6-the-dependency-array--deep-dive)
7. [The Cleanup Function](#7-the-cleanup-function)
8. [Common useEffect Patterns](#8-common-useeffect-patterns)
9. [Data Fetching with useEffect](#9-data-fetching-with-useeffect)
10. [Code Examples (Beginner → Advanced)](#10-code-examples-beginner--advanced)
11. [Real-World Use Cases](#11-real-world-use-cases)
12. [Best Practices](#12-best-practices)
13. [Common Mistakes](#13-common-mistakes)
14. [Performance Considerations](#14-performance-considerations)
15. [useEffect vs useLayoutEffect](#15-useeffect-vs-uselayouteffect)
16. [Interview Questions](#16-interview-questions)
17. [Practice Tasks](#17-practice-tasks)
18. [Summary](#18-summary)

---

## 1. What is a Side Effect?

### Simple Explanation

A **side effect** is anything a component does that reaches **outside** of itself — beyond just computing and returning JSX.

Think of it this way: a component's *pure job* is to take props/state and return JSX. Anything beyond that is a side effect.

**Real-world analogy:** Imagine a chef whose job is to cook a dish (pure function). A side effect would be the chef also calling the supplier to order ingredients, cleaning the kitchen, or turning on the exhaust fan — actions that happen *alongside* the main job but aren't part of cooking itself.

### Examples of Side Effects

| Side Effect | Description |
|-------------|-------------|
| **Data fetching** | Calling an API to load users, products, etc. |
| **DOM manipulation** | Directly reading or changing `document.title`, scroll position |
| **Subscriptions** | Listening to WebSocket messages, events |
| **Timers** | `setTimeout`, `setInterval` |
| **localStorage** | Reading from or writing to browser storage |
| **Logging** | Sending analytics or error reports |
| **Third-party integrations** | Initializing a map, chart library, or analytics SDK |

### Why Side Effects Need Special Handling

React renders components frequently — on every state or prop change. If you put side effects directly in the render body, they run on every render, which causes:

```jsx
// ❌ Side effect in render body — runs on EVERY render
function UserProfile({ userId }) {
  fetch(`/api/users/${userId}`)  // Called on EVERY render — infinite API calls!
    .then(r => r.json())
    .then(data => console.log(data));

  return <div>Profile</div>;
}
```

React needs a way to say: *"Run this code, but only at specific times — not every render."* That's what `useEffect` is for.

---

## 2. What is useEffect?

### Definition

`useEffect` is a React Hook that lets you **synchronize a component with an external system** — or more practically, lets you run code **after React has rendered** the component to the DOM.

```jsx
import { useEffect } from 'react';

function DocumentTitle({ title }) {
  useEffect(() => {
    document.title = title; // runs after render
  }, [title]); // only when title changes

  return <h1>{title}</h1>;
}
```

### The Core Mental Model

> **"useEffect lets you step outside React's rendering cycle and do something with the real world."**

Before hooks, this was done using class component lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). `useEffect` unifies all of these into a single, composable API.

```
Component renders → React updates the DOM → useEffect runs
```

---

## 3. The Component Lifecycle

Every React component goes through three phases. Understanding these is critical to using `useEffect` correctly.

### Phase 1: Mount (Birth)

The component appears in the DOM for the first time.

- React calls your function component
- React updates the DOM
- `useEffect` callbacks run (after paint)

### Phase 2: Update (Life)

The component re-renders because state or props changed.

- React calls your function component again
- React reconciles and updates the DOM
- `useEffect` cleanup for the previous render runs
- `useEffect` callbacks run again (only those whose dependencies changed)

### Phase 3: Unmount (Death)

The component is removed from the DOM.

- React runs the cleanup function of all active `useEffect`s
- React removes the component from the DOM

### Visualizing the Lifecycle

```
MOUNT
  ├── Component function runs (render)
  ├── React updates DOM
  └── useEffect() runs ← "componentDidMount"

UPDATE (state/props change)
  ├── Component function runs again (re-render)
  ├── React reconciles DOM
  ├── Previous useEffect cleanup runs ← "componentWillUnmount" (partial)
  └── useEffect() runs again ← "componentDidUpdate"

UNMOUNT
  └── useEffect cleanup runs ← "componentWillUnmount"
```

### Mapping Class Lifecycle to useEffect

| Class Lifecycle Method | useEffect Equivalent |
|------------------------|----------------------|
| `componentDidMount` | `useEffect(() => { ... }, [])` — empty array |
| `componentDidUpdate` | `useEffect(() => { ... }, [dep])` — with dependencies |
| `componentWillUnmount` | `useEffect(() => { return () => cleanup() }, [])` |
| `componentDidMount` + `componentDidUpdate` | `useEffect(() => { ... })` — no array |

---

## 4. How useEffect Works Internally

### Step-by-Step Execution

```
1. React calls your component function (render phase)
2. React computes the new JSX output
3. React commits changes to the real DOM (commit phase)
4. Browser paints the updated UI on screen
5. React runs your useEffect callback (after paint)

On next render (if deps changed):
6. React calls component function again
7. React updates DOM
8. Browser paints
9. React runs the CLEANUP of the previous effect
10. React runs the NEW effect callback
```

### Why useEffect Runs After Paint

`useEffect` is **asynchronous** — it runs *after* the browser has painted the screen. This means it never blocks the user from seeing the UI.

This is intentional. Most side effects (data fetching, subscriptions, analytics) don't need to block the visual render. The user sees the UI instantly, then the effect runs in the background.

> Exception: `useLayoutEffect` runs *before* the browser paints, synchronously. More on this in section 15.

### React StrictMode and Double Invocation

In development with `React.StrictMode`, React intentionally **mounts → unmounts → mounts** every component twice to help detect bugs in effects. This means your effect runs twice on mount in development, but only once in production. This is a feature, not a bug — it helps you catch effects that don't clean up properly.

```jsx
// In StrictMode (dev only), you'll see this sequence:
// 1. Mount → effect runs
// 2. Unmount → cleanup runs
// 3. Mount again → effect runs again
// Production: just 1. Mount → effect runs
```

---

## 5. useEffect — Complete Syntax Guide

### Basic Structure

```jsx
useEffect(
  () => {
    // Effect body — your side effect code goes here

    return () => {
      // Cleanup function (optional)
      // Runs before the next effect and on unmount
    };
  },
  [/* dependency array */]
);
```

### Variation 1: No Dependency Array — Runs After Every Render

```jsx
useEffect(() => {
  console.log('Runs after every single render');
});
```

**When to use:** Rarely. When you genuinely want the effect to run after every render. Most common for logging during debugging.

**Caution:** If this effect updates state, it will cause infinite re-renders.

### Variation 2: Empty Dependency Array — Runs Once on Mount

```jsx
useEffect(() => {
  console.log('Runs only once — when the component mounts');
  fetchInitialData();
  initializeThirdPartyLibrary();
}, []); // ← empty array
```

**When to use:** One-time initialization — fetching initial data, setting up subscriptions, initializing third-party libraries.

### Variation 3: With Dependencies — Runs When Dependencies Change

```jsx
useEffect(() => {
  console.log(`userId changed to: ${userId}`);
  fetchUserData(userId);
}, [userId]); // ← runs on mount AND whenever userId changes
```

**When to use:** Whenever you need to react to a specific prop or state change.

### Variation 4: With Cleanup — Runs Cleanup Before Next Effect and on Unmount

```jsx
useEffect(() => {
  const subscription = subscribeToEvents(channelId, handleEvent);

  return () => {
    subscription.unsubscribe(); // cleanup
  };
}, [channelId]);
```

### Multiple useEffect Calls

You can (and should) use multiple `useEffect` calls to separate concerns:

```jsx
function UserDashboard({ userId }) {
  // Effect 1: Fetch user data when userId changes
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  // Effect 2: Update document title when user loads
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Dashboard`;
    }
  }, [user]);

  // Effect 3: Set up analytics on mount
  useEffect(() => {
    analytics.trackPageView('dashboard');
    return () => analytics.endSession();
  }, []);
}
```

---

## 6. The Dependency Array — Deep Dive

The dependency array is the most misunderstood part of `useEffect`. Getting it wrong causes the most common React bugs.

### The Rule

> **Every reactive value used inside the effect MUST be listed in the dependency array.**

A "reactive value" is any value that can change between renders: state variables, props, variables derived from state/props, and functions defined inside the component.

### What Belongs in the Dependency Array

```jsx
function SearchResults({ query, pageSize }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  // ✅ query, pageSize, and page are all reactive — all must be deps
  useEffect(() => {
    fetchResults(query, page, pageSize).then(setResults);
  }, [query, page, pageSize]);
}
```

### What Does NOT Belong in the Dependency Array

```jsx
// These do NOT need to be in deps:

// 1. State setter functions — React guarantees they are stable
useEffect(() => {
  fetchUser().then(data => setUser(data)); // setUser is stable
}, []); // setUser omitted — correct

// 2. Values from outside the component (module-level constants)
const API_URL = 'https://api.example.com';
useEffect(() => {
  fetch(API_URL); // API_URL never changes
}, []); // API_URL omitted — correct

// 3. React built-ins (refs, etc.)
useEffect(() => {
  inputRef.current.focus(); // refs are stable
}, []);
```

### The Exhaustive Deps ESLint Rule

The `eslint-plugin-react-hooks` package provides the `exhaustive-deps` rule which warns you when your dependency array is incomplete. Always have this enabled:

```bash
npm install eslint-plugin-react-hooks --save-dev
```

```json
// .eslintrc
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

### The Stale Closure Problem

If you omit a dependency, your effect captures a **stale** (outdated) value from a previous render:

```jsx
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      // ❌ count is stale — always 0 because [] means this
      // effect only runs once and captures count = 0
      console.log(count); // always prints 0
      setCount(count + 1); // always sets to 1
    }, 1000);

    return () => clearInterval(interval);
  }, []); // ← missing count dependency
}
```

**Solutions:**

```jsx
// Solution 1 — Add count to deps (effect re-registers on every count change)
useEffect(() => {
  const interval = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(interval);
}, [count]); // ← now correct but re-runs every second

// Solution 2 — Use functional update (no need for count in deps at all)
useEffect(() => {
  const interval = setInterval(() => {
    setCount(prev => prev + 1); // ✅ No stale closure — functional update
  }, 1000);
  return () => clearInterval(interval);
}, []); // ← correct! count not needed
```

### Object and Function Dependencies

Objects and functions created inside the component are **new references every render**, which causes effects to run too often:

```jsx
// ❌ options is a new object every render → effect runs every render
function DataFetcher({ userId }) {
  const options = { method: 'GET', headers: { Auth: token } }; // new ref every render

  useEffect(() => {
    fetch(`/api/users/${userId}`, options);
  }, [userId, options]); // options always changes → runs on every render
}

// ✅ Solution 1 — Move object inside the effect
useEffect(() => {
  const options = { method: 'GET', headers: { Auth: token } };
  fetch(`/api/users/${userId}`, options);
}, [userId, token]);

// ✅ Solution 2 — useMemo to stabilize reference
const options = useMemo(
  () => ({ method: 'GET', headers: { Auth: token } }),
  [token]
);

useEffect(() => {
  fetch(`/api/users/${userId}`, options);
}, [userId, options]);

// ✅ Solution 3 — useCallback for function deps
const fetchData = useCallback(() => {
  fetch(`/api/users/${userId}`);
}, [userId]);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

---

## 7. The Cleanup Function

### What is Cleanup?

The cleanup function is the optional function you **return** from `useEffect`. React calls it:

1. **Before running the effect again** (when dependencies change)
2. **When the component unmounts**

Cleanup prevents **memory leaks**, **duplicate subscriptions**, and **race conditions**.

### Pattern 1: Clearing Timers

```jsx
function Countdown({ seconds }) {
  const [remaining, setRemaining] = useState(seconds);

  useEffect(() => {
    if (remaining <= 0) return;

    const timer = setTimeout(() => {
      setRemaining(prev => prev - 1);
    }, 1000);

    return () => clearTimeout(timer); // ✅ Cancel timer on cleanup
  }, [remaining]);

  return <p>Time left: {remaining}s</p>;
}
```

### Pattern 2: Removing Event Listeners

```jsx
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    function handleResize() {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    }

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize); // ✅ Remove listener
    };
  }, []); // Only set up once

  return <p>{size.width} × {size.height}</p>;
}
```

### Pattern 3: Cancelling API Requests (AbortController)

```jsx
function UserData({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const controller = new AbortController(); // ✅ Create abort controller

    async function fetchUser() {
      try {
        const res = await fetch(`/api/users/${userId}`, {
          signal: controller.signal // pass signal to fetch
        });
        const data = await res.json();
        setUser(data);
      } catch (err) {
        if (err.name === 'AbortError') return; // ignore cancelled requests
        console.error(err);
      }
    }

    fetchUser();

    return () => {
      controller.abort(); // ✅ Cancel the request on cleanup
    };
  }, [userId]);

  return user ? <div>{user.name}</div> : <p>Loading...</p>;
}
```

### Pattern 4: Unsubscribing

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const socket = connectToRoom(roomId);

    socket.on('message', (msg) => {
      setMessages(prev => [...prev, msg]);
    });

    return () => {
      socket.disconnect(); // ✅ Disconnect on cleanup
    };
  }, [roomId]); // Re-connect when roomId changes

  return <MessageList messages={messages} />;
}
```

### Why Cleanup Matters: The Race Condition

Without cleanup, you can get a classic race condition in data fetching:

```
User selects userId=1 → fetch starts
User quickly selects userId=2 → fetch starts
userId=2 fetch completes first → shows user 2
userId=1 fetch completes second → OVERWRITES with user 1 ❌
```

The AbortController pattern above prevents this — when userId changes, the previous fetch is cancelled.

---

## 8. Common useEffect Patterns

### Pattern 1: Fetching on Mount

```jsx
useEffect(() => {
  fetchProducts().then(setProducts);
}, []);
```

### Pattern 2: Fetching When a Dependency Changes

```jsx
useEffect(() => {
  fetchUserById(userId).then(setUser);
}, [userId]);
```

### Pattern 3: Syncing with localStorage

```jsx
// Read from localStorage on mount
const [theme, setTheme] = useState(() => localStorage.getItem('theme') || 'light');

// Write to localStorage whenever theme changes
useEffect(() => {
  localStorage.setItem('theme', theme);
}, [theme]);
```

### Pattern 4: Document Title

```jsx
useEffect(() => {
  document.title = `${unreadCount} unread — MyApp`;
}, [unreadCount]);
```

### Pattern 5: Scroll to Top on Route Change

```jsx
function ScrollToTop({ pathname }) {
  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);

  return null;
}
```

### Pattern 6: Focus Management

```jsx
function SearchBar({ isVisible }) {
  const inputRef = useRef(null);

  useEffect(() => {
    if (isVisible && inputRef.current) {
      inputRef.current.focus();
    }
  }, [isVisible]);

  return <input ref={inputRef} type="search" />;
}
```

### Pattern 7: Debouncing

```jsx
function LiveSearch({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      return;
    }

    // Wait 400ms after user stops typing before fetching
    const timer = setTimeout(() => {
      fetchSearchResults(query).then(setResults);
    }, 400);

    return () => clearTimeout(timer); // ✅ Cancel previous timer on each keystroke
  }, [query]);

  return <ResultsList results={results} />;
}
```

### Pattern 8: Polling

```jsx
function LivePriceTracker({ symbol }) {
  const [price, setPrice] = useState(null);

  useEffect(() => {
    async function fetchPrice() {
      const data = await getStockPrice(symbol);
      setPrice(data.price);
    }

    fetchPrice(); // fetch immediately
    const interval = setInterval(fetchPrice, 5000); // then every 5 seconds

    return () => clearInterval(interval); // ✅ Stop polling on unmount
  }, [symbol]);

  return <p>{symbol}: ₹{price ?? '...'}</p>;
}
```

---

## 9. Data Fetching with useEffect

Data fetching is the most common use case for `useEffect`. Here is the complete, production-ready pattern:

### The Complete Async Fetch Pattern

```jsx
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!url) return;

    const controller = new AbortController();
    setIsLoading(true);
    setError(null);

    async function fetchData() {
      try {
        const response = await fetch(url, { signal: controller.signal });

        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name === 'AbortError') return; // request was cancelled — ignore
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    }

    fetchData();

    return () => controller.abort();
  }, [url]);

  return { data, isLoading, error };
}

// Usage
function UserList() {
  const { data: users, isLoading, error } = useFetch('https://api.example.com/users');

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorBanner message={error} />;
  if (!users) return null;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Why You Can't Use async Directly in useEffect

```jsx
// ❌ useEffect callback cannot be async
useEffect(async () => {
  const data = await fetchData(); // This looks right but causes issues
  setData(data);
}, []);
// Problem: async function returns a Promise,
// but useEffect expects either nothing or a cleanup function.
// Returning a Promise confuses React.

// ✅ Define async function inside and call it immediately
useEffect(() => {
  async function load() {
    const data = await fetchData();
    setData(data);
  }
  load();
}, []);

// ✅ Or use .then()
useEffect(() => {
  fetchData().then(setData);
}, []);
```

---

## 10. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Document Title Sync

```jsx
// File: components/PageTitle.jsx
import { useEffect } from 'react';

function PageTitle({ title, suffix = 'MyApp' }) {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = `${title} | ${suffix}`;

    // Restore the previous title when the component unmounts
    return () => {
      document.title = previousTitle;
    };
  }, [title, suffix]);

  return null; // This component renders nothing visual
}

export default PageTitle;

// Usage — place at the top of any page component
function ProductsPage() {
  return (
    <>
      <PageTitle title="Products" />
      <h1>Our Products</h1>
      {/* rest of page */}
    </>
  );
}
```

---

### Example 2 — Intermediate: Click Outside to Close

```jsx
// File: components/Dropdown/Dropdown.jsx
import { useState, useEffect, useRef } from 'react';

function Dropdown({ trigger, items }) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  useEffect(() => {
    if (!isOpen) return; // Only add listener when open

    function handleClickOutside(event) {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target)) {
        setIsOpen(false);
      }
    }

    document.addEventListener('mousedown', handleClickOutside);

    return () => {
      document.removeEventListener('mousedown', handleClickOutside); // ✅ Cleanup
    };
  }, [isOpen]);

  // Close on Escape key
  useEffect(() => {
    if (!isOpen) return;

    function handleKeyDown(e) {
      if (e.key === 'Escape') setIsOpen(false);
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen]);

  return (
    <div ref={dropdownRef} className="dropdown">
      <button
        onClick={() => setIsOpen(prev => !prev)}
        aria-haspopup="listbox"
        aria-expanded={isOpen}
      >
        {trigger}
      </button>

      {isOpen && (
        <ul className="dropdown__menu" role="listbox">
          {items.map(item => (
            <li
              key={item.value}
              role="option"
              className="dropdown__item"
              onClick={() => {
                item.onSelect(item.value);
                setIsOpen(false);
              }}
            >
              {item.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default Dropdown;
```

---

### Example 3 — Intermediate: Debounced Search

```jsx
// File: components/SearchBox/SearchBox.jsx
import { useState, useEffect } from 'react';

function SearchBox({ onResultsChange, searchFn }) {
  const [query, setQuery] = useState('');
  const [isSearching, setIsSearching] = useState(false);

  useEffect(() => {
    if (!query.trim()) {
      onResultsChange([]);
      return;
    }

    setIsSearching(true);

    const debounceTimer = setTimeout(async () => {
      try {
        const results = await searchFn(query);
        onResultsChange(results);
      } catch (err) {
        console.error('Search failed:', err);
        onResultsChange([]);
      } finally {
        setIsSearching(false);
      }
    }, 400);

    return () => {
      clearTimeout(debounceTimer); // Cancel previous search on each keystroke
      setIsSearching(false);
    };
  }, [query, searchFn, onResultsChange]);

  return (
    <div className="search-box">
      <input
        type="search"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
        aria-label="Search"
      />
      {isSearching && <span className="search-box__spinner">⌛</span>}
    </div>
  );
}

export default SearchBox;
```

---

### Example 4 — Advanced: Real-Time Data with WebSocket

```jsx
// File: components/LiveFeed/LiveFeed.jsx
import { useState, useEffect, useCallback } from 'react';

const CONNECTION_STATUS = {
  CONNECTING: 'connecting',
  CONNECTED: 'connected',
  DISCONNECTED: 'disconnected',
  ERROR: 'error',
};

function LiveFeed({ channelId, userId }) {
  const [messages, setMessages] = useState([]);
  const [status, setStatus] = useState(CONNECTION_STATUS.CONNECTING);
  const [onlineCount, setOnlineCount] = useState(0);

  const addMessage = useCallback((msg) => {
    setMessages(prev => [msg, ...prev].slice(0, 100)); // keep last 100 messages
  }, []);

  useEffect(() => {
    if (!channelId) return;

    setStatus(CONNECTION_STATUS.CONNECTING);
    setMessages([]);

    const socket = new WebSocket(`wss://chat.example.com/channels/${channelId}`);

    socket.onopen = () => {
      setStatus(CONNECTION_STATUS.CONNECTED);
      socket.send(JSON.stringify({ type: 'JOIN', userId }));
    };

    socket.onmessage = (event) => {
      const payload = JSON.parse(event.data);

      switch (payload.type) {
        case 'MESSAGE':
          addMessage(payload.message);
          break;
        case 'PRESENCE':
          setOnlineCount(payload.onlineCount);
          break;
        case 'HISTORY':
          setMessages(payload.messages);
          break;
        default:
          break;
      }
    };

    socket.onerror = () => {
      setStatus(CONNECTION_STATUS.ERROR);
    };

    socket.onclose = () => {
      setStatus(CONNECTION_STATUS.DISCONNECTED);
    };

    return () => {
      // ✅ Cleanup: notify server and close connection
      if (socket.readyState === WebSocket.OPEN) {
        socket.send(JSON.stringify({ type: 'LEAVE', userId }));
      }
      socket.close();
    };
  }, [channelId, userId, addMessage]);

  const statusColors = {
    [CONNECTION_STATUS.CONNECTING]: '#f59e0b',
    [CONNECTION_STATUS.CONNECTED]: '#10b981',
    [CONNECTION_STATUS.DISCONNECTED]: '#6b7280',
    [CONNECTION_STATUS.ERROR]: '#ef4444',
  };

  return (
    <div className="live-feed">
      <div className="live-feed__header">
        <span
          className="live-feed__status"
          style={{ color: statusColors[status] }}
        >
          ● {status}
        </span>
        <span className="live-feed__count">{onlineCount} online</span>
      </div>

      <ul className="live-feed__messages">
        {messages.map((msg, i) => (
          <li key={i} className="live-feed__message">
            <strong>{msg.author}</strong>: {msg.text}
            <time dateTime={msg.timestamp}>
              {new Date(msg.timestamp).toLocaleTimeString()}
            </time>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default LiveFeed;
```

---

### Example 5 — Advanced: Custom useFetch Hook

```jsx
// File: hooks/useFetch.js
import { useState, useEffect, useCallback } from 'react';

function useFetch(url, options = {}) {
  const [state, setState] = useState({
    data: null,
    isLoading: !!url,
    error: null,
    statusCode: null,
  });

  const execute = useCallback(async (overrideUrl, overrideOptions = {}) => {
    const targetUrl = overrideUrl || url;
    if (!targetUrl) return;

    setState(prev => ({ ...prev, isLoading: true, error: null }));

    const controller = new AbortController();

    try {
      const response = await fetch(targetUrl, {
        ...options,
        ...overrideOptions,
        signal: controller.signal,
      });

      setState(prev => ({ ...prev, statusCode: response.status }));

      if (!response.ok) {
        throw new Error(`Request failed: ${response.status} ${response.statusText}`);
      }

      const contentType = response.headers.get('content-type');
      const data = contentType?.includes('application/json')
        ? await response.json()
        : await response.text();

      setState({ data, isLoading: false, error: null, statusCode: response.status });
      return data;
    } catch (err) {
      if (err.name === 'AbortError') return;
      setState(prev => ({
        ...prev,
        isLoading: false,
        error: err.message,
      }));
    }
  }, [url]); // eslint-disable-line react-hooks/exhaustive-deps

  useEffect(() => {
    if (!url) return;
    const controller = new AbortController();

    execute(url);

    return () => controller.abort();
  }, [url]); // eslint-disable-line react-hooks/exhaustive-deps

  return { ...state, refetch: () => execute(url) };
}

export default useFetch;

// ─── Usage ───────────────────────────────────────────────
function ProductsPage() {
  const [page, setPage] = useState(1);
  const { data, isLoading, error, refetch } = useFetch(
    `https://api.example.com/products?page=${page}`
  );

  return (
    <div>
      {isLoading && <Spinner />}
      {error && <p>Error: {error} <button onClick={refetch}>Retry</button></p>}
      {data && <ProductGrid products={data.items} />}
      <button onClick={() => setPage(p => p - 1)} disabled={page === 1}>Prev</button>
      <button onClick={() => setPage(p => p + 1)}>Next</button>
    </div>
  );
}
```

---

## 11. Real-World Use Cases

### 11.1 Infinite Scroll

```jsx
function InfiniteList({ fetchItems }) {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const loaderRef = useRef(null);

  useEffect(() => {
    async function loadMore() {
      const newItems = await fetchItems(page);
      if (newItems.length === 0) {
        setHasMore(false);
        return;
      }
      setItems(prev => [...prev, ...newItems]);
    }
    loadMore();
  }, [page, fetchItems]);

  // Intersection Observer to detect when loader is visible
  useEffect(() => {
    if (!hasMore) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setPage(prev => prev + 1);
        }
      },
      { threshold: 0.1 }
    );

    if (loaderRef.current) observer.observe(loaderRef.current);

    return () => observer.disconnect(); // ✅ Cleanup observer
  }, [hasMore]);

  return (
    <div>
      {items.map(item => <ItemCard key={item.id} item={item} />)}
      {hasMore && <div ref={loaderRef} className="loader">Loading more...</div>}
    </div>
  );
}
```

### 11.2 Authentication Token Refresh

```jsx
function useAutoTokenRefresh(token, refreshFn) {
  useEffect(() => {
    if (!token) return;

    // Calculate time until token expires (token contains exp as Unix timestamp)
    const payload = JSON.parse(atob(token.split('.')[1]));
    const expiresIn = payload.exp * 1000 - Date.now() - 60_000; // refresh 1 min before expiry

    if (expiresIn <= 0) {
      refreshFn();
      return;
    }

    const timer = setTimeout(refreshFn, expiresIn);
    return () => clearTimeout(timer); // ✅ Cancel refresh if component unmounts
  }, [token, refreshFn]);
}
```

### 11.3 Online/Offline Status

```jsx
function useNetworkStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const goOnline = () => setIsOnline(true);
    const goOffline = () => setIsOnline(false);

    window.addEventListener('online', goOnline);
    window.addEventListener('offline', goOffline);

    return () => {
      window.removeEventListener('online', goOnline);
      window.removeEventListener('offline', goOffline);
    };
  }, []);

  return isOnline;
}

function App() {
  const isOnline = useNetworkStatus();
  return (
    <>
      {!isOnline && <div className="offline-banner">You are offline</div>}
      <Router />
    </>
  );
}
```

---

## 12. Best Practices

### 12.1 Always Specify the Dependency Array

```jsx
// ❌ No deps — runs after every render (probably not what you want)
useEffect(() => {
  document.title = title;
});

// ✅ With deps — runs only when title changes
useEffect(() => {
  document.title = title;
}, [title]);
```

### 12.2 One Concern Per useEffect

```jsx
// ❌ Mixing concerns in one effect
useEffect(() => {
  fetchUser(userId).then(setUser);
  document.title = `User ${userId}`;
  analytics.track('view_user', { userId });
}, [userId]);

// ✅ Separate effects for separate concerns
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]);

useEffect(() => {
  document.title = `User ${userId}`;
}, [userId]);

useEffect(() => {
  analytics.track('view_user', { userId });
}, [userId]);
```

### 12.3 Always Clean Up After Yourself

If your effect starts something (timer, listener, subscription, request), the cleanup must stop it.

### 12.4 Move Logic into Custom Hooks

```jsx
// ❌ Data fetching logic inline — hard to reuse
function ProductPage() {
  const [products, setProducts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  useEffect(() => {
    fetch('/api/products').then(r => r.json()).then(data => {
      setProducts(data);
      setIsLoading(false);
    });
  }, []);
}

// ✅ Extracted to custom hook — reusable and clean
function useProducts() {
  const [products, setProducts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  useEffect(() => {
    fetch('/api/products').then(r => r.json()).then(data => {
      setProducts(data);
      setIsLoading(false);
    });
  }, []);
  return { products, isLoading };
}

function ProductPage() {
  const { products, isLoading } = useProducts();
}
```

### 12.5 Use the ESLint Plugin

Always run `eslint-plugin-react-hooks` to catch missing dependencies and Rules of Hooks violations.

### 12.6 Prefer Libraries for Data Fetching

For production apps, prefer dedicated data-fetching libraries over raw `useEffect` + `fetch`:

| Library | Best For |
|---------|----------|
| **React Query / TanStack Query** | REST APIs, caching, background refetch |
| **SWR** | Lightweight REST fetching with stale-while-revalidate |
| **Apollo Client** | GraphQL APIs |
| **RTK Query** | Redux-based apps |

These libraries handle caching, deduplication, background refresh, error retries, and race conditions automatically.

---

## 13. Common Mistakes

### Mistake 1: Infinite Loop — State Update in Effect Without Deps

```jsx
// ❌ Infinite loop — effect updates state → re-render → effect runs again
function BadComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(count + 1); // triggers re-render → effect runs again → infinite loop
  }); // no dependency array!
}

// ✅ Add proper dependency or use different approach
useEffect(() => {
  setCount(c => c + 1); // Still bad without a condition to stop it
}, []); // runs once — intentional one-time increment
```

### Mistake 2: Missing Cleanup Causing Memory Leaks

```jsx
// ❌ Event listener is never removed — memory leak
useEffect(() => {
  window.addEventListener('resize', handleResize);
  // forgot return () => window.removeEventListener(...)
}, []);

// ✅ Always clean up
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### Mistake 3: Setting State on Unmounted Component

```jsx
// ❌ Component may unmount before fetch completes — React warning
useEffect(() => {
  fetch('/api/data').then(r => r.json()).then(data => {
    setData(data); // might set state on an unmounted component
  });
}, []);

// ✅ Use AbortController or a mounted flag
useEffect(() => {
  let mounted = true;
  fetch('/api/data')
    .then(r => r.json())
    .then(data => {
      if (mounted) setData(data); // only update if still mounted
    });
  return () => { mounted = false; };
}, []);
```

### Mistake 4: async Directly in useEffect

```jsx
// ❌ Cannot make useEffect callback async directly
useEffect(async () => {
  const data = await fetch('/api');
  setData(data);
}, []);

// ✅ Define async function inside
useEffect(() => {
  async function load() {
    const res = await fetch('/api');
    const data = await res.json();
    setData(data);
  }
  load();
}, []);
```

### Mistake 5: Omitting Dependencies (Stale Closure Bug)

```jsx
// ❌ userId is used but not listed — stale closure
useEffect(() => {
  fetchUser(userId).then(setUser);
}, []); // userId missing! Always fetches the initial userId

// ✅ Include all used reactive values
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]);
```

### Mistake 6: Object/Function as Dependency Without Memoization

```jsx
// ❌ New object reference every render → effect runs every render
function Component({ userId }) {
  const config = { timeout: 5000 }; // new object every render

  useEffect(() => {
    fetchWithConfig(userId, config);
  }, [userId, config]); // config always triggers re-run
}

// ✅ Move static objects outside or use useMemo
const config = { timeout: 5000 }; // defined outside component

function Component({ userId }) {
  useEffect(() => {
    fetchWithConfig(userId, config);
  }, [userId]); // config is stable
}
```

### Mistake 7: Overusing useEffect

```jsx
// ❌ Using useEffect to derive/transform data — not needed
useEffect(() => {
  const fullName = `${firstName} ${lastName}`;
  setFullName(fullName);
}, [firstName, lastName]);

// ✅ Compute during render — no effect needed
const fullName = `${firstName} ${lastName}`;
```

---

## 14. Performance Considerations

### 14.1 Use the Dependency Array Correctly

An effect that runs too often wastes computation and may cause bugs. An effect that runs too rarely causes stale data. The dependency array must be **exact**.

### 14.2 Debounce Effects for User Input

Any effect triggered by fast-changing input (keystrokes, scroll) should be debounced:

```jsx
useEffect(() => {
  const timer = setTimeout(() => {
    // expensive operation
    searchAPI(query);
  }, 300);
  return () => clearTimeout(timer);
}, [query]);
```

### 14.3 Avoid Side Effects in Render

Never put side effects directly in the render body — only in `useEffect`, event handlers, or custom hooks.

### 14.4 Consider useCallback for Effect Dependencies

When a function is a dependency of an effect, wrap it in `useCallback` to prevent the effect from re-running unnecessarily:

```jsx
const fetchData = useCallback(async () => {
  const result = await api.getUsers();
  setUsers(result);
}, []); // stable reference

useEffect(() => {
  fetchData();
}, [fetchData]); // fetchData never changes → effect runs once
```

---

## 15. useEffect vs useLayoutEffect

| Feature | useEffect | useLayoutEffect |
|---------|-----------|-----------------|
| **When it runs** | After browser paint | Before browser paint (synchronous) |
| **Blocks paint?** | No (async) | Yes (sync) |
| **Use case** | Data fetching, subscriptions, logging | DOM measurements, preventing flicker |
| **SSR safe?** | ✅ Yes | ⚠️ Warning on server (no DOM) |
| **Performance** | Better (non-blocking) | Worse (blocks paint) |

### When to Use useLayoutEffect

Use `useLayoutEffect` only when you need to **read layout from the DOM and synchronously re-render** before the browser paints — to prevent visible flickering:

```jsx
// ❌ useEffect — user sees the tooltip in wrong position for a frame (flicker)
useEffect(() => {
  const rect = buttonRef.current.getBoundingClientRect();
  setTooltipPosition({ top: rect.bottom, left: rect.left });
}, []);

// ✅ useLayoutEffect — position is set before the browser paints
useLayoutEffect(() => {
  const rect = buttonRef.current.getBoundingClientRect();
  setTooltipPosition({ top: rect.bottom, left: rect.left });
}, []);
```

**Rule of thumb:** Start with `useEffect`. Only switch to `useLayoutEffect` if you see visual flickering caused by the effect.

---

## 16. Interview Questions

### Q1: What is a side effect in React? Give examples.

**Answer:** A side effect is any operation that reaches outside a component's rendering scope — anything beyond computing JSX from props and state. Examples include: API calls, DOM manipulation (setting `document.title`, scroll position), setting up event listeners, starting timers (`setTimeout`, `setInterval`), WebSocket subscriptions, reading/writing `localStorage`, and logging/analytics calls. Side effects need special handling because React may render components many times, and uncontrolled side effects running on every render cause bugs like infinite API calls.

---

### Q2: What are the three ways to use useEffect based on the dependency array?

**Answer:**
1. **No dependency array** — `useEffect(() => {...})` — runs after every render
2. **Empty array** — `useEffect(() => {...}, [])` — runs once on mount (componentDidMount equivalent)
3. **With dependencies** — `useEffect(() => {...}, [dep1, dep2])` — runs on mount and whenever any listed dependency changes

---

### Q3: What is the cleanup function in useEffect and why is it important?

**Answer:** The cleanup function is the optional function returned from a `useEffect` callback. React runs it before the effect runs again (when deps change) and when the component unmounts. It's critical for preventing memory leaks and bugs: timers must be cleared, event listeners removed, WebSocket connections closed, and in-flight API requests cancelled via `AbortController`. Without cleanup, you accumulate stale listeners and subscriptions that fire even after the component is gone.

---

### Q4: What is a stale closure in useEffect and how do you fix it?

**Answer:** A stale closure happens when an effect captures a variable from a previous render and the dependency array doesn't include it, so the effect never re-runs with the updated value. For example, an effect using `count` in a `setInterval` but with `[]` as deps always sees the initial value of `count`. Fixes: (1) Add the variable to the dependency array so the effect re-runs when it changes, or (2) Use a functional state update (`setCount(prev => prev + 1)`) so the effect doesn't need the current value at all.

---

### Q5: Why can't you make the useEffect callback async directly?

**Answer:** An async function always returns a Promise. React expects the `useEffect` callback to return either nothing (`undefined`) or a cleanup function. If you return a Promise, React can't call it as a cleanup function — it silently ignores it, which means your cleanup never runs. The fix is to define an async function inside the effect and call it: `useEffect(() => { async function load() {...} load(); }, [deps])`.

---

### Q6: How do you prevent a race condition in data fetching with useEffect?

**Answer:** A race condition occurs when multiple fetch requests are in flight and a slower earlier request completes after a faster later one, overwriting the correct data. The fix is to use the `AbortController` API: create a controller before the fetch, pass `controller.signal` to `fetch()`, and in the cleanup function call `controller.abort()`. This cancels the previous fetch when the component re-renders with new dependencies (e.g., a new userId). Catch `AbortError` and ignore it since it's intentional.

---

### Q7: What is the difference between useEffect and useLayoutEffect?

**Answer:** Both run after React commits changes to the DOM, but at different times. `useEffect` runs asynchronously *after* the browser paints the screen — it never blocks the UI. `useLayoutEffect` runs synchronously *before* the browser paints — it blocks painting until it completes. Use `useEffect` for almost everything. Use `useLayoutEffect` only when you need to measure the DOM and update something (like a tooltip position) without letting the user see the intermediate state — to prevent visual flickering.

---

### Q8: What is the Rule of Hooks? Why does it exist?

**Answer:** The Rule of Hooks states: only call hooks at the top level of a React function component (never inside conditions, loops, or nested functions), and only call hooks from React function components or custom hooks. It exists because React tracks hooks by their call order. Each hook call corresponds to a specific "slot" in React's internal array for that component. If you conditionally call hooks, the order changes between renders, and React reads data from the wrong slot, causing unpredictable bugs.

---

### Q9: How do you avoid an effect running too often when it depends on an object or function?

**Answer:** Objects and functions created inside a component are new references on every render, causing effects that depend on them to re-run every render. Solutions: (1) Move the object/function *inside* the effect body so it's not a dependency; (2) Move it *outside* the component if it doesn't depend on props/state; (3) Use `useMemo` to memoize objects or `useCallback` to memoize functions so their references only change when their own dependencies change.

---

### Q10: When should you NOT use useEffect?

**Answer:** You should avoid useEffect for: (1) **Derived state** — compute values from state/props during render instead; (2) **Event-driven operations** — put logic in event handlers, not effects; (3) **Synchronous data transformation** — transform data during render; (4) **Resetting child state** — use `key` prop instead. The React team's documentation notes that many `useEffect` usages in real code are either unnecessary or can be simplified. A good rule of thumb: if the effect sets state based on other state, you probably have derived state and should remove the effect.

---

## 17. Practice Tasks

### Task 1 — Beginner: Live Clock

Build a `LiveClock` component.

**Requirements:**
- Display the current time updated every second using `setInterval`
- Format: `HH:MM:SS` (24-hour)
- Clean up the interval when the component unmounts
- Accept a `timezone` prop (optional) — if provided, show time in that timezone using `Intl.DateTimeFormat`
- Show "Paused" text and stop updating when the browser tab is not visible (use `visibilitychange` event)
- Clean up the visibility event listener too

---

### Task 2 — Intermediate: User Search with Debounce & Abort

Build a `UserSearch` component.

**Requirements:**
- Input field for searching users by name
- Debounce API calls by 400ms
- Fetch from `https://jsonplaceholder.typicode.com/users` and filter client-side by name
- Show a loading spinner while searching
- Handle and display errors gracefully
- Cancel in-flight requests when the query changes (use `AbortController`)
- Show "No results found" when query doesn't match anything
- Show results count: "Found 3 users"
- Clear button to reset the search
- Separate the fetch logic into a `useUserSearch(query)` custom hook

---

### Task 3 — Advanced: Real-Time Notifications System

Build a `NotificationCenter` component.

**Requirements:**
- Simulate WebSocket with `setInterval` that generates random notifications every 3–6 seconds
- Notification object: `{ id, type: 'info'|'success'|'warning'|'error', message, timestamp }`
- Show a bell icon with unread badge count
- Clicking the bell opens/closes the notification panel
- Each notification shows its type icon, message, and relative time ("2 minutes ago")
- Mark individual notifications as read on click
- "Mark all as read" button
- Auto-dismiss `success` and `info` notifications after 5 seconds using `setTimeout`
- Persist notifications to `localStorage` using `useEffect`
- Load from `localStorage` on mount using lazy initialization
- Clean up all timers and the interval on unmount
- Limit to 50 stored notifications (remove oldest when over limit)

---

## 18. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| Side effect | Anything outside computing JSX — fetching, DOM, subscriptions, timers |
| useEffect | Runs code *after* React renders — never blocks the UI |
| No deps | Runs after every render |
| Empty deps `[]` | Runs once on mount |
| With deps `[x, y]` | Runs on mount + when x or y changes |
| Cleanup function | Returned from effect — prevents memory leaks and stale subscriptions |
| Dependency array | Must include every reactive value used inside the effect |
| Stale closure | Effect capturing an outdated value — fix by adding to deps or using functional update |
| Async in effects | Define async function inside effect and call it — never make callback async |
| AbortController | Cancel in-flight fetch requests on cleanup — prevents race conditions |
| useLayoutEffect | Synchronous, before paint — only for DOM measurements to prevent flicker |
| Custom hooks | Extract reusable effect logic into `use*` functions |

### The Dependency Array Decision Tree

```
Does the effect use any value from the component (state, props, variables)?
  ├── No → use []  (run once on mount)
  └── Yes → list every used value in the array
            ├── Is the value a function defined in the component?
            │   └── Wrap it in useCallback first
            └── Is the value an object defined in the component?
                └── Move it inside the effect OR wrap in useMemo
```

### useEffect Lifecycle Mental Map

```
MOUNT:     [] or [deps] → effect runs once
UPDATE:    [deps changed] → cleanup runs → effect runs again
UNMOUNT:   cleanup function runs (always)
```

---

> **Next Topic:** `06-event-handling.md` — Handling user interactions in React: synthetic events, event delegation, common event types, controlled inputs, and real-world form handling patterns.
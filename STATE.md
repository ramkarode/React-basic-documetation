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

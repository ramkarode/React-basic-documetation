# 06 — Event Handling in React (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** Event Handling
> **Level:** Beginner → Advanced
> **Prerequisites:** State & useState (Module 04), Components & Props (Module 03), JSX (Module 02)

---

## Table of Contents

1. [What is Event Handling?](#1-what-is-event-handling)
2. [How React Handles Events Internally](#2-how-react-handles-events-internally)
3. [Synthetic Events](#3-synthetic-events)
4. [Basic Event Syntax in React](#4-basic-event-syntax-in-react)
5. [Common Event Types](#5-common-event-types)
6. [Event Object — Properties & Methods](#6-event-object--properties--methods)
7. [Passing Arguments to Event Handlers](#7-passing-arguments-to-event-handlers)
8. [Controlled vs Uncontrolled Inputs](#8-controlled-vs-uncontrolled-inputs)
9. [Form Handling — Complete Guide](#9-form-handling--complete-guide)
10. [Event Propagation — Bubbling & Capturing](#10-event-propagation--bubbling--capturing)
11. [Code Examples (Beginner → Advanced)](#11-code-examples-beginner--advanced)
12. [Real-World Use Cases](#12-real-world-use-cases)
13. [Best Practices](#13-best-practices)
14. [Common Mistakes](#14-common-mistakes)
15. [Performance Considerations](#15-performance-considerations)
16. [Interview Questions](#16-interview-questions)
17. [Practice Tasks](#17-practice-tasks)
18. [Summary](#18-summary)

---

## 1. What is Event Handling?

### Simple Explanation

**Event handling** is the mechanism that lets your application respond to user actions — clicks, key presses, form submissions, mouse movements, and more.

Every time a user interacts with your UI, the browser generates an **event**. Your job as a React developer is to listen for these events and respond to them by updating state, calling APIs, navigating pages, or any other action.

```jsx
// The simplest event handler — responding to a click
function SaveButton() {
  function handleClick() {
    console.log('Button was clicked!');
  }

  return <button onClick={handleClick}>Save</button>;
}
```

### Technical Explanation

In the browser, events are objects dispatched by the DOM when user interactions occur. Normally in plain JavaScript you'd attach listeners with `addEventListener`. React wraps this system with its own **Synthetic Event** layer, giving you a consistent, cross-browser API with additional React-specific optimizations.

---

## 2. How React Handles Events Internally

### Event Delegation — Not Direct Attachment

This is one of React's most important internal optimizations. React does **not** attach event listeners directly to individual DOM elements. Instead, it attaches **a single listener at the root of the document** (or the React root container in React 17+) and uses **event delegation** to handle all events from there.

```
Traditional DOM approach:
  <button> ← addEventListener('click', handler)  ← per element
  <button> ← addEventListener('click', handler)  ← per element
  <button> ← addEventListener('click', handler)  ← per element
  (1000 buttons = 1000 listeners)

React's approach:
  <div id="root"> ← ONE listener for ALL events
    <button>  ← no direct listener
    <button>  ← no direct listener
    <button>  ← no direct listener
  (1000 buttons = still 1 listener)
```

### How Event Delegation Works

When you click a button deep in the component tree:
1. The browser fires the click event on the button
2. The event **bubbles up** through the DOM to the React root
3. React's single root listener intercepts the event
4. React identifies which component/handler should receive it
5. React creates a **Synthetic Event** wrapping the native event
6. React calls your handler with the Synthetic Event

### React 16 vs React 17+ — Event Attachment Point

```
React 16:  Events attached to document (window level)
React 17+: Events attached to the React root DOM container

Why the change? React 17+ allows multiple React versions on the same page
without their event systems conflicting with each other.
```

---

## 3. Synthetic Events

### What is a Synthetic Event?

A **SyntheticEvent** is React's wrapper around the browser's native event object. It normalizes event properties and methods across all browsers, giving you a single consistent API regardless of whether your user is on Chrome, Firefox, Safari, or Edge.

```jsx
function handleClick(event) {
  // event is a SyntheticEvent — not the native browser event
  console.log(event.type);          // 'click'
  console.log(event.target);        // the DOM element that was clicked
  console.log(event.currentTarget); // the element the handler is attached to
  console.log(event.nativeEvent);   // access the underlying native browser event
}

<button onClick={handleClick}>Click Me</button>
```

### SyntheticEvent vs Native Event

| Feature | Native Event | SyntheticEvent |
|---------|-------------|----------------|
| Browser consistent? | ❌ Varies | ✅ Normalized |
| Access via | `addEventListener` callback | JSX event prop |
| Pooling (React 16) | N/A | Was pooled (now deprecated) |
| Native event access | Direct | `event.nativeEvent` |
| `preventDefault()` | Available | Available (same behavior) |
| `stopPropagation()` | Available | Available (same behavior) |

### Event Pooling (Historical — React 16 and earlier)

In React 16 and earlier, SyntheticEvents were **pooled** — after your handler returned, the event properties were nullified and the object was reused. This caused bugs with async code:

```jsx
// React 16 — Event pooling caused this bug
function handleClick(event) {
  setTimeout(() => {
    console.log(event.target); // null! Event was nullified after handler returned
  }, 100);

  // Fix was to call event.persist() to prevent pooling
  event.persist();
  setTimeout(() => {
    console.log(event.target); // works now
  }, 100);
}
```

**In React 17+, event pooling was removed entirely.** You no longer need `event.persist()`. Events behave like regular JavaScript objects.

---

## 4. Basic Event Syntax in React

### 4.1 Inline Arrow Function

```jsx
<button onClick={() => console.log('clicked')}>Click</button>
```

**When to use:** Simple one-liners, passing arguments, toggling state directly.

### 4.2 Named Handler Function (Preferred)

```jsx
function Form() {
  function handleSubmit() {
    console.log('Form submitted');
  }

  return <button onClick={handleSubmit}>Submit</button>;
}
```

**When to use:** Any logic more than one line. Named functions are easier to test, debug, and read.

### 4.3 Arrow Function Method (in Component Body)

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => setCount(prev => prev + 1);
  const handleDecrement = () => setCount(prev => prev - 1);
  const handleReset = () => setCount(0);

  return (
    <div>
      <button onClick={handleDecrement}>−</button>
      <span>{count}</span>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}
```

### Critical Syntax Rules

```jsx
// ✅ CORRECT — passing a function reference
<button onClick={handleClick}>Click</button>

// ❌ WRONG — calling the function immediately during render
// handleClick() runs right when the component renders, not on click
<button onClick={handleClick()}>Click</button>

// ✅ CORRECT — wrap in arrow function if you need to call with arguments
<button onClick={() => handleDelete(item.id)}>Delete</button>

// ❌ WRONG — this calls handleDelete(item.id) during render
<button onClick={handleDelete(item.id)}>Delete</button>
```

### camelCase Event Names

React uses camelCase for all event names (unlike HTML which uses lowercase):

```jsx
// HTML                        // React JSX
onclick="handler()"     →     onClick={handler}
onchange="handler()"    →     onChange={handler}
onsubmit="handler()"    →     onSubmit={handler}
onkeydown="handler()"   →     onKeyDown={handler}
onmouseenter="handler()" →    onMouseEnter={handler}
```

---

## 5. Common Event Types

### 5.1 Mouse Events

```jsx
function MouseDemo() {
  return (
    <div
      onClick={(e) => console.log('clicked', e.clientX, e.clientY)}
      onDoubleClick={() => console.log('double clicked')}
      onMouseEnter={() => console.log('mouse entered')}
      onMouseLeave={() => console.log('mouse left')}
      onMouseMove={(e) => console.log('mouse at', e.clientX, e.clientY)}
      onMouseDown={() => console.log('mouse button pressed')}
      onMouseUp={() => console.log('mouse button released')}
      onContextMenu={(e) => {
        e.preventDefault(); // prevent default right-click menu
        console.log('right clicked');
      }}
    >
      Interact with me
    </div>
  );
}
```

### 5.2 Keyboard Events

```jsx
function KeyboardDemo() {
  function handleKeyDown(e) {
    console.log('Key:', e.key);          // 'Enter', 'a', 'ArrowUp', etc.
    console.log('Code:', e.code);        // 'KeyA', 'Enter', 'ArrowUp'
    console.log('Shift:', e.shiftKey);   // true if Shift held
    console.log('Ctrl:', e.ctrlKey);     // true if Ctrl held
    console.log('Alt:', e.altKey);       // true if Alt held
    console.log('Meta:', e.metaKey);     // true if Cmd (Mac) held

    // Common keyboard shortcuts
    if (e.key === 'Enter') handleSubmit();
    if (e.key === 'Escape') handleClose();
    if ((e.ctrlKey || e.metaKey) && e.key === 's') {
      e.preventDefault();
      handleSave();
    }
  }

  return (
    <input
      onKeyDown={handleKeyDown}
      onKeyUp={(e) => console.log('Key up:', e.key)}
      onKeyPress={(e) => console.log('Key press:', e.key)} // deprecated — use onKeyDown
    />
  );
}
```

> **Note:** `onKeyPress` is deprecated. Use `onKeyDown` or `onKeyUp` instead.

### 5.3 Form Events

```jsx
function FormEvents() {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault(); // ALWAYS prevent default on forms
        console.log('form submitted');
      }}
    >
      <input
        onChange={(e) => console.log('value:', e.target.value)}
        onFocus={() => console.log('input focused')}
        onBlur={() => console.log('input blurred — user left field')}
      />
      <select onChange={(e) => console.log('selected:', e.target.value)}>
        <option value="a">Option A</option>
        <option value="b">Option B</option>
      </select>
      <input
        type="checkbox"
        onChange={(e) => console.log('checked:', e.target.checked)}
      />
    </form>
  );
}
```

### 5.4 Focus Events

```jsx
function FocusDemo() {
  return (
    <div
      onFocus={() => console.log('focus entered div or child')}  // bubbles
      onBlur={() => console.log('focus left div or child')}      // bubbles
    >
      <input
        onFocus={() => console.log('input focused')}   // does not bubble
        onBlur={() => console.log('input blurred')}    // does not bubble
      />
    </div>
  );
}
```

> **`onFocus` vs `onfocusin`:** React's `onFocus` bubbles (unlike the native `focus` event). This means you can listen on a parent for focus events from any child.

### 5.5 Clipboard Events

```jsx
function ClipboardDemo() {
  return (
    <input
      onCopy={(e) => {
        e.preventDefault();
        console.log('Copy prevented!');
      }}
      onPaste={(e) => {
        const pastedText = e.clipboardData.getData('text');
        console.log('Pasted:', pastedText);
      }}
      onCut={() => console.log('Cut!')}
    />
  );
}
```

### 5.6 Drag Events

```jsx
function DragDemo() {
  return (
    <div
      draggable
      onDragStart={(e) => {
        e.dataTransfer.setData('text/plain', 'some data');
        console.log('drag started');
      }}
      onDragEnd={() => console.log('drag ended')}
    >
      Drag me
    </div>
  );
}

function DropZone() {
  return (
    <div
      onDragOver={(e) => e.preventDefault()} // required to allow drop
      onDrop={(e) => {
        e.preventDefault();
        const data = e.dataTransfer.getData('text/plain');
        console.log('Dropped:', data);
      }}
    >
      Drop here
    </div>
  );
}
```

### 5.7 Scroll & Wheel Events

```jsx
function ScrollDemo() {
  return (
    <div
      style={{ height: 200, overflowY: 'scroll' }}
      onScroll={(e) => {
        const { scrollTop, scrollHeight, clientHeight } = e.target;
        const progress = (scrollTop / (scrollHeight - clientHeight)) * 100;
        console.log(`Scrolled ${progress.toFixed(0)}%`);
      }}
    >
      {/* Long content */}
    </div>
  );
}
```

### 5.8 Touch Events (Mobile)

```jsx
function TouchDemo() {
  return (
    <div
      onTouchStart={(e) => console.log('Touch started', e.touches[0].clientX)}
      onTouchMove={(e) => console.log('Touch moved', e.touches[0].clientX)}
      onTouchEnd={() => console.log('Touch ended')}
    >
      Touch me on mobile
    </div>
  );
}
```

---

## 6. Event Object — Properties & Methods

### Most Important Properties

```jsx
function handleEvent(event) {
  // ── Target Info ──────────────────────────────────────
  event.target          // the element that triggered the event
  event.currentTarget   // the element the handler is attached to
  event.type            // 'click', 'change', 'keydown', etc.

  // ── Mouse Position ───────────────────────────────────
  event.clientX         // X position relative to viewport
  event.clientY         // Y position relative to viewport
  event.pageX           // X position relative to document
  event.pageY           // Y position relative to document
  event.offsetX         // X position relative to target element
  event.offsetY         // Y position relative to target element

  // ── Keyboard ─────────────────────────────────────────
  event.key             // 'Enter', 'a', 'ArrowUp', 'Escape'
  event.code            // 'KeyA', 'Enter', 'ArrowUp'
  event.keyCode         // deprecated — use event.key instead
  event.shiftKey        // boolean
  event.ctrlKey         // boolean
  event.altKey          // boolean
  event.metaKey         // boolean (Cmd on Mac, Win key on Windows)

  // ── Form / Input ─────────────────────────────────────
  event.target.value    // current value of input/select/textarea
  event.target.checked  // boolean — for checkboxes/radio buttons
  event.target.name     // the 'name' attribute of the input
  event.target.files    // FileList — for file inputs

  // ── Propagation Control ──────────────────────────────
  event.preventDefault()    // prevent default browser behavior
  event.stopPropagation()   // stop event from bubbling up
  event.stopImmediatePropagation() // stop other handlers on same element too

  // ── Native Event ─────────────────────────────────────
  event.nativeEvent     // the underlying browser event object
}
```

### preventDefault() — When to Use

```jsx
// 1. Prevent form page reload
<form onSubmit={(e) => {
  e.preventDefault();
  handleSubmit();
}}>

// 2. Prevent link navigation
<a href="/dashboard" onClick={(e) => {
  e.preventDefault();
  // Handle with React Router instead
  navigate('/dashboard');
}}>

// 3. Prevent right-click context menu
<div onContextMenu={(e) => {
  e.preventDefault();
  showCustomMenu(e.clientX, e.clientY);
}}>

// 4. Prevent drag default behavior
<div onDragOver={(e) => e.preventDefault()}>
```

---

## 7. Passing Arguments to Event Handlers

### Problem: Event Handlers Only Receive the Event Object

By default, event handlers receive only the event object. To pass additional data, you need a wrapper:

### Solution 1: Inline Arrow Function (Simple)

```jsx
function ProductList({ products, onDelete }) {
  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {/* Arrow function wraps the call and passes product.id */}
          <button onClick={() => onDelete(product.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

### Solution 2: Curried Function (Clean)

```jsx
// A function that returns a function
function handleDelete(id) {
  return function(event) {
    console.log('Deleting item:', id);
    // event is available if needed
  };
}

// Usage — no inline arrow needed
<button onClick={handleDelete(product.id)}>Delete</button>
```

### Solution 3: data-* Attributes (DOM Approach)

```jsx
function ProductList({ products }) {
  function handleAction(e) {
    const action = e.currentTarget.dataset.action; // 'delete' or 'edit'
    const id = e.currentTarget.dataset.id;
    console.log(`Action: ${action}, ID: ${id}`);
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          <button data-action="edit" data-id={product.id} onClick={handleAction}>
            Edit
          </button>
          <button data-action="delete" data-id={product.id} onClick={handleAction}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

### Solution 4: useCallback for Memoized Handlers

```jsx
function TodoItem({ todo, onToggle, onDelete }) {
  // ✅ Stable references — won't cause child re-renders
  const handleToggle = useCallback(() => onToggle(todo.id), [todo.id, onToggle]);
  const handleDelete = useCallback(() => onDelete(todo.id), [todo.id, onDelete]);

  return (
    <li>
      <input type="checkbox" checked={todo.completed} onChange={handleToggle} />
      <span>{todo.text}</span>
      <button onClick={handleDelete}>Delete</button>
    </li>
  );
}
```

---

## 8. Controlled vs Uncontrolled Inputs

### What is a Controlled Input?

A **controlled input** is one where React fully controls its value via state. The `value` prop is set from state, and `onChange` updates that state. The input always reflects the current state.

```jsx
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    <input
      value={value}                          // React controls the value
      onChange={e => setValue(e.target.value)} // State updates on every keystroke
      placeholder="Controlled input"
    />
  );
}
```

**Data flow:**
```
User types → onChange fires → setState → re-render → input shows new value
```

### What is an Uncontrolled Input?

An **uncontrolled input** stores its own value in the DOM. You use a **ref** to read the value when needed (e.g., on form submit), not on every keystroke.

```jsx
import { useRef } from 'react';

function UncontrolledInput() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Value:', inputRef.current.value); // read when needed
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="Initial value" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Controlled vs Uncontrolled — Comparison

| Feature | Controlled | Uncontrolled |
|---------|-----------|--------------|
| Value source | React state | DOM |
| How to read value | From state variable | Via `ref.current.value` |
| Real-time validation | ✅ Easy | ❌ Difficult |
| Conditional formatting | ✅ Easy | ❌ Difficult |
| Programmatic reset | ✅ `setValue('')` | ✅ `ref.current.value = ''` |
| Form submission | Read from state | Read from ref |
| Performance | Slightly more re-renders | Fewer re-renders |
| Recommended by React | ✅ Yes | For simple/legacy cases |

### The `defaultValue` vs `value` Distinction

```jsx
// Controlled — React owns the value
<input value={name} onChange={e => setName(e.target.value)} />

// Uncontrolled — DOM owns the value, React sets only the initial value
<input defaultValue="Initial Name" ref={inputRef} />

// ❌ Never mix value and defaultValue — leads to warnings
```

---

## 9. Form Handling — Complete Guide

### 9.1 Basic Form with Controlled Inputs

```jsx
function ContactForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    message: '',
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitSuccess, setSubmitSuccess] = useState(false);

  function handleChange(e) {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  }

  function validate() {
    const newErrors = {};
    if (!form.name.trim()) newErrors.name = 'Name is required';
    if (!form.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
      newErrors.email = 'Please enter a valid email';
    }
    if (!form.message.trim()) newErrors.message = 'Message is required';
    if (form.message.length > 500) newErrors.message = 'Max 500 characters';
    return newErrors;
  }

  async function handleSubmit(e) {
    e.preventDefault();
    const validationErrors = validate();
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }

    setIsSubmitting(true);
    try {
      await submitContactForm(form);
      setSubmitSuccess(true);
      setForm({ name: '', email: '', message: '' });
    } catch (err) {
      setErrors({ submit: 'Failed to send. Please try again.' });
    } finally {
      setIsSubmitting(false);
    }
  }

  if (submitSuccess) {
    return <div className="success-message">✅ Message sent! We'll get back to you soon.</div>;
  }

  return (
    <form onSubmit={handleSubmit} noValidate className="contact-form">
      <h2>Contact Us</h2>

      {errors.submit && (
        <div className="form-error form-error--global" role="alert">
          {errors.submit}
        </div>
      )}

      <div className="form-group">
        <label htmlFor="name">Full Name *</label>
        <input
          id="name"
          name="name"
          type="text"
          value={form.name}
          onChange={handleChange}
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
          className={errors.name ? 'input--error' : ''}
        />
        {errors.name && (
          <p id="name-error" className="field-error" role="alert">{errors.name}</p>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email Address *</label>
        <input
          id="email"
          name="email"
          type="email"
          value={form.email}
          onChange={handleChange}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
          className={errors.email ? 'input--error' : ''}
        />
        {errors.email && (
          <p id="email-error" className="field-error" role="alert">{errors.email}</p>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="message">Message *</label>
        <textarea
          id="message"
          name="message"
          value={form.message}
          onChange={handleChange}
          rows={5}
          aria-invalid={!!errors.message}
          aria-describedby={errors.message ? 'message-error' : undefined}
          className={errors.message ? 'input--error' : ''}
        />
        <p className="char-count">{form.message.length}/500</p>
        {errors.message && (
          <p id="message-error" className="field-error" role="alert">{errors.message}</p>
        )}
      </div>

      <button type="submit" disabled={isSubmitting} className="btn btn--primary">
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>
    </form>
  );
}
```

### 9.2 Handling Checkboxes

```jsx
function PreferencesForm() {
  const [preferences, setPreferences] = useState({
    emailNotifications: true,
    smsNotifications: false,
    marketingEmails: false,
    weeklyDigest: true,
  });

  function handleCheckboxChange(e) {
    const { name, checked } = e.target; // use .checked, not .value
    setPreferences(prev => ({ ...prev, [name]: checked }));
  }

  return (
    <fieldset>
      <legend>Notification Preferences</legend>
      {Object.entries(preferences).map(([key, value]) => (
        <label key={key} className="checkbox-label">
          <input
            type="checkbox"
            name={key}
            checked={value}
            onChange={handleCheckboxChange}
          />
          {key.replace(/([A-Z])/g, ' $1').trim()} {/* camelCase to words */}
        </label>
      ))}
    </fieldset>
  );
}
```

### 9.3 Handling Radio Buttons

```jsx
function PaymentMethodSelector() {
  const [paymentMethod, setPaymentMethod] = useState('card');

  const methods = [
    { value: 'card', label: 'Credit/Debit Card' },
    { value: 'upi', label: 'UPI' },
    { value: 'netbanking', label: 'Net Banking' },
    { value: 'cod', label: 'Cash on Delivery' },
  ];

  return (
    <fieldset>
      <legend>Select Payment Method</legend>
      {methods.map(method => (
        <label key={method.value} className="radio-label">
          <input
            type="radio"
            name="paymentMethod"
            value={method.value}
            checked={paymentMethod === method.value}
            onChange={e => setPaymentMethod(e.target.value)}
          />
          {method.label}
        </label>
      ))}
      <p>Selected: {paymentMethod}</p>
    </fieldset>
  );
}
```

### 9.4 Handling File Inputs

```jsx
function FileUploader() {
  const [files, setFiles] = useState([]);
  const [previews, setPreviews] = useState([]);

  function handleFileChange(e) {
    const selectedFiles = Array.from(e.target.files);
    setFiles(selectedFiles);

    // Generate image previews
    const previewURLs = selectedFiles.map(file => URL.createObjectURL(file));
    setPreviews(previewURLs);

    // Cleanup old object URLs to prevent memory leaks
    return () => previewURLs.forEach(URL.revokeObjectURL);
  }

  function handleUpload() {
    const formData = new FormData();
    files.forEach(file => formData.append('files', file));
    // POST formData to your API
  }

  return (
    <div className="file-uploader">
      <input
        type="file"
        accept="image/*"
        multiple
        onChange={handleFileChange}
      />
      <div className="preview-grid">
        {previews.map((url, i) => (
          <img key={i} src={url} alt={`Preview ${i + 1}`} className="preview-image" />
        ))}
      </div>
      {files.length > 0 && (
        <button onClick={handleUpload}>Upload {files.length} file(s)</button>
      )}
    </div>
  );
}
```

### 9.5 Handling Select Dropdowns

```jsx
function CountrySelector() {
  const [country, setCountry] = useState('');
  const [city, setCity] = useState('');

  const cityMap = {
    IN: ['Mumbai', 'Delhi', 'Bengaluru', 'Chennai'],
    US: ['New York', 'Los Angeles', 'Chicago', 'Houston'],
    UK: ['London', 'Manchester', 'Birmingham', 'Leeds'],
  };

  function handleCountryChange(e) {
    setCountry(e.target.value);
    setCity(''); // reset city when country changes
  }

  return (
    <div>
      <select value={country} onChange={handleCountryChange}>
        <option value="">-- Select Country --</option>
        <option value="IN">India</option>
        <option value="US">United States</option>
        <option value="UK">United Kingdom</option>
      </select>

      {country && (
        <select value={city} onChange={e => setCity(e.target.value)}>
          <option value="">-- Select City --</option>
          {cityMap[country].map(c => (
            <option key={c} value={c}>{c}</option>
          ))}
        </select>
      )}

      {city && <p>You selected: {city}, {country}</p>}
    </div>
  );
}
```

---

## 10. Event Propagation — Bubbling & Capturing

### The Three Phases of Event Propagation

When an event fires, it travels through the DOM in three phases:

```
1. CAPTURE PHASE   → Event travels DOWN from document to target
2. TARGET PHASE    → Event fires on the actual target element
3. BUBBLE PHASE    → Event travels UP from target to document (default)
```

```
document
  └── body
        └── div.container   ← 1. Capture (down)
              └── section    ← 2. Capture
                    └── button  ← 3. Target (event fires here)
                    └── section  ← 4. Bubble (up)
              └── div.container  ← 5. Bubble
        └── body               ← 6. Bubble
document                        ← 7. Bubble
```

### Bubbling in React

By default, React events bubble from child to parent:

```jsx
function EventBubbling() {
  return (
    <div onClick={() => console.log('3. DIV clicked')}>
      <section onClick={() => console.log('2. SECTION clicked')}>
        <button onClick={() => console.log('1. BUTTON clicked')}>
          Click me
        </button>
      </section>
    </div>
  );
}

// When button is clicked, console shows:
// 1. BUTTON clicked
// 2. SECTION clicked
// 3. DIV clicked
```

### Stopping Propagation

```jsx
function Modal({ onClose, children }) {
  return (
    <div className="modal-overlay" onClick={onClose}>
      {/* Stop click from reaching overlay — prevents modal from closing */}
      <div
        className="modal-content"
        onClick={e => e.stopPropagation()}
      >
        {children}
      </div>
    </div>
  );
}
```

### Capture Phase Events

React supports capture phase via the `Capture` suffix:

```jsx
<div
  onClickCapture={() => console.log('DIV capture — fires first!')}
  onClick={() => console.log('DIV bubble — fires last')}
>
  <button onClick={() => console.log('BUTTON bubble')}>Click</button>
</div>

// Order:
// 1. DIV capture (capture phase, going down)
// 2. BUTTON bubble (target)
// 3. DIV bubble (bubble phase, going up)
```

**When to use capture:** Rare. Useful when you need to intercept events before they reach their target — e.g., a global error boundary that logs all click events, or preventing certain interactions at a high level.

### Event Delegation Pattern in React

You can handle events for many children from a single parent handler — exactly like React does internally:

```jsx
function MenuList({ items, onSelect }) {
  // Single handler on the parent instead of one per item
  function handleClick(e) {
    const item = e.target.closest('[data-item-id]');
    if (item) {
      onSelect(item.dataset.itemId);
    }
  }

  return (
    <ul onClick={handleClick}>
      {items.map(item => (
        <li key={item.id} data-item-id={item.id}>
          <span className="item-icon">{item.icon}</span>
          <span className="item-label">{item.label}</span>
        </li>
      ))}
    </ul>
  );
}
```

---

## 11. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Interactive Rating Component

```jsx
// File: components/StarRating/StarRating.jsx
import { useState } from 'react';

function StarRating({ maxStars = 5, onRate, initialRating = 0 }) {
  const [rating, setRating] = useState(initialRating);
  const [hoverRating, setHoverRating] = useState(0);

  function handleClick(star) {
    setRating(star);
    onRate?.(star);
  }

  const displayRating = hoverRating || rating;

  return (
    <div className="star-rating" role="group" aria-label="Star rating">
      {Array.from({ length: maxStars }, (_, i) => i + 1).map(star => (
        <button
          key={star}
          type="button"
          className={`star ${star <= displayRating ? 'star--filled' : 'star--empty'}`}
          onClick={() => handleClick(star)}
          onMouseEnter={() => setHoverRating(star)}
          onMouseLeave={() => setHoverRating(0)}
          aria-label={`Rate ${star} out of ${maxStars} stars`}
          aria-pressed={rating === star}
          style={{ background: 'none', border: 'none', cursor: 'pointer', fontSize: '2rem' }}
        >
          {star <= displayRating ? '★' : '☆'}
        </button>
      ))}
      <span className="sr-only">{rating ? `Rated ${rating} stars` : 'Not rated'}</span>
    </div>
  );
}

export default StarRating;
```

---

### Example 2 — Intermediate: Keyboard-Navigable Dropdown

```jsx
// File: components/KeyboardDropdown/KeyboardDropdown.jsx
import { useState, useRef, useEffect } from 'react';

function KeyboardDropdown({ label, options, value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(-1);
  const containerRef = useRef(null);
  const listRef = useRef(null);

  const selectedOption = options.find(o => o.value === value);

  // Close on outside click
  useEffect(() => {
    if (!isOpen) return;
    function handleOutsideClick(e) {
      if (!containerRef.current?.contains(e.target)) setIsOpen(false);
    }
    document.addEventListener('mousedown', handleOutsideClick);
    return () => document.removeEventListener('mousedown', handleOutsideClick);
  }, [isOpen]);

  // Focus the focused item in the list
  useEffect(() => {
    if (isOpen && focusedIndex >= 0) {
      listRef.current?.children[focusedIndex]?.focus();
    }
  }, [focusedIndex, isOpen]);

  function handleButtonKeyDown(e) {
    switch (e.key) {
      case 'Enter':
      case ' ':
      case 'ArrowDown':
        e.preventDefault();
        setIsOpen(true);
        setFocusedIndex(0);
        break;
      case 'Escape':
        setIsOpen(false);
        break;
    }
  }

  function handleOptionKeyDown(e, index, option) {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex(prev => Math.min(prev + 1, options.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex(prev => Math.max(prev - 1, 0));
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        onChange(option.value);
        setIsOpen(false);
        break;
      case 'Escape':
        setIsOpen(false);
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(options.length - 1);
        break;
    }
  }

  return (
    <div ref={containerRef} className="kb-dropdown" style={{ position: 'relative', width: 240 }}>
      <button
        type="button"
        onClick={() => setIsOpen(prev => !prev)}
        onKeyDown={handleButtonKeyDown}
        aria-haspopup="listbox"
        aria-expanded={isOpen}
        aria-labelledby="dropdown-label"
        className="kb-dropdown__trigger"
        style={{ width: '100%', padding: '8px 12px', textAlign: 'left', cursor: 'pointer' }}
      >
        <span id="dropdown-label">{label}: </span>
        <strong>{selectedOption?.label ?? 'Select...'}</strong>
        <span style={{ float: 'right' }}>{isOpen ? '▲' : '▼'}</span>
      </button>

      {isOpen && (
        <ul
          ref={listRef}
          role="listbox"
          aria-label={label}
          className="kb-dropdown__list"
          style={{
            position: 'absolute', top: '100%', left: 0, right: 0,
            listStyle: 'none', margin: 0, padding: 0,
            border: '1px solid #ccc', background: '#fff', zIndex: 100,
          }}
        >
          {options.map((option, index) => (
            <li
              key={option.value}
              role="option"
              tabIndex={-1}
              aria-selected={option.value === value}
              onKeyDown={e => handleOptionKeyDown(e, index, option)}
              onClick={() => {
                onChange(option.value);
                setIsOpen(false);
              }}
              style={{
                padding: '8px 12px',
                cursor: 'pointer',
                background: option.value === value ? '#e0e7ff' : 'transparent',
                outline: 'none',
              }}
              onFocus={e => (e.currentTarget.style.background = '#f3f4f6')}
              onBlur={e => (e.currentTarget.style.background =
                option.value === value ? '#e0e7ff' : 'transparent'
              )}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default KeyboardDropdown;
```

---

### Example 3 — Advanced: Drag-and-Drop Kanban Column

```jsx
// File: components/DragDrop/DragDropList.jsx
import { useState } from 'react';

function DragDropList({ title, items, onDrop }) {
  const [isDragOver, setIsDragOver] = useState(false);
  const [draggedItemId, setDraggedItemId] = useState(null);

  function handleDragStart(e, itemId) {
    setDraggedItemId(itemId);
    e.dataTransfer.effectAllowed = 'move';
    e.dataTransfer.setData('text/plain', itemId.toString());
    e.currentTarget.style.opacity = '0.5';
  }

  function handleDragEnd(e) {
    e.currentTarget.style.opacity = '1';
    setDraggedItemId(null);
  }

  function handleDragOver(e) {
    e.preventDefault(); // required to allow drop
    e.dataTransfer.dropEffect = 'move';
    setIsDragOver(true);
  }

  function handleDragLeave(e) {
    // Only trigger if leaving the column, not its children
    if (!e.currentTarget.contains(e.relatedTarget)) {
      setIsDragOver(false);
    }
  }

  function handleDrop(e) {
    e.preventDefault();
    setIsDragOver(false);
    const itemId = parseInt(e.dataTransfer.getData('text/plain'));
    onDrop(itemId, title); // Notify parent of the drop
  }

  return (
    <div
      className={`kanban-column ${isDragOver ? 'kanban-column--active' : ''}`}
      onDragOver={handleDragOver}
      onDragLeave={handleDragLeave}
      onDrop={handleDrop}
      style={{
        minHeight: 200,
        padding: 16,
        background: isDragOver ? '#e0e7ff' : '#f9fafb',
        border: '2px dashed',
        borderColor: isDragOver ? '#6366f1' : '#e5e7eb',
        borderRadius: 8,
        transition: 'all 0.2s',
      }}
    >
      <h3 style={{ margin: '0 0 12px' }}>
        {title} <span style={{ color: '#6b7280' }}>({items.length})</span>
      </h3>

      {items.map(item => (
        <div
          key={item.id}
          draggable
          onDragStart={e => handleDragStart(e, item.id)}
          onDragEnd={handleDragEnd}
          className="kanban-card"
          style={{
            padding: '10px 14px',
            marginBottom: 8,
            background: '#fff',
            borderRadius: 6,
            boxShadow: '0 1px 3px rgba(0,0,0,0.1)',
            cursor: 'grab',
            userSelect: 'none',
            opacity: draggedItemId === item.id ? 0.5 : 1,
          }}
        >
          <strong>{item.title}</strong>
          <p style={{ margin: '4px 0 0', fontSize: 12, color: '#6b7280' }}>{item.description}</p>
        </div>
      ))}

      {items.length === 0 && (
        <p style={{ color: '#9ca3af', textAlign: 'center', paddingTop: 40 }}>
          Drop items here
        </p>
      )}
    </div>
  );
}

// Parent Board Component
function KanbanBoard() {
  const [board, setBoard] = useState({
    'To Do': [
      { id: 1, title: 'Design mockups', description: 'Create Figma designs' },
      { id: 2, title: 'Write tests', description: 'Unit and integration tests' },
    ],
    'In Progress': [
      { id: 3, title: 'Build API', description: 'REST endpoints' },
    ],
    'Done': [
      { id: 4, title: 'Setup project', description: 'CRA + ESLint + Prettier' },
    ],
  });

  function handleDrop(itemId, targetColumn) {
    setBoard(prev => {
      // Find which column the item currently belongs to
      let movedItem = null;
      const newBoard = {};

      for (const [col, items] of Object.entries(prev)) {
        const found = items.find(i => i.id === itemId);
        if (found) movedItem = found;
        newBoard[col] = items.filter(i => i.id !== itemId);
      }

      if (movedItem) {
        newBoard[targetColumn] = [...newBoard[targetColumn], movedItem];
      }

      return newBoard;
    });
  }

  return (
    <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: 16, padding: 24 }}>
      {Object.entries(board).map(([column, items]) => (
        <DragDropList
          key={column}
          title={column}
          items={items}
          onDrop={handleDrop}
        />
      ))}
    </div>
  );
}

export default KanbanBoard;
```

---

## 12. Real-World Use Cases

### 12.1 Global Keyboard Shortcuts

```jsx
function useGlobalShortcuts(shortcuts) {
  useEffect(() => {
    function handleKeyDown(e) {
      for (const [combo, handler] of Object.entries(shortcuts)) {
        const keys = combo.toLowerCase().split('+');
        const requiresCtrl  = keys.includes('ctrl');
        const requiresMeta  = keys.includes('meta');
        const requiresShift = keys.includes('shift');
        const requiresAlt   = keys.includes('alt');
        const mainKey = keys.find(k => !['ctrl','meta','shift','alt'].includes(k));

        const ctrlOrMeta = requiresCtrl ? e.ctrlKey : requiresMeta ? e.metaKey : true;

        if (
          e.key.toLowerCase() === mainKey &&
          (requiresCtrl || requiresMeta ? ctrlOrMeta : true) &&
          (requiresShift ? e.shiftKey : !e.shiftKey) &&
          (requiresAlt ? e.altKey : !e.altKey)
        ) {
          e.preventDefault();
          handler(e);
          break;
        }
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [shortcuts]);
}

// Usage
function Editor() {
  const [content, setContent] = useState('');

  useGlobalShortcuts({
    'ctrl+s': () => saveDocument(content),
    'ctrl+z': () => undoLastChange(),
    'ctrl+shift+z': () => redoLastChange(),
    'escape': () => closeEditor(),
  });

  return <textarea value={content} onChange={e => setContent(e.target.value)} />;
}
```

### 12.2 Long Press Detection

```jsx
function useLongPress(callback, delay = 500) {
  const timerRef = useRef(null);

  function start(e) {
    timerRef.current = setTimeout(() => callback(e), delay);
  }

  function stop() {
    if (timerRef.current) clearTimeout(timerRef.current);
  }

  return {
    onMouseDown: start,
    onMouseUp: stop,
    onMouseLeave: stop,
    onTouchStart: start,
    onTouchEnd: stop,
  };
}

// Usage
function HoldToDelete({ onDelete }) {
  const longPressHandlers = useLongPress(onDelete, 800);

  return (
    <button
      {...longPressHandlers}
      className="hold-to-delete"
    >
      Hold to Delete
    </button>
  );
}
```

### 12.3 Prevent Accidental Form Abandonment

```jsx
function useUnsavedChangesWarning(hasChanges) {
  useEffect(() => {
    if (!hasChanges) return;

    function handleBeforeUnload(e) {
      e.preventDefault();
      e.returnValue = ''; // Required for Chrome
    }

    window.addEventListener('beforeunload', handleBeforeUnload);
    return () => window.removeEventListener('beforeunload', handleBeforeUnload);
  }, [hasChanges]);
}

// Usage
function EditProfileForm() {
  const [form, setForm] = useState({ name: '', bio: '' });
  const [savedForm, setSavedForm] = useState({ name: '', bio: '' });

  const hasChanges = JSON.stringify(form) !== JSON.stringify(savedForm);

  useUnsavedChangesWarning(hasChanges);

  return (
    <form>
      {hasChanges && <div className="unsaved-badge">Unsaved changes</div>}
      {/* ... */}
    </form>
  );
}
```

---

## 13. Best Practices

### 13.1 Name Handlers with "handle" Prefix

```jsx
// ✅ Clear naming convention
function handleClick() { ... }
function handleSubmit() { ... }
function handleInputChange() { ... }
function handleModalClose() { ... }

// ❌ Unclear
function clicked() { ... }
function doSomething() { ... }
function fn1() { ... }
```

### 13.2 Always Prevent Default on Form Submissions

```jsx
// ✅ Always do this — prevent page reload
<form onSubmit={(e) => {
  e.preventDefault();
  handleSubmit();
}}>
```

### 13.3 Avoid Inline Arrow Functions in Performance-Critical Lists

```jsx
// ❌ New function created on every render — causes child re-renders
{items.map(item => (
  <Item key={item.id} onClick={() => handleSelect(item.id)} />
))}

// ✅ Memoize the handler or use data attributes
const handleSelect = useCallback((id) => {
  setSelected(id);
}, []);

{items.map(item => (
  <Item key={item.id} id={item.id} onClick={handleSelect} />
))}
```

### 13.4 Keep Handlers Small — Delegate to Helper Functions

```jsx
// ❌ Bloated inline handler
<button onClick={() => {
  if (!isLoggedIn) { navigate('/login'); return; }
  const newCart = [...cart, item];
  setCart(newCart);
  localStorage.setItem('cart', JSON.stringify(newCart));
  analytics.track('add_to_cart', { itemId: item.id });
  toast.success('Added to cart!');
}}>
  Add to Cart
</button>

// ✅ Clean delegation
function handleAddToCart() {
  if (!isLoggedIn) { navigate('/login'); return; }
  addItemToCart(item);
}
<button onClick={handleAddToCart}>Add to Cart</button>
```

### 13.5 Handle Keyboard Events for Accessibility

Interactive custom components (custom dropdowns, modals, tabs) must handle keyboard events. Every clickable element should also work with `Enter`, `Space`, and `Escape`.

### 13.6 Use `e.currentTarget` Not `e.target` for Handler Element

```jsx
// e.target — the element actually clicked (could be a child)
// e.currentTarget — the element the handler is attached to (always predictable)

<button onClick={(e) => {
  // If button contains an icon span, e.target might be the span
  // e.currentTarget is always the button
  const buttonId = e.currentTarget.dataset.id; // ✅ Safe
}}>
  <span>🛒</span> Add to Cart
</button>
```

---

## 14. Common Mistakes

### Mistake 1: Calling the Handler Instead of Passing It

```jsx
// ❌ handleClick() is CALLED during render, not on click
<button onClick={handleClick()}>Click</button>

// ✅ Pass the function reference
<button onClick={handleClick}>Click</button>

// ✅ If you need arguments, wrap in arrow function
<button onClick={() => handleDelete(item.id)}>Delete</button>
```

### Mistake 2: Forgetting e.preventDefault() on Forms

```jsx
// ❌ Page reloads on submit — all state is lost
<form onSubmit={handleSubmit}>...</form>

function handleSubmit() { /* no e.preventDefault() */ }

// ✅ Always prevent default
function handleSubmit(e) {
  e.preventDefault();
  // now process the form
}
```

### Mistake 3: Using e.target vs e.currentTarget Incorrectly

```jsx
// ❌ Dangerous — if button has children, e.target might be a child element
<button onClick={(e) => {
  const id = e.target.dataset.id; // might be undefined if a child was clicked
}}>
  <span className="icon">🗑️</span>
  Delete
</button>

// ✅ e.currentTarget is always the element with the handler
<button data-id={item.id} onClick={(e) => {
  const id = e.currentTarget.dataset.id; // always the button
}}>
```

### Mistake 4: Updating State Inside Event Without Functional Update

```jsx
// ❌ May use stale count value if multiple updates happen
function handleClick() {
  setCount(count + 1);
  setCount(count + 1); // Both see the same stale count!
}

// ✅ Functional update — always fresh
function handleClick() {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1); // Correctly increments twice
}
```

### Mistake 5: Not Removing Event Listeners (Memory Leak)

```jsx
// ❌ Listener is added on every render and never removed
function Component() {
  useEffect(() => {
    document.addEventListener('keydown', handleKeyDown);
    // No return cleanup!
  }, []);
}

// ✅ Always return a cleanup function
useEffect(() => {
  document.addEventListener('keydown', handleKeyDown);
  return () => document.removeEventListener('keydown', handleKeyDown);
}, []);
```

### Mistake 6: Using Deprecated Events

```jsx
// ❌ Deprecated
<input onKeyPress={handleKeyPress} />

// ✅ Use onKeyDown or onKeyUp
<input onKeyDown={handleKeyDown} />
```

### Mistake 7: Not Handling Both Mouse and Touch for Mobile

```jsx
// ❌ Only handles mouse — breaks on mobile/tablet
<div onMouseDown={handleStart} onMouseUp={handleEnd}>

// ✅ Handle both
<div
  onMouseDown={handleStart}
  onMouseUp={handleEnd}
  onTouchStart={handleStart}
  onTouchEnd={handleEnd}
>
```

---

## 15. Performance Considerations

### 15.1 useCallback for Stable Handler References

When passing handlers to memoized child components, wrap them in `useCallback`:

```jsx
const handleDelete = useCallback((id) => {
  setItems(prev => prev.filter(item => item.id !== id));
}, []); // no deps — setItems setter is stable

// Now MemoizedItem won't re-render when parent re-renders
<MemoizedItem onDelete={handleDelete} />
```

### 15.2 Debounce Handlers for Expensive Operations

```jsx
import { useMemo } from 'react';
import { debounce } from 'lodash';

function SearchInput({ onSearch }) {
  // Stable debounced handler
  const debouncedSearch = useMemo(
    () => debounce((query) => onSearch(query), 300),
    [onSearch]
  );

  // Cleanup debounce on unmount
  useEffect(() => {
    return () => debouncedSearch.cancel();
  }, [debouncedSearch]);

  return (
    <input
      onChange={e => debouncedSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### 15.3 Throttle for Scroll and Resize

```jsx
function useThrottledScroll(callback, delay = 100) {
  const lastRunRef = useRef(0);

  useEffect(() => {
    function handleScroll() {
      const now = Date.now();
      if (now - lastRunRef.current >= delay) {
        lastRunRef.current = now;
        callback();
      }
    }

    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, [callback, delay]);
}
```

### 15.4 Use Passive Event Listeners for Scroll Performance

```jsx
useEffect(() => {
  // ✅ passive: true — tells browser the handler won't call preventDefault()
  // Browser can optimize scrolling — up to 8x faster on mobile
  window.addEventListener('scroll', handleScroll, { passive: true });
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

---

## 16. Interview Questions

### Q1: What is a SyntheticEvent in React? How is it different from a native browser event?

**Answer:** A SyntheticEvent is React's cross-browser wrapper around the native browser event. It normalizes event properties and methods so they behave consistently across all browsers. You access it the same way as a native event — `event.target`, `event.preventDefault()`, etc. — but it's not the native event object. You can access the underlying native event via `event.nativeEvent`. In React 17+, event pooling was removed, so SyntheticEvents now behave like regular objects and you no longer need `event.persist()` for async code.

---

### Q2: What is event delegation and how does React use it?

**Answer:** Event delegation is attaching a single event listener to a parent element instead of individual listeners on each child. The parent intercepts events that bubble up from children. React uses this by attaching a single listener to the React root container (not individual DOM nodes). This means 1000 button components still only have one listener, making React apps memory-efficient. When you click a button, the event bubbles to the root, React identifies the component that should handle it, creates a SyntheticEvent, and calls your handler.

---

### Q3: What is the difference between a controlled and uncontrolled input?

**Answer:** A controlled input has its value driven by React state — `value` is set from state and `onChange` updates state on every keystroke. React is always the source of truth. An uncontrolled input stores its own value in the DOM — you use a `ref` to read the value when needed. Controlled inputs are recommended because they enable real-time validation, conditional formatting, and predictable behavior. Uncontrolled inputs are simpler for basic cases where you only need the value on submit.

---

### Q4: Why must you use `e.preventDefault()` in form onSubmit handlers?

**Answer:** By default, a form submission causes the browser to make a GET or POST request to the form's `action` URL, which reloads the page. In a React SPA, this would destroy all your component state and unmount the entire app. `e.preventDefault()` stops this default browser behavior, letting you handle the submission with JavaScript — validating data, calling APIs, updating state — without a page reload.

---

### Q5: What is the difference between `e.target` and `e.currentTarget`?

**Answer:** `e.target` is the element that **actually triggered** the event — the element the user directly clicked. `e.currentTarget` is the element **the event handler is attached to**. When event bubbling occurs, `e.target` stays the same (the original target) while `e.currentTarget` changes as the event moves up the tree. In practice: if a button contains a span icon and the user clicks the icon, `e.target` is the span, but `e.currentTarget` is the button (where your handler is). Always use `e.currentTarget` when you need to reliably reference the element your handler is on.

---

### Q6: How do you pass arguments to event handlers in React?

**Answer:** Event handlers receive only the event object by default. To pass additional data: (1) Use an inline arrow function that wraps the call: `onClick={() => handleDelete(item.id)}`; (2) Use a curried function that returns a handler: `onClick={handleDelete(item.id)}`; (3) Use `data-*` attributes on the element and read them from `e.currentTarget.dataset` in a single shared handler. For performance-sensitive lists, use `useCallback` with a stable handler that takes an id argument, and pass the id via a separate prop.

---

### Q7: What is event bubbling? How do you stop it in React?

**Answer:** Event bubbling is the propagation of an event from the target element upward through all ancestor elements in the DOM tree. For example, clicking a button inside a div fires handlers on both the button and the div. React supports this — handlers on parent elements receive events from their children by default. To stop an event from bubbling, call `e.stopPropagation()` in the handler. A common use case is clicking inside a modal to prevent the click from reaching the overlay's close handler.

---

### Q8: Why is using `useCallback` important for event handlers passed as props?

**Answer:** Every time a component re-renders, functions defined inside it are recreated — they get new references. If you pass these as props to a child wrapped in `React.memo`, the child sees new prop references on every parent render and re-renders unnecessarily, defeating memoization. `useCallback` memoizes the function — it returns the same function reference as long as its dependencies don't change. This makes `React.memo` effective and prevents unnecessary child re-renders.

---

### Q9: How do you implement keyboard accessibility in custom interactive components?

**Answer:** For any custom interactive element (dropdown, modal, tab panel, etc.): (1) Use appropriate semantic HTML or `role` attributes (`role="button"`, `role="listbox"`, etc.); (2) Add `tabIndex={0}` to make it focusable; (3) Handle `onKeyDown` for `Enter` (activate), `Space` (activate), `Escape` (close/cancel), `ArrowUp`/`ArrowDown` (navigate), `Home`/`End` (jump to first/last); (4) Use `aria-expanded`, `aria-selected`, `aria-controls` to communicate state to screen readers; (5) Manage focus programmatically with refs when the UI changes.

---

### Q10: What is the difference between onFocus/onBlur and native focus/blur in React?

**Answer:** The native DOM `focus` and `blur` events do not bubble — they only fire on the exact element that gains or loses focus. React's `onFocus` and `onBlur` do bubble — they fire on the element and all its ancestors. This is actually more aligned with the native `focusin` and `focusout` events. In React, this means you can listen for any focus event within a subtree by attaching `onFocus` to a parent div, which is useful for detecting when focus enters or leaves a complex widget like a dropdown or form section.

---

## 17. Practice Tasks

### Task 1 — Beginner: Interactive Color Picker

Build a `ColorPicker` component.

**Requirements:**
- Display a grid of color swatches (at least 12 colors)
- Track the selected color in state
- On click, update the selected color
- Show the selected color's hex code below the grid
- Show a preview box filled with the selected color
- Highlight the currently selected swatch with a border/checkmark
- On hover, show a tooltip with the color name
- Support keyboard navigation — arrow keys move between swatches, Enter selects

---

### Task 2 — Intermediate: Advanced Form with Validation

Build a `RegistrationForm` component.

**Requirements:**
- Fields: First Name, Last Name, Email, Password, Confirm Password, Date of Birth, Gender (radio), Country (select), Accept Terms (checkbox)
- Validate on submit AND on field blur
- Validation rules:
  - First/Last name: required, min 2 chars
  - Email: required, valid format
  - Password: required, min 8 chars, at least one number, one uppercase letter
  - Confirm password: must match password
  - Date of birth: required, user must be 18+
  - Country: required
  - Terms: must be checked
- Show field-level error messages with `role="alert"`
- Show a real-time password strength indicator (Weak / Medium / Strong)
- Disable submit button while submitting
- Show a success screen on successful submission
- Reset form on success with a "Register Another" button

---

### Task 3 — Advanced: Rich Text-Style Input with Mentions

Build a `MentionInput` component simulating a `@mention` feature.

**Requirements:**
- A textarea-like input where users type normally
- When the user types `@`, show a dropdown of users to mention
- Filter the user list as the user continues typing after `@`
- Clicking or pressing Enter on a suggestion inserts the mention and closes the dropdown
- Pressing Escape closes the dropdown without inserting
- Arrow Up/Down navigate the suggestion list
- Mentioned names are visually highlighted in the input display
- Track all mentioned users as a separate state array
- Show mentioned users as chips below the input with an X to remove each
- Close dropdown on click outside

---

## 18. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| Event handling | Responding to user actions — clicks, keystrokes, form inputs |
| SyntheticEvent | React's cross-browser event wrapper — behaves like native events |
| Event delegation | React attaches ONE listener to root, not per-element |
| camelCase events | `onClick`, `onChange`, `onKeyDown` — not lowercase like HTML |
| Handler syntax | Pass reference `onClick={fn}`, not call `onClick={fn()}` |
| e.preventDefault() | Stop default browser behavior — essential for forms |
| e.stopPropagation() | Stop event from bubbling to parent elements |
| e.target | The element that fired the event (could be a child) |
| e.currentTarget | The element your handler is attached to (always predictable) |
| Controlled input | React state drives the value — `value` + `onChange` |
| Uncontrolled input | DOM drives the value — accessed via `ref` |
| Keyboard events | Use `onKeyDown` — `onKeyPress` is deprecated |
| Accessibility | Handle `Enter`, `Space`, `Escape`, Arrow keys on custom components |
| useCallback | Memoize handlers passed to memoized children |
| Debounce/Throttle | Rate-limit expensive handlers (search, scroll, resize) |

### Event Handler Decision Tree

```
Do you need to handle a user interaction?
  ├── Is it a standard HTML element (button, input, form)?
  │     └── Use built-in React event props (onClick, onChange, onSubmit)
  │
  └── Is it a global event (window, document)?
        └── Use useEffect with addEventListener + cleanup
              ├── Fast-changing (scroll, resize, mousemove)?
              │     └── Debounce or throttle the handler
              └── One-time (keydown shortcut)?
                    └── Remove in cleanup function
```

---

> **Next Topic:** `07-conditional-rendering.md` — Mastering all patterns for conditionally showing and hiding UI in React: ternary operators, logical AND, early returns, switch statements, and component-based conditional strategies.
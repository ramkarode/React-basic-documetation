# ⚛️ React `useReducer` — Complete Beginner to Advanced Guide

> **A premium course-style documentation** that takes you from zero to production-ready understanding of React's `useReducer` hook.

---

## 📋 Table of Contents

1. [Introduction to State Management](#1-introduction-to-state-management)
2. [What is useReducer?](#2-what-is-usereducer)
3. [Understanding the Reducer Concept](#3-understanding-the-reducer-concept)
4. [Syntax of useReducer](#4-syntax-of-usereducer)
5. [First Beginner Example — Counter App](#5-first-beginner-example--counter-app)
6. [Understanding Actions](#6-understanding-actions)
7. [Multiple State Management](#7-multiple-state-management)
8. [useReducer vs useState](#8-usereducer-vs-usestate)
9. [Advanced Reducer Patterns](#9-advanced-reducer-patterns)
10. [useReducer with Context API](#10-usereducer-with-context-api)
11. [Async Operations with useReducer](#11-async-operations-with-usereducer)
12. [Real World Projects](#12-real-world-projects)
13. [Performance Optimization](#13-performance-optimization)
14. [Common Mistakes](#14-common-mistakes)
15. [Best Practices](#15-usereducer-best-practices)
16. [Redux vs useReducer](#16-redux-vs-usereducer)
17. [Interview Questions](#17-interview-questions)
18. [Practice Exercises](#18-practice-exercises)
19. [Mini Assignments](#19-mini-assignments)
20. [Final Summary & Cheat Sheet](#20-final-summary--cheat-sheet)

---

## 1. Introduction to State Management

### 🤔 What is State in React?

**State** is any data in your application that can change over time and, when it changes, should cause the UI to update.

Think of state like a **whiteboard in a classroom**:
- The whiteboard holds information (state)
- When the teacher erases and rewrites something (state update), everyone in the class sees the new information (UI re-render)

```jsx
// Simple example of state
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0); // 'count' is the state

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

### 🏗️ Why State Management Is Important

Without state management:
- You can't track user interactions (clicks, form inputs)
- Data doesn't persist across renders
- Components can't "remember" anything
- UI stays static (just HTML)

With proper state management:
- UI reflects the real-time truth of your data
- Components communicate meaningfully
- You can build complex, interactive applications

### ⚠️ Problems with Multiple `useState`

Imagine building a shopping cart with `useState` for everything:

```jsx
// ❌ This gets messy FAST
function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [totalPrice, setTotalPrice] = useState(0);
  const [itemCount, setItemCount] = useState(0);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [discount, setDiscount] = useState(0);
  const [couponApplied, setCouponApplied] = useState(false);
  const [shippingCost, setShippingCost] = useState(5.99);

  // Now imagine updating ALL of these when user adds one item...
  const addItem = (item) => {
    setItems([...items, item]);          // update items
    setItemCount(itemCount + 1);         // update count
    setTotalPrice(totalPrice + item.price); // update price
    // What if one setter fails? State becomes inconsistent!
  };
}
```

**Problems here:**
- 8 separate states that are all related
- They must stay in sync manually
- Easy to forget one setter
- Causes bugs where UI shows wrong data
- Code becomes hard to read and maintain

### 📉 When `useState` Becomes Difficult

`useState` struggles when:

| Situation | Why It's Painful |
|-----------|-----------------|
| Many related state variables | Must update all manually and keep them in sync |
| Complex update logic | Conditionals spread across multiple setters |
| State transitions depend on each other | Race conditions, stale closures |
| Same state updated from many places | Logic is duplicated everywhere |
| Debugging state issues | Hard to trace what changed and why |

### 🏦 Real-World Analogy for Reducers

Think of a **bank teller system**:

```
YOU (Customer) → SUBMIT FORM (Dispatch Action) → BANK TELLER (Reducer) → ACCOUNT UPDATED (New State)
```

- **You** want to deposit money → You fill out a form (action: `{ type: "DEPOSIT", amount: 500 }`)
- **You submit the form** to the teller (dispatch)
- **The teller** (reducer) looks at the form, verifies it, and updates your account
- **Your account balance** (state) is now updated
- The teller follows **bank rules** (pure function) — same deposit always adds the same amount

The teller doesn't decide randomly what to do. They follow strict rules. That's exactly what a reducer does.

### 🌐 Local State vs Global State

```
LOCAL STATE                         GLOBAL STATE
───────────────                     ────────────────────
Lives in one component              Lives outside components
Passed down via props               Accessible from anywhere
Good for isolated UI                Good for shared data
Example: modal open/close           Example: logged-in user, cart
```

---

## 2. What is `useReducer`?

### 📖 Definition

`useReducer` is a React Hook that lets you manage state using a **reducer function** — a pure function that takes the current state and an action, and returns the new state.

```
useReducer = useState + Logic Centralization + Predictable Updates
```

### 🏛️ Why React Introduced It

React introduced `useReducer` because:

1. Complex state logic was scattered across multiple `useState` calls
2. Related state wasn't grouped together
3. State transitions were hard to understand and debug
4. Developers were repeating update logic in multiple places

`useReducer` was inspired by **Redux** (a popular state management library) and the **Flux architecture** pattern from Facebook.

### ✅ When to Use `useReducer`

Use `useReducer` when:

- ✅ You have **3+ related state variables**
- ✅ The next state **depends on the previous state**
- ✅ State transitions are **complex** (multiple operations per action)
- ✅ You want to **centralize update logic** in one place
- ✅ You're building features like: todo list, shopping cart, auth system, multi-step form
- ✅ Multiple actions can update the same piece of state

### 🆚 Advantages Over `useState`

| Advantage | Description |
|-----------|-------------|
| **Centralized logic** | All update logic in one reducer function |
| **Predictable** | Given same state + action = always same result |
| **Debuggable** | You can log every action and see state history |
| **Testable** | Pure functions are easy to unit test |
| **Scalable** | Adding new behavior = adding a new case |
| **Separation of concerns** | Logic is separated from UI |

### 💡 Real-Life Examples Where Reducers Are Useful

1. **Authentication System** — login, logout, token refresh, error states
2. **Shopping Cart** — add item, remove item, update quantity, apply coupon, checkout
3. **Todo App** — add, delete, toggle, filter, clear completed
4. **Multi-step Form Wizard** — next step, prev step, save data, submit
5. **Video Player** — play, pause, seek, volume, fullscreen
6. **API Data Fetching** — loading, success, error, cache

### 🔄 Flow Explanation

```
┌─────────────────────────────────────────────────────────┐
│                    useReducer Flow                       │
│                                                         │
│   User clicks button                                    │
│          │                                              │
│          ▼                                              │
│   dispatch({ type: "INCREMENT" })   ← YOU call this    │
│          │                                              │
│          ▼                                              │
│   reducer(currentState, action)     ← React calls this  │
│          │                                              │
│          ▼                                              │
│   Returns new state object                              │
│          │                                              │
│          ▼                                              │
│   React re-renders with new state   ← UI updates       │
└─────────────────────────────────────────────────────────┘
```

More precisely:

```
State ──────→ [Displayed in UI]
  ↑                  │
  │                  │ User Action (click, type, submit)
  │                  ▼
  │          dispatch({ type, payload })
  │                  │
  │                  ▼
  └──────── reducer(state, action)
              returns NEW state
```

---

## 3. Understanding the Reducer Concept

### 🧮 What is a Reducer Function?

A reducer is a function that:
1. Takes **two inputs**: current state + action
2. Returns **one output**: new state
3. Is **pure**: no side effects, no API calls, no randomness
4. Is **predictable**: same inputs → always same output

```javascript
// Simplest possible reducer
function reducer(state, action) {
  // Based on the action, decide what new state should be
  // Return the new state
}
```

### 🧼 Pure Functions

A **pure function** is a function that:
- Always returns the same output for the same inputs
- Has no side effects (doesn't modify external variables, make API calls, etc.)

```javascript
// ✅ PURE FUNCTION
function add(a, b) {
  return a + b; // always same result for same inputs
}

// ❌ NOT PURE (has side effect)
let total = 0;
function addToTotal(amount) {
  total += amount; // modifies external variable!
  return total;
}

// ❌ NOT PURE (random output)
function randomGreeting(name) {
  const greeting = Math.random() > 0.5 ? "Hello" : "Hi";
  return `${greeting}, ${name}!`; // different result each time!
}
```

**Reducers must be pure functions.** This is what makes state predictable.

### 🔒 Immutable Updates

**Immutability** means you never modify the existing state object. Instead, you create a NEW state object.

```javascript
// ❌ WRONG - mutating state directly
function badReducer(state, action) {
  if (action.type === "ADD_ITEM") {
    state.items.push(action.payload); // MUTATING! React won't detect this change!
    return state;
  }
}

// ✅ CORRECT - creating new state
function goodReducer(state, action) {
  if (action.type === "ADD_ITEM") {
    return {
      ...state,                         // copy all existing properties
      items: [...state.items, action.payload], // create NEW array with new item
    };
  }
}
```

### 💥 Why Mutation is Bad in React

React uses **reference equality** to detect changes:

```javascript
const obj1 = { count: 0 };
const obj2 = obj1; // same reference
obj2.count = 1;    // mutate

console.log(obj1 === obj2); // true! React thinks nothing changed!
console.log(obj1.count);    // 1 (but React won't re-render)

// This is why mutation = invisible changes = bugs!
```

```javascript
const obj1 = { count: 0 };
const obj3 = { ...obj1, count: 1 }; // new object

console.log(obj1 === obj3); // false! React knows something changed!
// React will re-render ✅
```

### 🧮 JavaScript Reducer Example (Plain JS, No React)

```javascript
// Think of this like Array.reduce()
// accumulator = state, currentValue = action
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((accumulator, currentValue) => {
  return accumulator + currentValue;
}, 0); // 0 is the initial value (like initialState)

console.log(sum); // 15

// Same concept for state:
const initialState = { count: 0 };

const actions = [
  { type: "INCREMENT" },
  { type: "INCREMENT" },
  { type: "DECREMENT" },
  { type: "INCREMENT" },
];

function counterReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Manually "reducing" through actions (like replay)
const finalState = actions.reduce(counterReducer, initialState);
console.log(finalState); // { count: 2 }
```

### 🗺️ State Transition Diagram

```
                    ┌──────────────────┐
                    │   initialState   │
                    │   { count: 0 }   │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
         INCREMENT       DECREMENT        RESET
              │              │              │
              ▼              ▼              ▼
       { count: 1 }   { count: -1 }   { count: 0 }
              │
         INCREMENT
              │
              ▼
       { count: 2 }
              │
         DECREMENT
              │
              ▼
       { count: 1 }
```

---

## 4. Syntax of `useReducer`

### 📝 The Complete Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

Let's break every single part:

---

### 🔧 Part 1: `reducer`

The **reducer** is a function you define. It receives the current state and an action, and returns new state.

```javascript
// YOU define this function
function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    default:
      return state; // ALWAYS have a default case!
  }
}
```

**Rules for the reducer:**
- Must be a **pure function**
- Must **always return** a state (don't forget your default case!)
- Must **not mutate** the existing state
- Must be defined **outside** the component (or it gets recreated every render)

---

### 📦 Part 2: `initialState`

The **initial state** is the starting value of your state. It can be any JavaScript value:

```javascript
// Simple value
const initialState = 0;

// Object (most common)
const initialState = {
  count: 0,
  name: "",
  isLoading: false,
};

// Array
const initialState = [];

// Complex nested object
const initialState = {
  user: null,
  todos: [],
  settings: {
    theme: "light",
    language: "en",
  },
};
```

---

### 🗃️ Part 3: `state`

The **state** variable holds the current state. It's what you read in your UI.

```jsx
const [state, dispatch] = useReducer(reducer, { count: 0 });

// Now you can use state.count in your JSX
return <p>Count: {state.count}</p>;
```

---

### 📡 Part 4: `dispatch`

**dispatch** is a function that sends actions to the reducer. When you call `dispatch`, React:
1. Calls your `reducer(currentState, action)`
2. Gets the new state from the reducer
3. Re-renders the component with the new state

```javascript
// dispatch takes an action object
dispatch({ type: "INCREMENT" });
dispatch({ type: "ADD_TODO", payload: "Buy groceries" });
dispatch({ type: "SET_USER", payload: { id: 1, name: "Alice" } });
```

**Important:** `dispatch` is stable — its identity doesn't change between renders, so it's safe to put in dependency arrays.

---

### 📨 Part 5: `action`

An **action** is a plain JavaScript object that describes *what happened* or *what you want to do*.

```javascript
// Minimal action (no data needed)
{ type: "INCREMENT" }

// Action with data
{ type: "ADD_TODO", payload: "Buy groceries" }

// Action with multiple data points
{ type: "UPDATE_USER", payload: { name: "Alice", age: 25 } }
```

**Convention:** Actions always have a `type` property (string). Additional data goes in `payload`.

---

### 🏷️ Part 6: `action.type`

The `type` is a **string** that identifies what kind of action this is. The reducer uses it in a `switch` statement to decide what to do.

```javascript
// Common naming conventions:
"INCREMENT"           // SCREAMING_SNAKE_CASE (most common)
"ADD_TODO"
"SET_LOADING"
"FETCH_USER_SUCCESS"

// Some teams use:
"todos/add"           // Redux Toolkit style
"counter/increment"
```

---

### 📦 Part 7: `payload`

The **payload** is the data that comes with an action. Not all actions need a payload.

```javascript
// No payload needed — the action type says it all
dispatch({ type: "INCREMENT" });
dispatch({ type: "RESET" });
dispatch({ type: "TOGGLE_DARK_MODE" });

// Payload is the data the reducer needs to do its work
dispatch({ type: "ADD_TODO", payload: "Buy groceries" });
dispatch({ type: "DELETE_TODO", payload: 3 }); // id to delete
dispatch({ type: "SET_FILTER", payload: "completed" });
dispatch({ type: "UPDATE_ITEM", payload: { id: 2, quantity: 5 } });
```

---

### 🔭 Complete Syntax Diagram

```
const [  state  ,  dispatch  ] = useReducer(  reducer  ,  initialState  );
         │              │                      │              │
         │              │                      │              └── Starting value
         │              │                      │                  of state
         │              │                      │
         │              │                      └── Your pure function:
         │              │                          reducer(state, action) => newState
         │              │
         │              └── Function to send actions:
         │                  dispatch({ type: "ACTION_TYPE", payload: data })
         │
         └── Current state value (what you display in UI)
```

---

## 5. First Beginner Example — Counter App

### 🎯 Goal

Build a counter with:
- ➕ Increment (add 1)
- ➖ Decrement (subtract 1)
- 🔄 Reset (go back to 0)

### 📂 Complete Code

```jsx
// Counter.jsx
import { useReducer } from "react";

// ─────────────────────────────────────────────
// STEP 1: Define initial state
// This is what the state looks like at the start
// ─────────────────────────────────────────────
const initialState = {
  count: 0,
};

// ─────────────────────────────────────────────
// STEP 2: Define the reducer function
// This function DECIDES how state changes
// It lives OUTSIDE the component (important!)
// ─────────────────────────────────────────────
function counterReducer(state, action) {
  // 'state' = current state (e.g., { count: 0 })
  // 'action' = what happened (e.g., { type: "INCREMENT" })

  switch (action.type) {
    case "INCREMENT":
      // Return a NEW state object with count increased by 1
      return { count: state.count + 1 };

    case "DECREMENT":
      // Return a NEW state object with count decreased by 1
      return { count: state.count - 1 };

    case "RESET":
      // Return the initial state (count = 0)
      return { count: 0 };

    default:
      // ALWAYS handle unknown actions by returning current state
      // This prevents bugs from typos in action.type
      return state;
  }
}

// ─────────────────────────────────────────────
// STEP 3: The Component
// ─────────────────────────────────────────────
function Counter() {
  // Connect state and dispatch to our reducer
  // state = { count: 0 } initially
  // dispatch = function to send actions
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div style={{ textAlign: "center", padding: "40px" }}>
      <h1>Counter: {state.count}</h1>

      {/* Each button dispatches an action to the reducer */}
      <button onClick={() => dispatch({ type: "INCREMENT" })}>
        ➕ Increment
      </button>

      <button onClick={() => dispatch({ type: "DECREMENT" })}>
        ➖ Decrement
      </button>

      <button onClick={() => dispatch({ type: "RESET" })}>
        🔄 Reset
      </button>
    </div>
  );
}

export default Counter;
```

### 🔍 Step-by-Step Flow: What Happens Internally?

Let's trace through clicking "Increment":

```
Step 1: User clicks "➕ Increment" button

Step 2: onClick fires:
        dispatch({ type: "INCREMENT" })

Step 3: React takes your action { type: "INCREMENT" }
        and calls: counterReducer({ count: 0 }, { type: "INCREMENT" })

Step 4: Inside the reducer:
        switch("INCREMENT") → case "INCREMENT":
        return { count: 0 + 1 }
        → returns { count: 1 }

Step 5: React receives new state { count: 1 }
        React compares { count: 1 } !== { count: 0 } → DIFFERENT!
        React schedules a re-render

Step 6: Component re-renders with state = { count: 1 }
        <h1>Counter: 1</h1> appears on screen
```

### 📊 State After Each Click

| Action Dispatched | Previous State | New State |
|------------------|---------------|-----------|
| (initial) | — | `{ count: 0 }` |
| `INCREMENT` | `{ count: 0 }` | `{ count: 1 }` |
| `INCREMENT` | `{ count: 1 }` | `{ count: 2 }` |
| `DECREMENT` | `{ count: 2 }` | `{ count: 1 }` |
| `RESET` | `{ count: 1 }` | `{ count: 0 }` |

---

## 6. Understanding Actions

### 📨 What is an Action Object?

An action is a **plain JavaScript object** that describes what event occurred in your application. It's the "message" you send to the reducer.

```javascript
// An action is just a plain object
const action = {
  type: "ADD_TO_CART",   // Required: identifies the action
  payload: {             // Optional: carries data
    id: 42,
    name: "Laptop",
    price: 999.99,
  },
};
```

### 🏷️ Why `action.type` Exists

The `type` property is the **identifier** of the action. The reducer uses it to decide *what to do*.

Think of it like a **command name**:
- `"INCREMENT"` → add 1
- `"ADD_TODO"` → add a new todo
- `"LOGIN_SUCCESS"` → store user info and mark as authenticated

```javascript
// Without type, reducer has no idea what to do
dispatch({ payload: "buy milk" }); // ❌ reducer: "...what am I supposed to do?"

// With type, it's crystal clear
dispatch({ type: "ADD_TODO", payload: "Buy milk" }); // ✅ clear intent
```

### 📦 Payload Concept

`payload` is the **data** your action carries. Not all actions need payload.

```javascript
// Actions WITHOUT payload (the type says everything)
dispatch({ type: "INCREMENT" });
dispatch({ type: "LOGOUT" });
dispatch({ type: "CLEAR_CART" });
dispatch({ type: "TOGGLE_MENU" });

// Actions WITH payload (need additional data)
dispatch({ type: "SET_USER",       payload: { id: 1, name: "Alice" } });
dispatch({ type: "ADD_TODO",       payload: "Walk the dog" });
dispatch({ type: "REMOVE_TODO",    payload: 5 }); // the ID to remove
dispatch({ type: "SET_QUANTITY",   payload: { itemId: 3, qty: 2 } });
dispatch({ type: "SET_ERROR",      payload: "Something went wrong" });
```

### ⚙️ Multiple Actions Handling

```jsx
// reducer handling many action types
function todoReducer(state, action) {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),          // unique id
            text: action.payload,    // todo text from payload
            completed: false,
          },
        ],
      };

    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          // If this is the todo to toggle, flip its 'completed'
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo // leave other todos unchanged
        ),
      };

    case "DELETE_TODO":
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };

    case "CLEAR_COMPLETED":
      return {
        ...state,
        todos: state.todos.filter((todo) => !todo.completed),
      };

    default:
      return state;
  }
}
```

### ✅ Best Practices for Actions

```javascript
// ✅ 1. Use SCREAMING_SNAKE_CASE for action types
dispatch({ type: "ADD_USER" });
dispatch({ type: "FETCH_PRODUCTS_SUCCESS" });

// ✅ 2. Use descriptive names — say WHAT happened, not HOW
dispatch({ type: "USER_LOGGED_IN" });      // Good: describes event
dispatch({ type: "SET_AUTH_TRUE" });       // Bad: describes implementation

// ✅ 3. Keep payload minimal — only what the reducer needs
dispatch({ type: "DELETE_ITEM", payload: item.id }); // ✅ just the ID

// ❌ Don't send the whole item if you only need the ID
dispatch({ type: "DELETE_ITEM", payload: item });     // ❌ unnecessary data

// ✅ 4. Use action creator functions for reusability
const addTodo = (text) => ({ type: "ADD_TODO", payload: text });
const deleteTodo = (id) => ({ type: "DELETE_TODO", payload: id });

dispatch(addTodo("Buy milk"));  // clean and reusable

// ✅ 5. Use constants for action types (prevents typos)
const ACTIONS = {
  ADD_TODO: "ADD_TODO",
  DELETE_TODO: "DELETE_TODO",
  TOGGLE_TODO: "TOGGLE_TODO",
};

dispatch({ type: ACTIONS.ADD_TODO, payload: "Walk dog" });
// If you typo ACTIONS.ADD_TOOD → undefined → caught immediately!
```

---

## 7. Multiple State Management

### 📦 Managing Object State

When state is an object, use the spread operator to create immutable updates:

```jsx
// formReducer.jsx
import { useReducer } from "react";

const initialState = {
  firstName: "",
  lastName: "",
  email: "",
  password: "",
  errors: {},
  isSubmitting: false,
};

function formReducer(state, action) {
  switch (action.type) {
    case "UPDATE_FIELD":
      return {
        ...state, // keep all existing fields
        [action.payload.field]: action.payload.value, // update only this field
        errors: { ...state.errors, [action.payload.field]: "" }, // clear that field's error
      };

    case "SET_ERRORS":
      return {
        ...state,
        errors: action.payload,
      };

    case "SET_SUBMITTING":
      return {
        ...state,
        isSubmitting: action.payload,
      };

    case "RESET_FORM":
      return initialState;

    default:
      return state;
  }
}

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  // Generic handler — works for any field!
  const handleChange = (e) => {
    dispatch({
      type: "UPDATE_FIELD",
      payload: { field: e.target.name, value: e.target.value },
    });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    dispatch({ type: "SET_SUBMITTING", payload: true });

    // Validate
    const errors = {};
    if (!state.email.includes("@")) errors.email = "Invalid email";
    if (state.password.length < 6) errors.password = "Too short";

    if (Object.keys(errors).length > 0) {
      dispatch({ type: "SET_ERRORS", payload: errors });
      dispatch({ type: "SET_SUBMITTING", payload: false });
      return;
    }

    // Submit (pretend API call)
    await new Promise((r) => setTimeout(r, 1000));
    alert("Submitted!");
    dispatch({ type: "RESET_FORM" });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="firstName"
        value={state.firstName}
        onChange={handleChange}
        placeholder="First Name"
      />
      <input
        name="email"
        value={state.email}
        onChange={handleChange}
        placeholder="Email"
      />
      {state.errors.email && <span style={{color:"red"}}>{state.errors.email}</span>}
      <input
        name="password"
        type="password"
        value={state.password}
        onChange={handleChange}
        placeholder="Password"
      />
      {state.errors.password && <span style={{color:"red"}}>{state.errors.password}</span>}
      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? "Submitting..." : "Register"}
      </button>
    </form>
  );
}
```

### 📋 Managing Array State

The key rules for arrays (never mutate!):

```javascript
// ─────────────────────────────────────────────────────
// ARRAY OPERATIONS — IMMUTABLE PATTERNS
// ─────────────────────────────────────────────────────

// ➕ ADD item
case "ADD_TODO":
  return {
    ...state,
    todos: [...state.todos, newItem], // spread existing + add new
  };

// ❌ REMOVE item by id
case "DELETE_TODO":
  return {
    ...state,
    todos: state.todos.filter((todo) => todo.id !== action.payload),
    // filter creates a new array without the item
  };

// ✏️ UPDATE item
case "UPDATE_TODO":
  return {
    ...state,
    todos: state.todos.map((todo) =>
      todo.id === action.payload.id
        ? { ...todo, ...action.payload.updates } // create new object with updates
        : todo // unchanged items stay the same
    ),
  };

// 🔀 MOVE item (reorder)
case "MOVE_UP":
  const index = state.todos.findIndex((t) => t.id === action.payload);
  if (index <= 0) return state; // already at top
  const newTodos = [...state.todos];
  // Swap with previous item
  [newTodos[index - 1], newTodos[index]] = [newTodos[index], newTodos[index - 1]];
  return { ...state, todos: newTodos };
```

### 🗺️ Managing Nested State

Nested state requires careful spreading at each level:

```javascript
// State with nesting
const initialState = {
  user: {
    profile: {
      name: "Alice",
      address: {
        city: "Mumbai",
        pincode: "400001",
      },
    },
    preferences: {
      theme: "light",
      notifications: true,
    },
  },
};

function reducer(state, action) {
  switch (action.type) {
    // Updating deeply nested: user.profile.address.city
    case "UPDATE_CITY":
      return {
        ...state,               // spread top level
        user: {
          ...state.user,        // spread user level
          profile: {
            ...state.user.profile, // spread profile level
            address: {
              ...state.user.profile.address, // spread address level
              city: action.payload, // update only city
            },
          },
        },
      };

    case "TOGGLE_THEME":
      return {
        ...state,
        user: {
          ...state.user,
          preferences: {
            ...state.user.preferences,
            theme: state.user.preferences.theme === "light" ? "dark" : "light",
          },
        },
      };

    default:
      return state;
  }
}

// 💡 TIP: For very deep nesting, consider using Immer library
// which lets you write "mutating" code that's actually immutable:
// import produce from 'immer';
// case "UPDATE_CITY":
//   return produce(state, draft => {
//     draft.user.profile.address.city = action.payload;
//   });
```

### 🛒 Shopping Cart Example

```jsx
// ShoppingCart.jsx
import { useReducer } from "react";

const initialState = {
  items: [],       // array of { id, name, price, quantity }
  total: 0,        // total price
  itemCount: 0,    // total number of items
};

// Helper: calculate total from items array
const calculateTotal = (items) =>
  items.reduce((sum, item) => sum + item.price * item.quantity, 0);

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      // Check if item already exists
      const existingItem = state.items.find((item) => item.id === action.payload.id);

      let updatedItems;
      if (existingItem) {
        // Item exists → just increase quantity
        updatedItems = state.items.map((item) =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        // New item → add with quantity 1
        updatedItems = [...state.items, { ...action.payload, quantity: 1 }];
      }

      return {
        ...state,
        items: updatedItems,
        total: calculateTotal(updatedItems),
        itemCount: state.itemCount + 1,
      };
    }

    case "REMOVE_ITEM": {
      const updatedItems = state.items.filter((item) => item.id !== action.payload);
      const removedItem = state.items.find((item) => item.id === action.payload);

      return {
        ...state,
        items: updatedItems,
        total: calculateTotal(updatedItems),
        itemCount: state.itemCount - (removedItem ? removedItem.quantity : 0),
      };
    }

    case "UPDATE_QUANTITY": {
      const { id, quantity } = action.payload;
      if (quantity <= 0) {
        // If quantity is 0, remove the item
        return cartReducer(state, { type: "REMOVE_ITEM", payload: id });
      }

      const updatedItems = state.items.map((item) =>
        item.id === id ? { ...item, quantity } : item
      );

      return {
        ...state,
        items: updatedItems,
        total: calculateTotal(updatedItems),
        itemCount: updatedItems.reduce((sum, item) => sum + item.quantity, 0),
      };
    }

    case "CLEAR_CART":
      return initialState;

    default:
      return state;
  }
}

function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, initialState);

  const sampleProduct = { id: 1, name: "Laptop", price: 999.99 };

  return (
    <div>
      <button onClick={() => dispatch({ type: "ADD_ITEM", payload: sampleProduct })}>
        Add Laptop to Cart
      </button>

      <h3>Cart ({cart.itemCount} items)</h3>
      {cart.items.map((item) => (
        <div key={item.id}>
          <span>{item.name} × {item.quantity} = ${(item.price * item.quantity).toFixed(2)}</span>
          <button onClick={() => dispatch({ type: "UPDATE_QUANTITY", payload: { id: item.id, quantity: item.quantity - 1 }})}>
            -
          </button>
          <button onClick={() => dispatch({ type: "UPDATE_QUANTITY", payload: { id: item.id, quantity: item.quantity + 1 }})}>
            +
          </button>
          <button onClick={() => dispatch({ type: "REMOVE_ITEM", payload: item.id })}>
            Remove
          </button>
        </div>
      ))}

      <h3>Total: ${cart.total.toFixed(2)}</h3>
      <button onClick={() => dispatch({ type: "CLEAR_CART" })}>Clear Cart</button>
    </div>
  );
}

export default ShoppingCart;
```

---

## 8. `useReducer` vs `useState`

### 📊 Detailed Comparison Table

| Factor | `useState` | `useReducer` |
|--------|-----------|--------------|
| **Best for** | Simple, independent values | Complex, related state |
| **Syntax** | `const [x, setX] = useState(0)` | `const [state, dispatch] = useReducer(fn, {})` |
| **Update mechanism** | `setX(newValue)` | `dispatch({ type: "ACTION" })` |
| **Logic location** | Scattered in event handlers | Centralized in reducer |
| **Readability** | Simple for 1-2 states | Cleaner when many states |
| **Debugging** | Hard to trace across handlers | Easy — log all dispatched actions |
| **Testing** | Test component behavior | Test pure reducer function directly |
| **Performance** | Same | Same (tiny overhead on first render) |
| **Learning curve** | Easy | Moderate |
| **Scalability** | Doesn't scale well | Scales very well |
| **Related state** | Possible but messy | Natural fit |

### 🤔 When to Choose Which

```
CHOOSE useState WHEN:
━━━━━━━━━━━━━━━━━━━━━━
✅ Boolean toggle: const [isOpen, setIsOpen] = useState(false)
✅ Single counter: const [count, setCount] = useState(0)
✅ Single text input: const [name, setName] = useState("")
✅ Simple show/hide: const [visible, setVisible] = useState(true)
✅ Independent values that don't affect each other

CHOOSE useReducer WHEN:
━━━━━━━━━━━━━━━━━━━━━━━
✅ 3+ related state values that belong together
✅ Next state depends on previous state
✅ Multiple actions update the same state
✅ Complex logic (conditions, calculations) in updates
✅ You want to log/track all state changes
✅ State is an object or array being updated in complex ways
✅ Building: forms, carts, auth systems, data tables
```

### 💻 Side-by-Side Code Comparison

```jsx
// SAME FEATURE, different approaches

// ──────────────────────────────────
// useState approach (gets messy)
// ──────────────────────────────────
function TodoWithUseState() {
  const [todos, setTodos] = useState([]);
  const [inputText, setInputText] = useState("");
  const [filter, setFilter] = useState("all");
  const [isLoading, setIsLoading] = useState(false);

  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: inputText, done: false }]);
    setInputText("");
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(t => t.id === id ? { ...t, done: !t.done } : t));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(t => t.id !== id));
  };
  // Logic is scattered everywhere...
}

// ──────────────────────────────────
// useReducer approach (clean!)
// ──────────────────────────────────
const initialState = {
  todos: [],
  inputText: "",
  filter: "all",
  isLoading: false,
};

function todoReducer(state, action) {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), text: state.inputText, done: false }],
        inputText: "",
      };
    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map(t =>
          t.id === action.payload ? { ...t, done: !t.done } : t
        ),
      };
    case "DELETE_TODO":
      return { ...state, todos: state.todos.filter(t => t.id !== action.payload) };
    case "SET_INPUT":
      return { ...state, inputText: action.payload };
    case "SET_FILTER":
      return { ...state, filter: action.payload };
    default:
      return state;
  }
}

function TodoWithUseReducer() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  // All logic is in the reducer — component is clean!
}
```

---

## 9. Advanced Reducer Patterns

### 🔀 Splitting Reducers

When your reducer gets large, split it by feature/concern:

```javascript
// Instead of one giant reducer handling everything...

// ────────────────────────────────
// todosReducer.js
// ────────────────────────────────
export function todosReducer(state = [], action) {
  switch (action.type) {
    case "ADD_TODO":
      return [...state, { id: Date.now(), text: action.payload, done: false }];
    case "TOGGLE_TODO":
      return state.map(t => t.id === action.payload ? { ...t, done: !t.done } : t);
    case "DELETE_TODO":
      return state.filter(t => t.id !== action.payload);
    default:
      return state;
  }
}

// ────────────────────────────────
// filterReducer.js
// ────────────────────────────────
export function filterReducer(state = "all", action) {
  switch (action.type) {
    case "SET_FILTER":
      return action.payload;
    default:
      return state;
  }
}

// ────────────────────────────────
// uiReducer.js
// ────────────────────────────────
export function uiReducer(state = { isLoading: false, error: null }, action) {
  switch (action.type) {
    case "SET_LOADING":
      return { ...state, isLoading: action.payload };
    case "SET_ERROR":
      return { ...state, error: action.payload };
    default:
      return state;
  }
}
```

### 🧩 Combining Reducers

Combine split reducers into one root reducer:

```javascript
// rootReducer.js
import { todosReducer } from "./todosReducer";
import { filterReducer } from "./filterReducer";
import { uiReducer } from "./uiReducer";

// Manual combineReducers
function rootReducer(state = {}, action) {
  return {
    todos: todosReducer(state.todos, action),
    filter: filterReducer(state.filter, action),
    ui: uiReducer(state.ui, action),
  };
}

// Or build a helper (like Redux's combineReducers)
function combineReducers(reducers) {
  return function (state = {}, action) {
    return Object.keys(reducers).reduce((nextState, key) => {
      nextState[key] = reducers[key](state[key], action);
      return nextState;
    }, {});
  };
}

const rootReducer = combineReducers({
  todos: todosReducer,
  filter: filterReducer,
  ui: uiReducer,
});

// Usage:
const [state, dispatch] = useReducer(rootReducer, {});
// state.todos, state.filter, state.ui
```

### 🏭 Action Creators

Action creators are functions that return action objects. They make your code reusable and reduce typos:

```javascript
// actionCreators.js

// Simple action creators
export const increment = () => ({ type: "INCREMENT" });
export const decrement = () => ({ type: "DECREMENT" });
export const reset = () => ({ type: "RESET" });

// Action creators with parameters
export const addTodo = (text) => ({
  type: "ADD_TODO",
  payload: text,
});

export const deleteTodo = (id) => ({
  type: "DELETE_TODO",
  payload: id,
});

export const updateQuantity = (id, quantity) => ({
  type: "UPDATE_QUANTITY",
  payload: { id, quantity },
});

// Usage in component:
dispatch(addTodo("Buy groceries"));   // ✅ clean
dispatch(deleteTodo(3));              // ✅ clean
dispatch(updateQuantity(5, 2));       // ✅ clean

// vs manually:
dispatch({ type: "ADD_TODO", payload: "Buy groceries" });  // repetitive
```

### 🔄 Reducer Composition

Reducers can call each other:

```javascript
// cartReducer.js
function cartReducer(state, action) {
  switch (action.type) {
    case "REMOVE_ITEM": {
      // Reuse logic from within the same reducer
      const quantity = state.items.find(i => i.id === action.payload)?.quantity || 0;
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload),
        itemCount: state.itemCount - quantity,
        total: state.total - calculateItemTotal(state, action.payload),
      };
    }

    case "CLEAR_CART":
      return initialState;

    case "UPDATE_QUANTITY": {
      const { id, quantity } = action.payload;
      // Reuse REMOVE_ITEM logic when quantity reaches 0
      if (quantity <= 0) {
        return cartReducer(state, { type: "REMOVE_ITEM", payload: id });
      }
      // ... rest of update logic
    }

    default:
      return state;
  }
}
```

### 🏷️ Action Constants Pattern

```javascript
// actionTypes.js — define all action types as constants
export const COUNTER_ACTIONS = {
  INCREMENT: "INCREMENT",
  DECREMENT: "DECREMENT",
  RESET: "RESET",
  SET: "SET",
};

export const TODO_ACTIONS = {
  ADD: "ADD_TODO",
  DELETE: "DELETE_TODO",
  TOGGLE: "TOGGLE_TODO",
  CLEAR_COMPLETED: "CLEAR_COMPLETED",
};

export const AUTH_ACTIONS = {
  LOGIN: "LOGIN",
  LOGOUT: "LOGOUT",
  REFRESH_TOKEN: "REFRESH_TOKEN",
  SET_ERROR: "AUTH_SET_ERROR",
};

// Usage — typos become obvious immediately
dispatch({ type: COUNTER_ACTIONS.INCREEMENT }); // ❌ undefined → caught!
dispatch({ type: COUNTER_ACTIONS.INCREMENT });   // ✅
```

---

## 10. `useReducer` with Context API

### 🌐 Why Combine useReducer + Context?

`useReducer` manages state; `Context` shares it. Together, they create a lightweight global state solution without Redux.

```
Without Context:
Component A (has state)
  └── Component B (needs state, gets via props)
        └── Component C (needs state, gets via props)
              └── Component D (needs state, gets via props)
              // Props drilling through 4 levels = PAIN

With Context + useReducer:
Context Provider (wraps everything)
  ├── Component A
  ├── Component B (reads from context directly)
  ├── Component C (reads from context directly)
  └── Component D (reads from context directly)
  // Any component can access state and dispatch!
```

### 🎨 Theme Switcher with Context

```jsx
// theme/ThemeContext.jsx
import { createContext, useContext, useReducer } from "react";

// 1. Create the context
const ThemeContext = createContext(null);

// 2. Initial state and reducer
const initialThemeState = {
  theme: "light",  // "light" | "dark"
  primaryColor: "#3b82f6",
  fontSize: "medium",
};

function themeReducer(state, action) {
  switch (action.type) {
    case "TOGGLE_THEME":
      return { ...state, theme: state.theme === "light" ? "dark" : "light" };
    case "SET_PRIMARY_COLOR":
      return { ...state, primaryColor: action.payload };
    case "SET_FONT_SIZE":
      return { ...state, fontSize: action.payload };
    default:
      return state;
  }
}

// 3. Provider component (wraps your app)
export function ThemeProvider({ children }) {
  const [themeState, dispatch] = useReducer(themeReducer, initialThemeState);

  return (
    // Pass both state AND dispatch through context
    <ThemeContext.Provider value={{ themeState, dispatch }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 4. Custom hook for easy access
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used inside <ThemeProvider>");
  }
  return context;
}

// ──────────────────────────────────────────
// App.jsx — wrap your app with the provider
// ──────────────────────────────────────────
import { ThemeProvider } from "./theme/ThemeContext";

function App() {
  return (
    <ThemeProvider>
      <Navbar />
      <MainContent />
      <Footer />
    </ThemeProvider>
  );
}

// ──────────────────────────────────────────
// Navbar.jsx — use theme anywhere
// ──────────────────────────────────────────
import { useTheme } from "./theme/ThemeContext";

function Navbar() {
  const { themeState, dispatch } = useTheme();

  return (
    <nav style={{
      backgroundColor: themeState.theme === "dark" ? "#1f2937" : "#ffffff",
      color: themeState.theme === "dark" ? "#ffffff" : "#000000",
    }}>
      <span>My App</span>
      <button onClick={() => dispatch({ type: "TOGGLE_THEME" })}>
        {themeState.theme === "light" ? "🌙 Dark" : "☀️ Light"}
      </button>
    </nav>
  );
}
```

### 🔐 Auth System with Context

```jsx
// auth/AuthContext.jsx
import { createContext, useContext, useReducer, useEffect } from "react";

const AuthContext = createContext(null);

const initialAuthState = {
  user: null,
  token: null,
  isAuthenticated: false,
  isLoading: true, // loading while checking stored token
  error: null,
};

function authReducer(state, action) {
  switch (action.type) {
    case "LOGIN_SUCCESS":
      return {
        ...state,
        user: action.payload.user,
        token: action.payload.token,
        isAuthenticated: true,
        isLoading: false,
        error: null,
      };

    case "LOGIN_FAILURE":
      return {
        ...state,
        user: null,
        token: null,
        isAuthenticated: false,
        isLoading: false,
        error: action.payload,
      };

    case "LOGOUT":
      return {
        ...initialAuthState,
        isLoading: false,
      };

    case "SET_LOADING":
      return { ...state, isLoading: action.payload };

    case "CLEAR_ERROR":
      return { ...state, error: null };

    case "RESTORE_SESSION":
      return {
        ...state,
        user: action.payload.user,
        token: action.payload.token,
        isAuthenticated: true,
        isLoading: false,
      };

    default:
      return state;
  }
}

export function AuthProvider({ children }) {
  const [authState, dispatch] = useReducer(authReducer, initialAuthState);

  // Restore session from localStorage on mount
  useEffect(() => {
    const token = localStorage.getItem("token");
    const user = localStorage.getItem("user");

    if (token && user) {
      dispatch({
        type: "RESTORE_SESSION",
        payload: { token, user: JSON.parse(user) },
      });
    } else {
      dispatch({ type: "SET_LOADING", payload: false });
    }
  }, []);

  // Login function
  const login = async (credentials) => {
    dispatch({ type: "SET_LOADING", payload: true });
    try {
      // API call
      const res = await fetch("/api/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(credentials),
      });
      const data = await res.json();

      if (!res.ok) throw new Error(data.message);

      // Save to localStorage
      localStorage.setItem("token", data.token);
      localStorage.setItem("user", JSON.stringify(data.user));

      dispatch({ type: "LOGIN_SUCCESS", payload: data });
    } catch (err) {
      dispatch({ type: "LOGIN_FAILURE", payload: err.message });
    }
  };

  const logout = () => {
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    dispatch({ type: "LOGOUT" });
  };

  return (
    <AuthContext.Provider value={{ authState, login, logout, dispatch }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error("useAuth must be used inside <AuthProvider>");
  return context;
}

// ──────────────────────────────────────────
// LoginPage.jsx
// ──────────────────────────────────────────
function LoginPage() {
  const { authState, login } = useAuth();
  const [form, setForm] = useState({ email: "", password: "" });

  const handleSubmit = (e) => {
    e.preventDefault();
    login(form);
  };

  if (authState.isAuthenticated) {
    return <p>Welcome, {authState.user.name}! You are logged in.</p>;
  }

  return (
    <form onSubmit={handleSubmit}>
      {authState.error && <p style={{ color: "red" }}>{authState.error}</p>}
      <input
        type="email"
        value={form.email}
        onChange={(e) => setForm({ ...form, email: e.target.value })}
        placeholder="Email"
      />
      <input
        type="password"
        value={form.password}
        onChange={(e) => setForm({ ...form, password: e.target.value })}
        placeholder="Password"
      />
      <button type="submit" disabled={authState.isLoading}>
        {authState.isLoading ? "Logging in..." : "Login"}
      </button>
    </form>
  );
}
```

### 📁 Recommended Folder Structure

```
src/
├── context/
│   ├── auth/
│   │   ├── AuthContext.jsx      ← Provider + useAuth hook
│   │   ├── authReducer.js       ← Reducer function
│   │   ├── authActions.js       ← Action creators
│   │   └── authTypes.js         ← Action type constants
│   ├── cart/
│   │   ├── CartContext.jsx
│   │   ├── cartReducer.js
│   │   └── cartTypes.js
│   └── theme/
│       ├── ThemeContext.jsx
│       └── themeReducer.js
├── components/
│   ├── Navbar.jsx
│   └── ...
└── App.jsx
```

---

## 11. Async Operations with `useReducer`

### ⚠️ The Challenge

`useReducer` is **synchronous** — the reducer function cannot be async. So how do we handle API calls?

**Answer:** The async work happens in your component (or custom hook), and you `dispatch` actions at different points:
- Before the call → dispatch `REQUEST` (show loading)
- After success → dispatch `SUCCESS` (show data)
- After failure → dispatch `FAILURE` (show error)

### 🎯 The REQUEST / SUCCESS / FAILURE Pattern

```javascript
// The "three state" pattern for any async operation

const initialState = {
  data: null,        // the actual data
  isLoading: false,  // are we waiting?
  error: null,       // did something go wrong?
};

function dataReducer(state, action) {
  switch (action.type) {
    case "FETCH_REQUEST":
      return {
        ...state,
        isLoading: true,   // show spinner
        error: null,       // clear previous errors
      };

    case "FETCH_SUCCESS":
      return {
        ...state,
        isLoading: false,   // hide spinner
        data: action.payload, // store the data
        error: null,
      };

    case "FETCH_FAILURE":
      return {
        ...state,
        isLoading: false,   // hide spinner
        error: action.payload, // show error message
      };

    default:
      return state;
  }
}
```

### 👤 User Fetching Example

```jsx
// UserFetcher.jsx
import { useReducer, useEffect } from "react";

const initialState = {
  users: [],
  isLoading: false,
  error: null,
};

function usersReducer(state, action) {
  switch (action.type) {
    case "FETCH_REQUEST":
      return { ...state, isLoading: true, error: null };

    case "FETCH_SUCCESS":
      return { ...state, isLoading: false, users: action.payload };

    case "FETCH_FAILURE":
      return { ...state, isLoading: false, error: action.payload };

    case "DELETE_USER":
      return {
        ...state,
        users: state.users.filter((u) => u.id !== action.payload),
      };

    default:
      return state;
  }
}

function UserList() {
  const [state, dispatch] = useReducer(usersReducer, initialState);

  const fetchUsers = async () => {
    dispatch({ type: "FETCH_REQUEST" }); // 1. Start loading

    try {
      const response = await fetch("https://jsonplaceholder.typicode.com/users");

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const users = await response.json();
      dispatch({ type: "FETCH_SUCCESS", payload: users }); // 2. Success

    } catch (error) {
      dispatch({ type: "FETCH_FAILURE", payload: error.message }); // 3. Failure
    }
  };

  useEffect(() => {
    fetchUsers(); // fetch on component mount
  }, []);

  // Render different UI based on state
  if (state.isLoading) {
    return <div>Loading users... ⏳</div>;
  }

  if (state.error) {
    return (
      <div>
        <p style={{ color: "red" }}>Error: {state.error}</p>
        <button onClick={fetchUsers}>Retry</button>
      </div>
    );
  }

  return (
    <div>
      <h2>Users ({state.users.length})</h2>
      <button onClick={fetchUsers}>Refresh</button>
      <ul>
        {state.users.map((user) => (
          <li key={user.id}>
            {user.name} — {user.email}
            <button onClick={() => dispatch({ type: "DELETE_USER", payload: user.id })}>
              ✕
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

### 📋 CRUD Example

```jsx
// PostsCRUD.jsx — Create, Read, Update, Delete with API
import { useReducer } from "react";

const initialState = {
  posts: [],
  isLoading: false,
  isSubmitting: false,
  error: null,
  editingPost: null,
};

function postsReducer(state, action) {
  switch (action.type) {
    // ── FETCH ──
    case "FETCH_REQUEST":
      return { ...state, isLoading: true };
    case "FETCH_SUCCESS":
      return { ...state, isLoading: false, posts: action.payload };
    case "FETCH_FAILURE":
      return { ...state, isLoading: false, error: action.payload };

    // ── CREATE ──
    case "CREATE_REQUEST":
      return { ...state, isSubmitting: true };
    case "CREATE_SUCCESS":
      return {
        ...state,
        isSubmitting: false,
        posts: [action.payload, ...state.posts],
      };
    case "CREATE_FAILURE":
      return { ...state, isSubmitting: false, error: action.payload };

    // ── UPDATE ──
    case "UPDATE_SUCCESS":
      return {
        ...state,
        posts: state.posts.map((p) =>
          p.id === action.payload.id ? action.payload : p
        ),
        editingPost: null,
      };

    // ── DELETE ──
    case "DELETE_SUCCESS":
      return {
        ...state,
        posts: state.posts.filter((p) => p.id !== action.payload),
      };

    // ── UI ──
    case "SET_EDITING":
      return { ...state, editingPost: action.payload };
    case "CLEAR_ERROR":
      return { ...state, error: null };

    default:
      return state;
  }
}

function PostsCRUD() {
  const [state, dispatch] = useReducer(postsReducer, initialState);

  const createPost = async (title) => {
    dispatch({ type: "CREATE_REQUEST" });
    try {
      const res = await fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ title, body: "...", userId: 1 }),
      });
      const newPost = await res.json();
      dispatch({ type: "CREATE_SUCCESS", payload: newPost });
    } catch (err) {
      dispatch({ type: "CREATE_FAILURE", payload: err.message });
    }
  };

  const deletePost = async (id) => {
    try {
      await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
        method: "DELETE",
      });
      dispatch({ type: "DELETE_SUCCESS", payload: id });
    } catch (err) {
      dispatch({ type: "FETCH_FAILURE", payload: err.message });
    }
  };

  // ... render UI
}
```

---

## 12. Real World Projects

### 📝 Project 1: Todo App (Complete)

```jsx
// TodoApp.jsx — Complete production-ready todo app
import { useReducer, useState } from "react";

// ── Types ──
const ACTIONS = {
  ADD: "ADD_TODO",
  TOGGLE: "TOGGLE_TODO",
  DELETE: "DELETE_TODO",
  EDIT: "EDIT_TODO",
  SET_FILTER: "SET_FILTER",
  CLEAR_COMPLETED: "CLEAR_COMPLETED",
};

// ── Initial State ──
const initialState = {
  todos: [
    { id: 1, text: "Learn useReducer", completed: true, createdAt: new Date() },
    { id: 2, text: "Build a project", completed: false, createdAt: new Date() },
  ],
  filter: "all", // "all" | "active" | "completed"
  nextId: 3,
};

// ── Reducer ──
function todoReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: state.nextId,
            text: action.payload.trim(),
            completed: false,
            createdAt: new Date(),
          },
        ],
        nextId: state.nextId + 1,
      };

    case ACTIONS.TOGGLE:
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        ),
      };

    case ACTIONS.DELETE:
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };

    case ACTIONS.EDIT:
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text }
            : todo
        ),
      };

    case ACTIONS.SET_FILTER:
      return { ...state, filter: action.payload };

    case ACTIONS.CLEAR_COMPLETED:
      return { ...state, todos: state.todos.filter((t) => !t.completed) };

    default:
      return state;
  }
}

// ── Component ──
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [inputText, setInputText] = useState("");

  // Compute filtered todos
  const filteredTodos = state.todos.filter((todo) => {
    if (state.filter === "active") return !todo.completed;
    if (state.filter === "completed") return todo.completed;
    return true; // "all"
  });

  const activeCount = state.todos.filter((t) => !t.completed).length;

  const handleAdd = (e) => {
    e.preventDefault();
    if (inputText.trim()) {
      dispatch({ type: ACTIONS.ADD, payload: inputText });
      setInputText("");
    }
  };

  return (
    <div style={{ maxWidth: "500px", margin: "40px auto", padding: "20px" }}>
      <h1>📝 Todo App</h1>

      {/* Input */}
      <form onSubmit={handleAdd} style={{ display: "flex", gap: "8px" }}>
        <input
          value={inputText}
          onChange={(e) => setInputText(e.target.value)}
          placeholder="What needs to be done?"
          style={{ flex: 1, padding: "8px" }}
        />
        <button type="submit">Add</button>
      </form>

      {/* Filter tabs */}
      <div style={{ display: "flex", gap: "8px", margin: "16px 0" }}>
        {["all", "active", "completed"].map((f) => (
          <button
            key={f}
            onClick={() => dispatch({ type: ACTIONS.SET_FILTER, payload: f })}
            style={{ fontWeight: state.filter === f ? "bold" : "normal" }}
          >
            {f.charAt(0).toUpperCase() + f.slice(1)}
          </button>
        ))}
      </div>

      {/* Todo list */}
      <ul style={{ listStyle: "none", padding: 0 }}>
        {filteredTodos.map((todo) => (
          <li
            key={todo.id}
            style={{ display: "flex", alignItems: "center", gap: "8px", padding: "8px 0" }}
          >
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: ACTIONS.TOGGLE, payload: todo.id })}
            />
            <span style={{ flex: 1, textDecoration: todo.completed ? "line-through" : "none" }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: ACTIONS.DELETE, payload: todo.id })}>
              🗑️
            </button>
          </li>
        ))}
      </ul>

      {/* Footer */}
      <div style={{ display: "flex", justifyContent: "space-between", marginTop: "16px" }}>
        <span>{activeCount} item{activeCount !== 1 ? "s" : ""} left</span>
        <button onClick={() => dispatch({ type: ACTIONS.CLEAR_COMPLETED })}>
          Clear completed
        </button>
      </div>
    </div>
  );
}

export default TodoApp;
```

### 🔐 Project 2: Multi-Step Form

```jsx
// MultiStepForm.jsx
import { useReducer } from "react";

const STEPS = {
  PERSONAL: 1,
  CONTACT: 2,
  REVIEW: 3,
  SUCCESS: 4,
};

const initialState = {
  currentStep: STEPS.PERSONAL,
  formData: {
    // Step 1
    firstName: "",
    lastName: "",
    dateOfBirth: "",
    // Step 2
    email: "",
    phone: "",
    address: "",
    // Step 3 (calculated)
    confirmed: false,
  },
  errors: {},
  isSubmitting: false,
};

function formReducer(state, action) {
  switch (action.type) {
    case "UPDATE_FIELD":
      return {
        ...state,
        formData: { ...state.formData, [action.payload.field]: action.payload.value },
        errors: { ...state.errors, [action.payload.field]: "" },
      };

    case "NEXT_STEP":
      return { ...state, currentStep: state.currentStep + 1, errors: {} };

    case "PREV_STEP":
      return { ...state, currentStep: state.currentStep - 1, errors: {} };

    case "SET_ERRORS":
      return { ...state, errors: action.payload };

    case "SUBMIT_REQUEST":
      return { ...state, isSubmitting: true };

    case "SUBMIT_SUCCESS":
      return { ...state, isSubmitting: false, currentStep: STEPS.SUCCESS };

    case "SUBMIT_FAILURE":
      return {
        ...state,
        isSubmitting: false,
        errors: { submit: action.payload },
      };

    case "RESET":
      return initialState;

    default:
      return state;
  }
}

// Validation for each step
function validateStep(step, formData) {
  const errors = {};

  if (step === STEPS.PERSONAL) {
    if (!formData.firstName.trim()) errors.firstName = "First name is required";
    if (!formData.lastName.trim()) errors.lastName = "Last name is required";
    if (!formData.dateOfBirth) errors.dateOfBirth = "Date of birth is required";
  }

  if (step === STEPS.CONTACT) {
    if (!formData.email.includes("@")) errors.email = "Valid email required";
    if (formData.phone.length < 10) errors.phone = "Valid phone required";
  }

  return errors;
}

function MultiStepForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  const { currentStep, formData, errors, isSubmitting } = state;

  const handleChange = (field) => (e) => {
    dispatch({ type: "UPDATE_FIELD", payload: { field, value: e.target.value } });
  };

  const handleNext = () => {
    const stepErrors = validateStep(currentStep, formData);
    if (Object.keys(stepErrors).length > 0) {
      dispatch({ type: "SET_ERRORS", payload: stepErrors });
      return;
    }
    dispatch({ type: "NEXT_STEP" });
  };

  const handleSubmit = async () => {
    dispatch({ type: "SUBMIT_REQUEST" });
    try {
      await new Promise((r) => setTimeout(r, 1500)); // Simulate API
      dispatch({ type: "SUBMIT_SUCCESS" });
    } catch (err) {
      dispatch({ type: "SUBMIT_FAILURE", payload: "Submission failed. Please try again." });
    }
  };

  // Progress indicator
  const progressPercent = ((currentStep - 1) / (Object.keys(STEPS).length - 1)) * 100;

  return (
    <div style={{ maxWidth: "500px", margin: "40px auto", padding: "20px" }}>
      <h2>Registration Form</h2>

      {/* Progress bar */}
      {currentStep !== STEPS.SUCCESS && (
        <div style={{ background: "#e5e7eb", borderRadius: "4px", marginBottom: "24px" }}>
          <div
            style={{
              width: `${progressPercent}%`,
              height: "8px",
              background: "#3b82f6",
              borderRadius: "4px",
              transition: "width 0.3s",
            }}
          />
        </div>
      )}

      {/* Step 1: Personal Info */}
      {currentStep === STEPS.PERSONAL && (
        <div>
          <h3>Step 1: Personal Information</h3>
          <div>
            <input
              placeholder="First Name"
              value={formData.firstName}
              onChange={handleChange("firstName")}
            />
            {errors.firstName && <span style={{ color: "red" }}>{errors.firstName}</span>}
          </div>
          <div>
            <input
              placeholder="Last Name"
              value={formData.lastName}
              onChange={handleChange("lastName")}
            />
            {errors.lastName && <span style={{ color: "red" }}>{errors.lastName}</span>}
          </div>
          <div>
            <input
              type="date"
              value={formData.dateOfBirth}
              onChange={handleChange("dateOfBirth")}
            />
            {errors.dateOfBirth && <span style={{ color: "red" }}>{errors.dateOfBirth}</span>}
          </div>
          <button onClick={handleNext}>Next →</button>
        </div>
      )}

      {/* Step 2: Contact Info */}
      {currentStep === STEPS.CONTACT && (
        <div>
          <h3>Step 2: Contact Information</h3>
          <div>
            <input
              type="email"
              placeholder="Email"
              value={formData.email}
              onChange={handleChange("email")}
            />
            {errors.email && <span style={{ color: "red" }}>{errors.email}</span>}
          </div>
          <div>
            <input
              placeholder="Phone"
              value={formData.phone}
              onChange={handleChange("phone")}
            />
            {errors.phone && <span style={{ color: "red" }}>{errors.phone}</span>}
          </div>
          <input
            placeholder="Address"
            value={formData.address}
            onChange={handleChange("address")}
          />
          <div>
            <button onClick={() => dispatch({ type: "PREV_STEP" })}>← Back</button>
            <button onClick={handleNext}>Next →</button>
          </div>
        </div>
      )}

      {/* Step 3: Review */}
      {currentStep === STEPS.REVIEW && (
        <div>
          <h3>Step 3: Review Your Info</h3>
          <p><strong>Name:</strong> {formData.firstName} {formData.lastName}</p>
          <p><strong>DOB:</strong> {formData.dateOfBirth}</p>
          <p><strong>Email:</strong> {formData.email}</p>
          <p><strong>Phone:</strong> {formData.phone}</p>
          {errors.submit && <p style={{ color: "red" }}>{errors.submit}</p>}
          <div>
            <button onClick={() => dispatch({ type: "PREV_STEP" })}>← Back</button>
            <button onClick={handleSubmit} disabled={isSubmitting}>
              {isSubmitting ? "Submitting..." : "Submit ✓"}
            </button>
          </div>
        </div>
      )}

      {/* Step 4: Success */}
      {currentStep === STEPS.SUCCESS && (
        <div style={{ textAlign: "center" }}>
          <h2>🎉 Registration Complete!</h2>
          <p>Welcome, {formData.firstName}!</p>
          <button onClick={() => dispatch({ type: "RESET" })}>Register Another</button>
        </div>
      )}
    </div>
  );
}

export default MultiStepForm;
```

### 💰 Project 3: Expense Tracker

```jsx
// ExpenseTracker.jsx
import { useReducer, useState } from "react";

const initialState = {
  expenses: [],
  balance: 3000, // starting balance
  filter: "all", // "all" | "income" | "expense"
};

function expenseReducer(state, action) {
  switch (action.type) {
    case "ADD_TRANSACTION": {
      const amount = action.payload.type === "income"
        ? Math.abs(action.payload.amount)
        : -Math.abs(action.payload.amount);

      return {
        ...state,
        expenses: [
          {
            id: Date.now(),
            description: action.payload.description,
            amount,
            type: action.payload.type,
            date: new Date().toLocaleDateString(),
            category: action.payload.category,
          },
          ...state.expenses,
        ],
        balance: state.balance + amount,
      };
    }

    case "DELETE_TRANSACTION": {
      const tx = state.expenses.find((e) => e.id === action.payload);
      return {
        ...state,
        expenses: state.expenses.filter((e) => e.id !== action.payload),
        balance: state.balance - tx.amount,
      };
    }

    case "SET_FILTER":
      return { ...state, filter: action.payload };

    default:
      return state;
  }
}

function ExpenseTracker() {
  const [state, dispatch] = useReducer(expenseReducer, initialState);
  const [form, setForm] = useState({ description: "", amount: "", type: "expense", category: "Food" });

  const totalIncome = state.expenses
    .filter((e) => e.type === "income")
    .reduce((sum, e) => sum + e.amount, 0);

  const totalExpenses = state.expenses
    .filter((e) => e.type === "expense")
    .reduce((sum, e) => sum + Math.abs(e.amount), 0);

  const handleAdd = (e) => {
    e.preventDefault();
    if (!form.description || !form.amount) return;
    dispatch({
      type: "ADD_TRANSACTION",
      payload: { ...form, amount: parseFloat(form.amount) },
    });
    setForm({ description: "", amount: "", type: "expense", category: "Food" });
  };

  const filteredExpenses = state.expenses.filter((e) => {
    if (state.filter === "income") return e.type === "income";
    if (state.filter === "expense") return e.type === "expense";
    return true;
  });

  return (
    <div style={{ maxWidth: "600px", margin: "40px auto", padding: "20px" }}>
      <h1>💰 Expense Tracker</h1>

      {/* Balance cards */}
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: "16px", marginBottom: "24px" }}>
        <div style={{ background: "#f3f4f6", padding: "16px", borderRadius: "8px", textAlign: "center" }}>
          <div style={{ fontSize: "24px", fontWeight: "bold" }}>${state.balance.toFixed(2)}</div>
          <div>Balance</div>
        </div>
        <div style={{ background: "#d1fae5", padding: "16px", borderRadius: "8px", textAlign: "center" }}>
          <div style={{ fontSize: "24px", fontWeight: "bold", color: "#065f46" }}>+${totalIncome.toFixed(2)}</div>
          <div>Income</div>
        </div>
        <div style={{ background: "#fee2e2", padding: "16px", borderRadius: "8px", textAlign: "center" }}>
          <div style={{ fontSize: "24px", fontWeight: "bold", color: "#991b1b" }}>-${totalExpenses.toFixed(2)}</div>
          <div>Expenses</div>
        </div>
      </div>

      {/* Add transaction form */}
      <form onSubmit={handleAdd}>
        <input
          placeholder="Description"
          value={form.description}
          onChange={(e) => setForm({ ...form, description: e.target.value })}
        />
        <input
          type="number"
          placeholder="Amount"
          value={form.amount}
          onChange={(e) => setForm({ ...form, amount: e.target.value })}
        />
        <select value={form.type} onChange={(e) => setForm({ ...form, type: e.target.value })}>
          <option value="expense">Expense</option>
          <option value="income">Income</option>
        </select>
        <select value={form.category} onChange={(e) => setForm({ ...form, category: e.target.value })}>
          <option>Food</option>
          <option>Transport</option>
          <option>Shopping</option>
          <option>Bills</option>
          <option>Salary</option>
          <option>Other</option>
        </select>
        <button type="submit">Add Transaction</button>
      </form>

      {/* Transaction list */}
      <div>
        {["all", "income", "expense"].map((f) => (
          <button key={f} onClick={() => dispatch({ type: "SET_FILTER", payload: f })}>
            {f}
          </button>
        ))}
      </div>

      <ul style={{ listStyle: "none", padding: 0 }}>
        {filteredExpenses.map((tx) => (
          <li
            key={tx.id}
            style={{
              display: "flex",
              justifyContent: "space-between",
              padding: "12px",
              marginBottom: "8px",
              borderLeft: `4px solid ${tx.type === "income" ? "#10b981" : "#ef4444"}`,
              background: "#f9fafb",
            }}
          >
            <div>
              <strong>{tx.description}</strong>
              <small style={{ marginLeft: "8px", color: "#6b7280" }}>{tx.category} · {tx.date}</small>
            </div>
            <div style={{ display: "flex", alignItems: "center", gap: "8px" }}>
              <span style={{ color: tx.amount >= 0 ? "#10b981" : "#ef4444", fontWeight: "bold" }}>
                {tx.amount >= 0 ? "+" : ""}{tx.amount.toFixed(2)}
              </span>
              <button onClick={() => dispatch({ type: "DELETE_TRANSACTION", payload: tx.id })}>✕</button>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default ExpenseTracker;
```

---

## 13. Performance Optimization

### ⚡ Lazy Initialization

If your initial state is expensive to compute, use the third argument of `useReducer`:

```jsx
// ❌ Without lazy init — runs on EVERY render
const expensiveInit = JSON.parse(localStorage.getItem("todos") || "[]");
const [state, dispatch] = useReducer(reducer, expensiveInit);

// ✅ With lazy init — runs ONLY ONCE on mount
function initState(defaultTodos) {
  // This function is called ONCE when component mounts
  const stored = localStorage.getItem("todos");
  return {
    todos: stored ? JSON.parse(stored) : defaultTodos,
    filter: "all",
  };
}

// Third argument is the init function, second is the arg passed to init function
const [state, dispatch] = useReducer(reducer, [], initState);
//                                             ↑           ↑
//                                   passed to initState   init function
```

### 🧠 Preventing Unnecessary Re-renders with `memo`

```jsx
// Without memo — child re-renders whenever parent state changes
function TodoItem({ todo, onToggle, onDelete }) {
  console.log("TodoItem re-rendering..."); // runs too often
  return (
    <li>
      <input type="checkbox" checked={todo.completed} onChange={() => onToggle(todo.id)} />
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>✕</button>
    </li>
  );
}

// With memo — child only re-renders if its OWN props change
import { memo, useCallback, useReducer } from "react";

const TodoItem = memo(function TodoItem({ todo, onToggle, onDelete }) {
  console.log("TodoItem re-rendering..."); // only when THIS todo changes
  return (
    <li>
      <input type="checkbox" checked={todo.completed} onChange={() => onToggle(todo.id)} />
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>✕</button>
    </li>
  );
});

// In parent component — use useCallback so function references stay stable
function TodoList() {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  // Without useCallback, new function created every render → memo is useless!
  const handleToggle = useCallback(
    (id) => dispatch({ type: "TOGGLE_TODO", payload: id }),
    [dispatch] // dispatch is stable, so this never changes
  );

  const handleDelete = useCallback(
    (id) => dispatch({ type: "DELETE_TODO", payload: id }),
    [dispatch]
  );

  return (
    <ul>
      {state.todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
}
```

### 💡 `useMemo` for Derived State

```jsx
import { useReducer, useMemo } from "react";

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  // ❌ Recalculates on EVERY render (even unrelated state changes)
  const completedCount = state.todos.filter((t) => t.completed).length;

  // ✅ Only recalculates when state.todos changes
  const completedCount = useMemo(
    () => state.todos.filter((t) => t.completed).length,
    [state.todos]
  );

  // ✅ Filtered todos — expensive operation
  const filteredTodos = useMemo(() => {
    return state.todos.filter((todo) => {
      if (state.filter === "active") return !todo.completed;
      if (state.filter === "completed") return todo.completed;
      return true;
    });
  }, [state.todos, state.filter]); // only recalc when these change

  return (
    <div>
      <p>Completed: {completedCount}</p>
      {filteredTodos.map((todo) => <div key={todo.id}>{todo.text}</div>)}
    </div>
  );
}
```

### 🔍 Optimizing Reducer Logic

```javascript
// ✅ Put expensive calculations in reducers, not components
// The reducer only runs when dispatched, not on every render

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
    case "REMOVE_ITEM":
    case "UPDATE_QUANTITY": {
      // Calculate derived values inside reducer
      // They'll be in state and don't need recalculation
      const updatedItems = /* ... compute items ... */;
      return {
        ...state,
        items: updatedItems,
        // Pre-compute these so components don't have to
        total: updatedItems.reduce((sum, i) => sum + i.price * i.quantity, 0),
        itemCount: updatedItems.reduce((sum, i) => sum + i.quantity, 0),
        hasItems: updatedItems.length > 0,
      };
    }
    default:
      return state;
  }
}

// Now in component: just read state.total, state.itemCount — no computation!
```

---

## 14. Common Mistakes

### ❌ Mistake 1: Mutating State Directly

```jsx
// ❌ WRONG — mutating state
function badReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      state.items.push(action.payload); // MUTATING state.items!
      return state;                     // React won't re-render!

    case "UPDATE_USER":
      state.user.name = action.payload; // MUTATING nested property!
      return state;                     // UI won't update!
  }
}

// ✅ CORRECT — creating new state
function goodReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      return {
        ...state,
        items: [...state.items, action.payload], // NEW array
      };

    case "UPDATE_USER":
      return {
        ...state,
        user: { ...state.user, name: action.payload }, // NEW user object
      };
  }
}
```

### ❌ Mistake 2: Missing Return / Missing Default Case

```jsx
// ❌ WRONG — no default case, some actions return undefined
function badReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    // What about all other action types?
    // → reducer returns undefined → state becomes undefined → app crashes!
  }
}

// ✅ CORRECT — always have a default
function goodReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    default:
      return state; // Return unchanged state for unknown actions
  }
}
```

### ❌ Mistake 3: Defining Reducer Inside Component

```jsx
// ❌ WRONG — reducer defined INSIDE component
function Counter() {
  // This creates a NEW reducer function on every render!
  // useReducer will see a different function reference → may cause issues
  function reducer(state, action) {
    // ...
  }

  const [state, dispatch] = useReducer(reducer, { count: 0 });
  // ...
}

// ✅ CORRECT — reducer defined OUTSIDE component
function counterReducer(state, action) {
  // ... defined once, stable reference
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  // ...
}
```

### ❌ Mistake 4: Wrong Payload Structure

```jsx
// ❌ WRONG — inconsistent payload structures
dispatch({ type: "UPDATE_USER", name: "Alice" });          // name as direct property?
dispatch({ type: "UPDATE_USER", data: { name: "Alice" }}); // data? payload? value?
dispatch({ type: "UPDATE_USER", user: { name: "Alice" }}); // user?

// → Reducer code:
case "UPDATE_USER":
  return { ...state, user: action.name ?? action.data?.name ?? action.user?.name };
// ↑ Confusing! You don't know which property to use!

// ✅ CORRECT — consistent payload convention
dispatch({ type: "UPDATE_USER", payload: { name: "Alice" } });  // always 'payload'
dispatch({ type: "DELETE_TODO", payload: 5 });                   // always 'payload'
dispatch({ type: "SET_FILTER", payload: "completed" });          // always 'payload'

// → Reducer code:
case "UPDATE_USER":
  return { ...state, user: { ...state.user, ...action.payload } };
// ↑ Clean and consistent!
```

### ❌ Mistake 5: Not Spreading Nested State

```jsx
// ❌ WRONG — accidentally wiping nested state
const state = {
  user: { name: "Alice", email: "alice@example.com" },
  settings: { theme: "dark" }
};

// Trying to update user.name but forgetting to spread user
case "UPDATE_NAME":
  return {
    ...state,
    user: { name: action.payload }  // ❌ email is GONE!
    // user is now { name: "Bob" } — email wiped!
  };

// ✅ CORRECT — spread at every level
case "UPDATE_NAME":
  return {
    ...state,
    user: { ...state.user, name: action.payload }  // ✅ email preserved
  };
```

### ❌ Mistake 6: Async Code in Reducer

```jsx
// ❌ WRONG — async operation inside reducer
function badReducer(state, action) {
  switch (action.type) {
    case "FETCH_USER":
      // ❌ NEVER do async stuff in reducers!
      fetch("/api/user").then(res => ...); // This doesn't work as expected
      return state;
  }
}

// ✅ CORRECT — async code in the component/event handler
function UserComponent() {
  const [state, dispatch] = useReducer(reducer, initialState);

  const fetchUser = async () => {
    dispatch({ type: "FETCH_REQUEST" });           // sync: set loading
    try {
      const res = await fetch("/api/user");        // async work HERE
      const user = await res.json();
      dispatch({ type: "FETCH_SUCCESS", payload: user }); // sync: set result
    } catch (err) {
      dispatch({ type: "FETCH_FAILURE", payload: err.message });
    }
  };
}
```

---

## 15. `useReducer` Best Practices

### 📁 File Organization

```
src/
├── features/
│   └── todos/
│       ├── components/
│       │   ├── TodoList.jsx
│       │   ├── TodoItem.jsx
│       │   └── TodoForm.jsx
│       ├── context/
│       │   └── TodoContext.jsx    ← Context + Provider + useContext hook
│       ├── reducer/
│       │   ├── todoReducer.js     ← Pure reducer function
│       │   ├── todoActions.js     ← Action creators
│       │   └── todoTypes.js       ← Action type constants
│       └── index.js               ← Public exports
```

### 🏷️ Action Types File

```javascript
// todoTypes.js — single source of truth for action types
export const TODO_TYPES = {
  ADD: "todos/ADD",
  DELETE: "todos/DELETE",
  TOGGLE: "todos/TOGGLE",
  EDIT: "todos/EDIT",
  SET_FILTER: "todos/SET_FILTER",
  CLEAR_COMPLETED: "todos/CLEAR_COMPLETED",
};
```

### 🏭 Action Creators File

```javascript
// todoActions.js — factories for creating actions
import { TODO_TYPES } from "./todoTypes";

export const addTodo = (text) => ({
  type: TODO_TYPES.ADD,
  payload: text,
});

export const deleteTodo = (id) => ({
  type: TODO_TYPES.DELETE,
  payload: id,
});

export const toggleTodo = (id) => ({
  type: TODO_TYPES.TOGGLE,
  payload: id,
});

export const setFilter = (filter) => ({
  type: TODO_TYPES.SET_FILTER,
  payload: filter,
});
```

### 📝 Reducer File

```javascript
// todoReducer.js — pure state logic
import { TODO_TYPES } from "./todoTypes";

export const initialTodoState = {
  todos: [],
  filter: "all",
};

export function todoReducer(state = initialTodoState, action) {
  switch (action.type) {
    case TODO_TYPES.ADD:
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, completed: false },
        ],
      };

    case TODO_TYPES.DELETE:
      return {
        ...state,
        todos: state.todos.filter((t) => t.id !== action.payload),
      };

    case TODO_TYPES.TOGGLE:
      return {
        ...state,
        todos: state.todos.map((t) =>
          t.id === action.payload ? { ...t, completed: !t.completed } : t
        ),
      };

    case TODO_TYPES.SET_FILTER:
      return { ...state, filter: action.payload };

    default:
      return state;
  }
}
```

### 🎣 Context File

```javascript
// TodoContext.jsx
import { createContext, useContext, useReducer } from "react";
import { todoReducer, initialTodoState } from "../reducer/todoReducer";

const TodoContext = createContext(null);

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialTodoState);
  return (
    <TodoContext.Provider value={{ state, dispatch }}>
      {children}
    </TodoContext.Provider>
  );
}

export function useTodos() {
  const context = useContext(TodoContext);
  if (!context) throw new Error("useTodos must be used inside <TodoProvider>");
  return context;
}
```

---

## 16. Redux vs `useReducer`

### 🔄 Similarities

Both Redux and `useReducer`:
- Use **reducer functions** (`(state, action) => newState`)
- Use **actions** (`{ type, payload }`)
- Enforce **immutable state updates**
- Produce **predictable state transitions**
- Support **action creators**
- Are great for **complex state logic**

### ⚡ Differences

| Feature | `useReducer` | Redux |
|---------|-------------|-------|
| **Scope** | One component (or via Context) | Global app-wide |
| **Setup** | Zero — built into React | Need `npm install redux react-redux` |
| **DevTools** | None out of the box | Powerful Redux DevTools |
| **Middleware** | Not supported | Middleware (thunk, saga, etc.) |
| **Time travel** | No | Yes (via DevTools) |
| **Selectors** | Manual or useMemo | `reselect` library |
| **Async** | Handle in component | Middleware handles it |
| **Boilerplate** | Minimal | More (actions, reducers, selectors) |
| **Bundle size** | 0kb extra | ~7kb (Redux Toolkit) |
| **Community** | React built-in | Huge ecosystem |

### 🤔 When to Use Which

```
USE useReducer WHEN:
━━━━━━━━━━━━━━━━━━━
✅ Medium complexity apps
✅ State is local to a feature
✅ 1-5 related state values
✅ You don't need DevTools
✅ Small-medium team
✅ You want zero extra dependencies
✅ Learning or prototyping

USE REDUX (Redux Toolkit) WHEN:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Large, complex apps
✅ Many features share state
✅ Team needs debugging tools (Redux DevTools)
✅ You need middleware (complex async, logging)
✅ You want normalized state management
✅ Large team needing strict conventions
✅ Enterprise applications
```

### 🏗️ Code Comparison

```javascript
// ─────────────────────────────────────────
// useReducer approach
// ─────────────────────────────────────────
const [state, dispatch] = useReducer(counterReducer, { count: 0 });
dispatch({ type: "INCREMENT" });

// ─────────────────────────────────────────
// Redux Toolkit approach
// ─────────────────────────────────────────
// store.js
import { configureStore, createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { count: 0 },
  reducers: {
    increment: (state) => { state.count += 1; }, // Immer handles immutability
    decrement: (state) => { state.count -= 1; },
  },
});

export const { increment, decrement } = counterSlice.actions;
export const store = configureStore({ reducer: counterSlice.reducer });

// Component
import { useSelector, useDispatch } from "react-redux";
const count = useSelector((state) => state.count);
const dispatch = useDispatch();
dispatch(increment());
```

---

## 17. Interview Questions

### 🟢 Beginner Questions

**Q1: What is `useReducer` and what does it do?**
> `useReducer` is a React Hook for managing state using a reducer function pattern. It's an alternative to `useState` for complex state logic. It takes a reducer function and initial state, and returns the current state plus a `dispatch` function for triggering state updates.

**Q2: What are the arguments to `useReducer`?**
> `useReducer(reducer, initialState, init?)` — (1) a reducer function that takes `(state, action)` and returns new state, (2) the initial state value, and optionally (3) a lazy initialization function.

**Q3: What is a reducer function?**
> A reducer is a pure function that takes the current state and an action object, and returns the new state. It must never mutate the current state, must always return a state value, and must be deterministic (same inputs → same output).

**Q4: What is `dispatch`?**
> `dispatch` is a function returned by `useReducer` that sends action objects to the reducer. When you call `dispatch({ type: "ACTION" })`, React calls your reducer with the current state and that action, then re-renders with the new state.

**Q5: Why should the reducer be a pure function?**
> Pure functions produce predictable results — same inputs always give same output. This makes state transitions reliable, debuggable, and testable. Side effects in reducers (API calls, randomness, mutations) would make state unpredictable.

---

### 🟡 Intermediate Questions

**Q6: When would you choose `useReducer` over `useState`?**
> Choose `useReducer` when: you have multiple related state variables that should be updated together; state updates depend on the previous state; the same state needs to be updated from many different events; you want to centralize update logic; or you're building complex features like carts, auth, forms with complex validation.

**Q7: What is an action object and what must it contain?**
> An action is a plain JavaScript object that describes what event occurred. It must have a `type` property (string identifier), and can optionally have a `payload` property carrying the data needed for the update. Example: `{ type: "ADD_TODO", payload: "Buy milk" }`.

**Q8: How do you handle multiple state properties with `useReducer`?**
> Use an object as state and spread it in each case: `return { ...state, updatedProperty: newValue }`. This creates a new state object while preserving all other properties (immutable update pattern).

**Q9: What is lazy initialization and when would you use it?**
> Lazy initialization passes a third `init` function to `useReducer`. React calls `init(initialArg)` once on mount to compute the initial state. Use it when computing initial state is expensive (like parsing from localStorage, complex transformations) to avoid running that logic on every render.

**Q10: Explain the flow of a dispatch call.**
> 1. Component calls `dispatch({ type: "INCREMENT" })` → 2. React internally calls `reducer(currentState, action)` → 3. Reducer returns a new state object → 4. React compares old and new state by reference → 5. If different, React re-renders the component with new state.

---

### 🔴 Advanced Questions

**Q11: How would you implement global state management with `useReducer` and Context?**
> Create a Context, define a reducer and initial state, create a Provider component that calls `useReducer` and passes `{state, dispatch}` through the context value, then create a custom hook (`useMyContext`) that calls `useContext` and throws if used outside the Provider. This gives any component in the tree access to both state and the ability to dispatch actions.

**Q12: How do you handle async operations with `useReducer`?**
> Reducers must be synchronous, so async work happens in component functions or custom hooks. Use the REQUEST/SUCCESS/FAILURE pattern: dispatch `FETCH_REQUEST` before the async call (set loading), dispatch `FETCH_SUCCESS` with data on success, dispatch `FETCH_FAILURE` with error message on failure. The reducer handles each action synchronously.

**Q13: What are the performance implications of `useReducer` vs `useState`?**
> Both have similar performance. With `useReducer`, `dispatch` is stable (doesn't change between renders), making it safer in dependency arrays and compatible with `useCallback` for memoized handlers. For performance optimization, combine `memo` for child components, `useCallback` for stable handler references, `useMemo` for derived/filtered data, and lazy initialization for expensive initial state.

**Q14: Explain the concept of reducer composition.**
> Reducer composition means combining multiple smaller, focused reducers into one larger one. Each sub-reducer handles one slice of state independently. A `combineReducers`-style function calls each sub-reducer with its own slice and an action, letting each decide if it handles that action. This keeps reducers small, focused, and testable.

**Q15: Why should you define the reducer outside the component? What happens if you define it inside?**
> Defining inside the component creates a new function reference on every render. While React's `useReducer` handles this correctly (it doesn't call the reducer just because the reference changed), it's unnecessary work for JavaScript's garbage collector and can cause subtle issues with some optimization patterns. It's also a signal that the reducer might accidentally close over component variables, tempting developers to make it impure.

---

## 18. Practice Exercises

### 🟢 Basic Exercises

**Exercise 1: Simple Counter**
Build a counter with increment, decrement, and reset. Add a "Step" feature where users can set the increment/decrement step size.

```jsx
// Starter
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  // TODO: Handle INCREMENT, DECREMENT, RESET, SET_STEP
}
```

**Exercise 2: Light Switch**
Build a light switch with on/off state. Add color control (red, green, blue, white).

**Exercise 3: Shopping List**
Build a shopping list where you can add items, mark them as bought, and remove them.

---

### 🟡 Intermediate Exercises

**Exercise 4: Stopwatch**
Build a stopwatch with start, pause, reset, and lap recording.

```jsx
const initialState = {
  isRunning: false,
  time: 0,     // in milliseconds
  laps: [],    // array of lap times
};
// Hint: Use useEffect with setInterval for the timer
// Handle: START, PAUSE, RESET, LAP actions
```

**Exercise 5: Image Gallery with Filters**
Build a photo gallery with category filters, favorites, and a lightbox modal.

```jsx
const initialState = {
  images: [...],      // array of image objects
  filter: "all",      // category filter
  favorites: [],      // favorite image IDs
  selectedImage: null, // currently opened in lightbox
};
```

**Exercise 6: Quiz App**
Build a quiz with multiple-choice questions, score tracking, and a results screen.

---

### 🔴 Advanced Exercises

**Exercise 7: Kanban Board**
Build a task board with columns (Todo, In Progress, Done) and drag-to-reorder functionality.

```jsx
const initialState = {
  columns: {
    todo: { title: "To Do", tasks: [] },
    inProgress: { title: "In Progress", tasks: [] },
    done: { title: "Done", tasks: [] },
  },
};
// Handle: ADD_TASK, MOVE_TASK, DELETE_TASK, REORDER_TASK
```

**Exercise 8: Rich Text Editor State**
Build the state layer for a text editor with bold/italic/underline formatting, undo/redo history.

```jsx
const initialState = {
  content: "",
  formatting: { bold: false, italic: false, underline: false },
  history: [],    // for undo/redo
  historyIndex: -1,
};
```

**Exercise 9: Data Table with Sort, Filter, Pagination**
Build a data table component that uses `useReducer` for all state:

```jsx
const initialState = {
  data: [...],          // original data
  filtered: [...],      // filtered/sorted view
  searchTerm: "",
  sortBy: null,
  sortDir: "asc",       // "asc" | "desc"
  currentPage: 1,
  pageSize: 10,
  selectedRows: [],
};
```

---

## 19. Mini Assignments

### 🎯 Assignment 1: Password Manager

**Build:** A password manager UI that lets users store, view, and generate passwords.

**State to manage:**
- Entries: `[{ id, website, username, password, isFavorite }]`
- UI: `{ showPasswords: Set<id>, searchTerm, filterFavorites }`
- Form: `{ isOpen, editingId, form: { website, username, password } }`

**Features:** Add, edit, delete entries; toggle password visibility; search; favorites; password strength indicator; copy to clipboard.

---

### 🎯 Assignment 2: Budget Planner

**Build:** A monthly budget planner with categories and a progress dashboard.

**State to manage:**
- Budget categories: `[{ id, name, budgeted, spent, color }]`
- Transactions: `[{ id, categoryId, amount, description, date }]`
- Month/year navigation
- Summary stats

**Features:** Set budgets per category; add transactions; visual progress bars; over-budget warnings; month-to-month navigation.

---

### 🎯 Assignment 3: Recipe Manager

**Build:** A recipe management app.

**State to manage:**
- Recipes: `[{ id, name, ingredients, steps, servings, tags, isFavorite }]`
- Filters: `{ search, tags, favoritesOnly }`
- UI: `{ viewingRecipe, editingRecipe, scaledServings }`

**Features:** Add/edit/delete recipes; ingredient scaling (2x servings = 2x ingredients); filter by tags; search; favorites; print view.

---

### 🎯 Assignment 4: Real-time Chat UI (State Layer)

**Build:** The state layer for a chat application (no actual WebSocket needed — simulate messages).

**State to manage:**
```jsx
const initialState = {
  rooms: [{ id, name, unread }],
  activeRoomId: null,
  messages: { [roomId]: [{ id, text, sender, timestamp, read }] },
  currentUser: { id, name },
  typingUsers: { [roomId]: [userId] },
  draft: { [roomId]: "" },
};
```

**Actions to implement:** Select room; send message; mark messages as read; update draft; typing indicators.

---

## 20. Final Summary & Cheat Sheet

### 🎓 Key Takeaways

1. **`useReducer` = useState + centralized logic** — it manages state like `useState` but keeps all update logic in one pure function.

2. **Reducers must be pure** — no side effects, no mutations, no async code. Same input → same output, always.

3. **Never mutate state** — always create new objects/arrays with spread operator or `.map()`, `.filter()`.

4. **Actions describe what happened** — `{ type: "USER_LOGGED_IN", payload: userData }`. The reducer decides what to do with it.

5. **Default case is mandatory** — always return unchanged state for unknown actions.

6. **Define reducers outside components** — keeps them stable and prevents accidental closure over component state.

7. **Context + useReducer = lightweight Redux** — perfect for medium-complexity apps without needing external libraries.

8. **Async goes in the component** — dispatch REQUEST before, SUCCESS or FAILURE after.

9. **Use action constants** — prevents typos that silently do nothing.

10. **Combine with `memo` + `useCallback`** — for performance when passing dispatch-based handlers to memoized children.

---

### 📋 `useReducer` Cheat Sheet

```jsx
// ─────────────────────────────────────────────────────
// SETUP
// ─────────────────────────────────────────────────────

// 1. Define initial state
const initialState = { count: 0, items: [], isLoading: false };

// 2. Define reducer (OUTSIDE component)
function reducer(state, action) {
  switch (action.type) {
    case "ACTION_NAME":
      return { ...state, property: newValue };  // immutable update
    default:
      return state;                              // always return state
  }
}

// 3. Use in component
const [state, dispatch] = useReducer(reducer, initialState);
// Optional: const [state, dispatch] = useReducer(reducer, arg, initFn);

// ─────────────────────────────────────────────────────
// DISPATCHING
// ─────────────────────────────────────────────────────

dispatch({ type: "INCREMENT" });                          // no payload
dispatch({ type: "ADD_ITEM", payload: item });            // with payload
dispatch({ type: "UPDATE", payload: { id: 1, name: "x" } }); // object payload

// ─────────────────────────────────────────────────────
// IMMUTABLE UPDATE PATTERNS
// ─────────────────────────────────────────────────────

// Update object property
return { ...state, name: "new name" };

// Add to array
return { ...state, items: [...state.items, newItem] };

// Remove from array
return { ...state, items: state.items.filter(i => i.id !== id) };

// Update item in array
return {
  ...state,
  items: state.items.map(i => i.id === id ? { ...i, ...updates } : i),
};

// Update nested object
return {
  ...state,
  user: { ...state.user, address: { ...state.user.address, city: "Mumbai" } },
};

// ─────────────────────────────────────────────────────
// WITH CONTEXT (Global State)
// ─────────────────────────────────────────────────────

const MyContext = createContext(null);

function MyProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <MyContext.Provider value={{ state, dispatch }}>
      {children}
    </MyContext.Provider>
  );
}

function useMyContext() {
  const ctx = useContext(MyContext);
  if (!ctx) throw new Error("Must be inside <MyProvider>");
  return ctx;
}

// ─────────────────────────────────────────────────────
// ASYNC PATTERN
// ─────────────────────────────────────────────────────

const fetchData = async () => {
  dispatch({ type: "FETCH_REQUEST" });    // loading: true
  try {
    const data = await apiCall();
    dispatch({ type: "FETCH_SUCCESS", payload: data }); // loading: false, data
  } catch (err) {
    dispatch({ type: "FETCH_FAILURE", payload: err.message }); // loading: false, error
  }
};

// ─────────────────────────────────────────────────────
// ACTION CREATORS PATTERN
// ─────────────────────────────────────────────────────

export const increment = () => ({ type: "INCREMENT" });
export const addItem = (item) => ({ type: "ADD_ITEM", payload: item });

dispatch(increment());
dispatch(addItem({ id: 1, name: "Laptop" }));
```

### ⚡ Quick Revision Notes

| Concept | One-Line Summary |
|---------|-----------------|
| `useReducer` | React hook: `(reducer, initialState)` → `[state, dispatch]` |
| `reducer` | Pure function: `(state, action)` → `newState` |
| `dispatch` | Function to send actions to the reducer |
| `action` | Plain object: `{ type: String, payload?: any }` |
| `initialState` | Starting value of state |
| `pure function` | Same inputs → same output, no side effects |
| `immutable update` | Create new state objects instead of mutating |
| `action.type` | Identifier string for what happened |
| `payload` | Data carried with an action |
| `action creator` | Function that returns an action object |
| `lazy init` | Third arg to `useReducer` for expensive initial computation |
| `REQUEST/SUCCESS/FAILURE` | Pattern for async state management |
| Context + useReducer | DIY global state without Redux |

---

### 🗺️ The Big Picture

```
┌──────────────────────────────────────────────────────────────────┐
│                    REACT APP WITH useReducer                      │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                      Context Provider                        │  │
│  │   const [state, dispatch] = useReducer(reducer, init)        │  │
│  │                                                               │  │
│  │  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐     │  │
│  │  │  Component A │   │  Component B │   │  Component C │     │  │
│  │  │              │   │              │   │              │     │  │
│  │  │  reads state │   │ dispatches   │   │ reads state  │     │  │
│  │  │  via context │   │ actions via  │   │ dispatches   │     │  │
│  │  │              │   │ context      │   │ actions      │     │  │
│  │  └──────────────┘   └──────────────┘   └──────────────┘     │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                               │                                    │
│                               │ dispatch({ type, payload })        │
│                               ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                        REDUCER                               │  │
│  │                                                               │  │
│  │   switch(action.type) {                                       │  │
│  │     case "ACTION": return { ...state, ...updates }           │  │
│  │     default: return state                                     │  │
│  │   }                                                           │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                               │                                    │
│                               │ new state                          │
│                               ▼                                    │
│                        React re-renders                            │
│                     components that read state                     │
└──────────────────────────────────────────────────────────────────┘
```

---

> **🎉 Congratulations!** You've completed the entire `useReducer` guide — from the very first concept of what state is, all the way to production patterns, performance optimization, and real-world projects. Practice makes perfect: pick one of the mini assignments and build it from scratch. Happy coding! ⚛️
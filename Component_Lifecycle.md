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
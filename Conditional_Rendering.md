# 07 — Conditional Rendering in React (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** Conditional Rendering
> **Level:** Beginner → Advanced
> **Prerequisites:** JSX (Module 02), State & useState (Module 04), Components & Props (Module 03)

---

## Table of Contents

1. [What is Conditional Rendering?](#1-what-is-conditional-rendering)
2. [How React Handles Conditional Rendering Internally](#2-how-react-handles-conditional-rendering-internally)
3. [Pattern 1 — if/else Statements](#3-pattern-1--ifelse-statements)
4. [Pattern 2 — Ternary Operator](#4-pattern-2--ternary-operator)
5. [Pattern 3 — Logical AND (&&)](#5-pattern-3--logical-and-)
6. [Pattern 4 — Logical OR (||) and Nullish Coalescing (??)](#6-pattern-4--logical-or--and-nullish-coalescing-)
7. [Pattern 5 — Early Return](#7-pattern-5--early-return)
8. [Pattern 6 — Switch Statements](#8-pattern-6--switch-statements)
9. [Pattern 7 — Object/Map Lookup](#9-pattern-7--objectmap-lookup)
10. [Pattern 8 — Component-Based Conditions](#10-pattern-8--component-based-conditions)
11. [Rendering null — Hiding Components Completely](#11-rendering-null--hiding-components-completely)
12. [Conditional CSS Classes](#12-conditional-css-classes)
13. [Conditional Rendering vs CSS display:none](#13-conditional-rendering-vs-css-displaynone)
14. [Code Examples (Beginner → Advanced)](#14-code-examples-beginner--advanced)
15. [Real-World Use Cases](#15-real-world-use-cases)
16. [Best Practices](#16-best-practices)
17. [Common Mistakes](#17-common-mistakes)
18. [Performance Considerations](#18-performance-considerations)
19. [Interview Questions](#19-interview-questions)
20. [Practice Tasks](#20-practice-tasks)
21. [Summary](#21-summary)

---

## 1. What is Conditional Rendering?

### Simple Explanation

**Conditional rendering** means showing different UI based on certain conditions — displaying a login form when the user is not authenticated, showing a loading spinner while data is being fetched, rendering an error message when something goes wrong, or hiding a button when the user doesn't have permission.

It's the React equivalent of "show this, but only if...".

```jsx
// Simple example — show different content based on a condition
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }
  return <h1>Please sign in.</h1>;
}
```

### Technical Explanation

Since JSX is JavaScript, you can use the full power of JavaScript's conditional logic — `if`, `else`, ternary operators, logical operators, `switch` statements — to decide what JSX to return or include. React renders whatever your component returns, so any JavaScript that controls the return value controls what appears on screen.

### Why It Matters

Nearly every real-world UI has conditional rendering:

| Scenario | Condition |
|----------|-----------|
| Auth screens | Show dashboard OR login page |
| Data loading | Show spinner OR actual content |
| Error states | Show error message OR normal content |
| Permissions | Show admin controls OR hide them |
| Empty states | Show "No items" OR the item list |
| Feature flags | Show new feature OR old feature |
| Form validation | Show error message OR nothing |

---

## 2. How React Handles Conditional Rendering Internally

### What React Does with Conditional Output

React does not have a special "conditional rendering" mechanism. It simply renders whatever value your component function returns or includes in JSX. The condition lives entirely in JavaScript — React just processes the result.

```jsx
// React sees the result of your condition — not the condition itself
function Badge({ isPremium }) {
  // JavaScript runs: isPremium is true → expression evaluates to <span>Premium</span>
  return isPremium ? <span>Premium</span> : null;

  // React receives: <span>Premium</span>  OR  null
  // React renders: the span              OR  nothing
}
```

### Mounting vs Updating — Key Difference

When a condition changes and a component goes from rendered to not-rendered (or vice versa):

```jsx
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div>
      <button onClick={() => setShowModal(true)}>Open</button>
      {showModal && <Modal />}  {/* Modal mounts/unmounts based on condition */}
    </div>
  );
}
```

- When `showModal` becomes `true` → `Modal` **mounts** (created fresh, `useEffect` runs)
- When `showModal` becomes `false` → `Modal` **unmounts** (destroyed, cleanup runs, all state lost)

This is fundamentally different from hiding with CSS — unmounting destroys the component and all its state.

### Reconciliation and Conditional Rendering

React's reconciler compares the previous render output with the new output. When a conditional changes:

```
Previous render: <div> <Header /> <Content /> </div>
New render:      <div> <Header /> <ErrorBanner /> <Content /> </div>

React sees: new element inserted at index 1
React: mounts ErrorBanner between Header and Content
```

The position in the JSX tree matters — React uses position to match elements across renders.

---

## 3. Pattern 1 — if/else Statements

### Basic if/else

The most explicit and readable pattern. Best used when the condition controls what the **entire component** returns.

```jsx
function UserStatus({ user, isLoading, error }) {
  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (error) {
    return <ErrorMessage message={error} />;
  }

  if (!user) {
    return <p>No user found.</p>;
  }

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### if/else with Variable Assignment

When only part of the JSX is conditional, assign the conditional piece to a variable:

```jsx
function Notification({ type, message }) {
  let icon;
  let colorClass;

  if (type === 'success') {
    icon = '✅';
    colorClass = 'notification--success';
  } else if (type === 'error') {
    icon = '❌';
    colorClass = 'notification--error';
  } else if (type === 'warning') {
    icon = '⚠️';
    colorClass = 'notification--warning';
  } else {
    icon = 'ℹ️';
    colorClass = 'notification--info';
  }

  return (
    <div className={`notification ${colorClass}`}>
      <span className="notification__icon">{icon}</span>
      <p className="notification__message">{message}</p>
    </div>
  );
}
```

### When to Use if/else

- When the entire component renders different output based on a condition
- When you have complex multi-branch conditions
- When using early returns for guard clauses
- When the condition involves more than one line of logic

---

## 4. Pattern 2 — Ternary Operator

### Basic Ternary

The ternary operator (`condition ? trueValue : falseValue`) is the most common pattern for inline conditional rendering inside JSX.

```jsx
function LoginButton({ isLoggedIn, onLogin, onLogout }) {
  return (
    <button onClick={isLoggedIn ? onLogout : onLogin}>
      {isLoggedIn ? 'Log Out' : 'Log In'}
    </button>
  );
}
```

### Ternary with JSX Blocks

```jsx
function CartSummary({ cart }) {
  return (
    <div className="cart">
      {cart.length > 0 ? (
        <ul>
          {cart.map(item => (
            <li key={item.id}>{item.name} — ₹{item.price}</li>
          ))}
        </ul>
      ) : (
        <div className="cart__empty">
          <p>Your cart is empty.</p>
          <a href="/shop">Start shopping →</a>
        </div>
      )}
    </div>
  );
}
```

### Nested Ternary — Use Sparingly

Nested ternaries quickly become unreadable. Limit to one level of nesting:

```jsx
// ❌ Hard to read — nested ternary
function StatusBadge({ status }) {
  return (
    <span className={
      status === 'active' ? 'badge--green' :
      status === 'pending' ? 'badge--yellow' :
      status === 'banned' ? 'badge--red' :
      'badge--gray'
    }>
      {status}
    </span>
  );
}

// ✅ Better — use object lookup (see Pattern 7) or variable
function StatusBadge({ status }) {
  const config = {
    active:  { className: 'badge--green',  label: 'Active' },
    pending: { className: 'badge--yellow', label: 'Pending' },
    banned:  { className: 'badge--red',    label: 'Banned' },
  };
  const { className, label } = config[status] ?? { className: 'badge--gray', label: status };

  return <span className={`badge ${className}`}>{label}</span>;
}
```

### When to Use Ternary

- Two clear alternatives: show A or show B
- Short, readable conditions that fit cleanly in JSX
- Inline text or attribute changes
- Avoid for more than two branches

---

## 5. Pattern 3 — Logical AND (&&)

### Basic Usage

The `&&` operator short-circuits: if the left side is falsy, it returns the left side without evaluating the right side. If truthy, it returns the right side.

```jsx
function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      {user.isVerified && <span className="badge">✅ Verified</span>}
      {user.isPremium && <span className="badge badge--gold">⭐ Premium</span>}
      {user.bio && <p className="bio">{user.bio}</p>}
    </div>
  );
}
```

### The Zero Bug — Critical!

When the left operand is `0` (number zero), JavaScript's `&&` returns `0`, not `false`. React renders `0` on screen.

```jsx
const items = [];

// ❌ Renders "0" when items is empty!
{items.length && <ItemList items={items} />}

// ✅ Fix 1 — Convert to boolean
{items.length > 0 && <ItemList items={items} />}

// ✅ Fix 2 — Double negation
{!!items.length && <ItemList items={items} />}

// ✅ Fix 3 — Ternary
{items.length ? <ItemList items={items} /> : null}
```

### Other Falsy Value Traps

```jsx
// ❌ What renders when count = 0?
{count && <Counter count={count} />}  // renders "0" — bug!

// ❌ What renders when name = ""?
{name && <Label text={name} />}        // renders nothing — might be intentional or bug

// ❌ What renders when value = NaN?
{value && <Display value={value} />}   // renders nothing — probably a bug

// ✅ Be explicit with your conditions
{count > 0 && <Counter count={count} />}
{name.length > 0 && <Label text={name} />}
{!isNaN(value) && <Display value={value} />}
```

### When to Use &&

- "Show this only if" — when there's no "else" case needed
- Optional UI elements (badges, hints, tooltips, extra buttons)
- Showing something only when data exists

---

## 6. Pattern 4 — Logical OR (||) and Nullish Coalescing (??)

### Logical OR for Fallback Content

`||` returns the right side when the left is falsy:

```jsx
function UserCard({ user }) {
  return (
    <div>
      {/* Show bio or fallback text */}
      <p>{user.bio || 'No bio provided.'}</p>

      {/* Show avatar or placeholder */}
      <img
        src={user.avatarUrl || '/images/default-avatar.png'}
        alt={user.name}
      />
    </div>
  );
}
```

### Nullish Coalescing for null/undefined Only

`??` only falls back when the left side is `null` or `undefined` — not `0`, `false`, or `''`:

```jsx
function PriceDisplay({ price, discountedPrice }) {
  // ❌ || would treat 0 as falsy — wrong for prices!
  const displayPrice = discountedPrice || price; // if discountedPrice is 0, shows price

  // ✅ ?? only falls back for null/undefined — 0 is a valid price
  const displayPrice2 = discountedPrice ?? price;

  return <p>₹{displayPrice2}</p>;
}
```

### OR vs AND vs ?? — Comparison

```jsx
const value = 0;

value || 'default'   // 'default' — 0 is falsy
value && 'show'      // 0          — returns the falsy left side
value ?? 'default'   // 0          — only null/undefined trigger fallback
```

---

## 7. Pattern 5 — Early Return

### What is an Early Return?

An early return exits the component function before reaching the main JSX. It's used as a **guard clause** to handle edge cases at the top, keeping the main render logic clean and unindented.

```jsx
// ❌ Without early return — deeply nested
function ProductCard({ product, isLoading, error }) {
  return (
    <div>
      {isLoading ? (
        <Spinner />
      ) : error ? (
        <ErrorBanner message={error} />
      ) : product ? (
        <div className="product">
          <h2>{product.name}</h2>
          <p>{product.description}</p>
          <span>₹{product.price}</span>
        </div>
      ) : null}
    </div>
  );
}

// ✅ With early returns — flat, readable, clean
function ProductCard({ product, isLoading, error }) {
  if (isLoading) return <Spinner />;
  if (error)     return <ErrorBanner message={error} />;
  if (!product)  return null;

  // Main render — no nesting needed
  return (
    <div className="product">
      <h2>{product.name}</h2>
      <p>{product.description}</p>
      <span>₹{product.price}</span>
    </div>
  );
}
```

### Guard Clause Order

Handle the most likely failure states first, main success render last:

```jsx
function DataTable({ data, isLoading, error, columns }) {
  // 1. Handle loading
  if (isLoading) return <TableSkeleton columns={columns} />;

  // 2. Handle error
  if (error) return <ErrorState message={error} onRetry={refetch} />;

  // 3. Handle empty state
  if (!data || data.length === 0) return <EmptyState message="No records found." />;

  // 4. Handle invalid data structure
  if (!Array.isArray(data)) return <ErrorState message="Invalid data format." />;

  // 5. Main render — we know data is a non-empty array
  return (
    <table>
      <thead>
        <tr>{columns.map(col => <th key={col.key}>{col.label}</th>)}</tr>
      </thead>
      <tbody>
        {data.map(row => (
          <tr key={row.id}>
            {columns.map(col => <td key={col.key}>{row[col.key]}</td>)}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### When to Use Early Returns

- Loading, error, and empty states at the top of a component
- Permission checks (user must be logged in, must be admin)
- Invalid prop combinations
- Any "if not X, bail out" logic

---

## 8. Pattern 6 — Switch Statements

### Basic Switch

Switch is great for multiple distinct cases based on a single value — cleaner than long if/else chains:

```jsx
function AlertBanner({ type, message }) {
  function renderIcon() {
    switch (type) {
      case 'success': return '✅';
      case 'error':   return '❌';
      case 'warning': return '⚠️';
      case 'info':    return 'ℹ️';
      default:        return null;
    }
  }

  return (
    <div className={`alert alert--${type}`} role="alert">
      <span className="alert__icon">{renderIcon()}</span>
      <p className="alert__message">{message}</p>
    </div>
  );
}
```

### Switch for Component Selection

```jsx
function Step({ stepNumber, formData, onUpdate }) {
  switch (stepNumber) {
    case 1:
      return <PersonalInfoStep data={formData} onUpdate={onUpdate} />;
    case 2:
      return <ContactDetailsStep data={formData} onUpdate={onUpdate} />;
    case 3:
      return <PaymentStep data={formData} onUpdate={onUpdate} />;
    case 4:
      return <ReviewStep data={formData} />;
    default:
      return <ErrorState message={`Unknown step: ${stepNumber}`} />;
  }
}
```

### Switch in a Helper Function Inside JSX

```jsx
function Dashboard({ userRole }) {
  function renderRoleContent() {
    switch (userRole) {
      case 'admin':
        return <AdminPanel />;
      case 'editor':
        return <EditorWorkspace />;
      case 'viewer':
        return <ReadOnlyDashboard />;
      default:
        return <AccessDenied />;
    }
  }

  return (
    <div className="dashboard">
      <Navbar />
      <main>{renderRoleContent()}</main>
      <Footer />
    </div>
  );
}
```

### When to Use Switch

- Three or more mutually exclusive cases based on a single value
- Status displays, step wizards, role-based rendering
- When object lookup (Pattern 7) would be less readable

---

## 9. Pattern 7 — Object/Map Lookup

### Why Object Lookup?

Object lookup replaces long `if/else` chains and `switch` statements with a clean, declarative, data-driven approach. It's especially powerful when the conditions map to components.

```jsx
// ❌ Long if/else — hard to extend
function StatusBadge({ status }) {
  if (status === 'active')   return <span className="badge green">Active</span>;
  if (status === 'pending')  return <span className="badge yellow">Pending</span>;
  if (status === 'inactive') return <span className="badge gray">Inactive</span>;
  if (status === 'banned')   return <span className="badge red">Banned</span>;
  return null;
}

// ✅ Object lookup — declarative and easy to extend
const STATUS_CONFIG = {
  active:   { label: 'Active',   className: 'badge--green'  },
  pending:  { label: 'Pending',  className: 'badge--yellow' },
  inactive: { label: 'Inactive', className: 'badge--gray'   },
  banned:   { label: 'Banned',   className: 'badge--red'    },
};

function StatusBadge({ status }) {
  const config = STATUS_CONFIG[status];
  if (!config) return null;

  return (
    <span className={`badge ${config.className}`}>
      {config.label}
    </span>
  );
}
```

### Component Map Pattern

Store component references in an object and look them up dynamically:

```jsx
import HomeIcon from './icons/HomeIcon';
import SettingsIcon from './icons/SettingsIcon';
import ProfileIcon from './icons/ProfileIcon';
import NotificationsIcon from './icons/NotificationsIcon';

// Map string keys to component references
const ICON_MAP = {
  home:          HomeIcon,
  settings:      SettingsIcon,
  profile:       ProfileIcon,
  notifications: NotificationsIcon,
};

function NavIcon({ name, ...props }) {
  const IconComponent = ICON_MAP[name];

  if (!IconComponent) {
    console.warn(`Unknown icon: ${name}`);
    return null;
  }

  return <IconComponent {...props} />;
}

// Usage — dynamic icon rendering
<NavIcon name="home" size={24} />
<NavIcon name="settings" size={24} />
```

### Page Router Pattern

```jsx
const PAGES = {
  home:     () => <HomePage />,
  about:    () => <AboutPage />,
  products: () => <ProductsPage />,
  contact:  () => <ContactPage />,
  404:      () => <NotFoundPage />,
};

function App({ currentPath }) {
  const renderPage = PAGES[currentPath] ?? PAGES[404];
  return (
    <div className="app">
      <Navbar />
      <main>{renderPage()}</main>
      <Footer />
    </div>
  );
}
```

### When to Use Object Lookup

- Multiple cases based on a string/enum value
- Mapping values to components or configuration
- When you need to easily add/remove cases without changing logic
- Feature flags, permission maps, theme configs

---

## 10. Pattern 8 — Component-Based Conditions

### Wrapper Components for Conditions

Extract complex conditional logic into dedicated wrapper components:

```jsx
// ─── Permission Guard ──────────────────────────────────
function CanAccess({ requiredRole, userRole, children, fallback = null }) {
  const roleHierarchy = { admin: 3, editor: 2, viewer: 1 };
  const hasAccess = (roleHierarchy[userRole] ?? 0) >= (roleHierarchy[requiredRole] ?? 0);

  return hasAccess ? children : fallback;
}

// Usage
<CanAccess requiredRole="admin" userRole={currentUser.role}>
  <AdminControlPanel />
</CanAccess>

<CanAccess
  requiredRole="editor"
  userRole={currentUser.role}
  fallback={<p>You need editor access to see this.</p>}
>
  <ContentEditor />
</CanAccess>
```

### Feature Flag Component

```jsx
function Feature({ flag, children, fallback = null }) {
  const { flags } = useFeatureFlags();
  return flags[flag] ? children : fallback;
}

// Usage
<Feature flag="new_checkout_flow">
  <NewCheckout />
</Feature>

<Feature flag="beta_dashboard" fallback={<ClassicDashboard />}>
  <BetaDashboard />
</Feature>
```

### Auth Guard Component

```jsx
function RequireAuth({ children, redirectTo = '/login' }) {
  const { user, isLoading } = useAuth();

  if (isLoading) return <PageLoadingSpinner />;
  if (!user) return <Navigate to={redirectTo} />;

  return children;
}

// Usage — protecting routes
<Route
  path="/dashboard"
  element={
    <RequireAuth>
      <Dashboard />
    </RequireAuth>
  }
/>
```

### Show/Hide Component

```jsx
function Show({ when, fallback = null, children }) {
  return when ? children : fallback;
}

// Usage — cleaner than inline ternary
<Show when={isLoggedIn} fallback={<LoginPrompt />}>
  <UserDashboard user={currentUser} />
</Show>

<Show when={items.length > 0} fallback={<EmptyState />}>
  <ItemGrid items={items} />
</Show>
```

---

## 11. Rendering null — Hiding Components Completely

### Returning null

A component can return `null` to render nothing at all. The component still exists in React's tree and can still hold state and effects, but it produces no DOM output.

```jsx
function ErrorBanner({ error }) {
  if (!error) return null; // Renders nothing when no error

  return (
    <div className="error-banner" role="alert">
      <strong>Error:</strong> {error}
    </div>
  );
}
```

### null vs undefined vs false

All of these cause React to render nothing:

```jsx
{null}       // renders nothing ✅
{undefined}  // renders nothing ✅
{false}      // renders nothing ✅
{0}          // renders "0" ⚠️ — this is the zero bug
{''}         // renders nothing ✅ — empty string
```

### Returning null from render doesn't affect lifecycle

```jsx
function ConditionalModal({ isOpen }) {
  const [data, setData] = useState(null);

  // useEffect still runs even when isOpen is false
  useEffect(() => {
    console.log('Modal effect ran');
  }, [isOpen]);

  if (!isOpen) return null; // Nothing rendered, but component is still mounted

  return <div className="modal">{data}</div>;
}
```

---

## 12. Conditional CSS Classes

### Manual Conditional Class

```jsx
function Button({ variant, isActive, isDisabled }) {
  const classes = [
    'btn',
    variant && `btn--${variant}`,
    isActive && 'btn--active',
    isDisabled && 'btn--disabled',
  ]
    .filter(Boolean) // remove falsy values
    .join(' ');

  return (
    <button className={classes} disabled={isDisabled}>
      Click Me
    </button>
  );
}
```

### Template Literal Approach

```jsx
function Card({ isSelected, isHighlighted, size = 'md' }) {
  return (
    <div
      className={`
        card
        card--${size}
        ${isSelected ? 'card--selected' : ''}
        ${isHighlighted ? 'card--highlighted' : ''}
      `.trim()}
    >
      Content
    </div>
  );
}
```

### clsx / classnames Library (Industry Standard)

```bash
npm install clsx
```

```jsx
import clsx from 'clsx';

function Button({ variant = 'primary', size = 'md', isLoading, isDisabled, className }) {
  return (
    <button
      className={clsx(
        'btn',
        `btn--${variant}`,
        `btn--${size}`,
        isLoading && 'btn--loading',
        isDisabled && 'btn--disabled',
        className  // allow parent to add classes
      )}
      disabled={isDisabled || isLoading}
    >
      {isLoading ? <Spinner /> : 'Submit'}
    </button>
  );
}
```

### CSS Modules with Conditional Classes

```jsx
import styles from './Card.module.css';
import clsx from 'clsx';

function Card({ isActive, isPinned }) {
  return (
    <div
      className={clsx(
        styles.card,
        isActive && styles.active,
        isPinned && styles.pinned
      )}
    >
      Content
    </div>
  );
}
```

---

## 13. Conditional Rendering vs CSS display:none

This is an important architectural decision with real performance implications.

### Conditional Rendering (Mount/Unmount)

```jsx
{isOpen && <HeavyModal />}
```

| Pros | Cons |
|------|------|
| Not in DOM when hidden — no memory usage | Animation on enter/exit harder to implement |
| State is reset when hidden | Takes time to mount (especially heavy components) |
| Effects run/clean up properly | |
| Screen readers don't see hidden content | |

### CSS display:none (Always Mounted)

```jsx
<HeavyModal style={{ display: isOpen ? 'block' : 'none' }} />
```

| Pros | Cons |
|------|------|
| No mount/unmount cost on toggle | Always in DOM — uses memory and CSS selectors |
| State preserved when hidden | Screen readers may still read hidden content |
| Easier CSS animations | Effects keep running when hidden |
| Instant show/hide | Initial page load is heavier |

### Decision Guide

```
Does the component need to preserve state when hidden?
  ├── YES (e.g., form mid-fill, video player) → CSS display:none or visibility:hidden
  └── NO → conditional rendering (&&, ternary)

Is the component heavy to mount?
  ├── YES and shown/hidden frequently → CSS display:none
  └── NO or shown/hidden rarely → conditional rendering

Does the component have cleanup effects?
  └── YES → conditional rendering (ensures proper cleanup)

Is this content for screen readers?
  └── Content MUST be hidden from all users → conditional rendering
      Content is visually hidden but accessible → aria-hidden or visually-hidden CSS
```

### The Best of Both — CSS + aria-hidden

```jsx
function Drawer({ isOpen, children }) {
  return (
    <aside
      className={`drawer ${isOpen ? 'drawer--open' : ''}`}
      aria-hidden={!isOpen}           // hide from screen readers when closed
      inert={!isOpen || undefined}    // prevent interaction when closed (modern browsers)
    >
      {children}
    </aside>
  );
}
```

---

## 14. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Loading/Error/Data States

```jsx
// File: components/UserCard/UserCard.jsx
import { useState, useEffect } from 'react';

function UserCard({ userId }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    setError(null);

    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error('User not found');
        return res.json();
      })
      .then(data => setUser(data))
      .catch(err => setError(err.message))
      .finally(() => setIsLoading(false));
  }, [userId]);

  // Guard clauses — early returns
  if (isLoading) {
    return (
      <div className="user-card user-card--skeleton">
        <div className="skeleton skeleton--avatar" />
        <div className="skeleton skeleton--line" />
        <div className="skeleton skeleton--line skeleton--short" />
      </div>
    );
  }

  if (error) {
    return (
      <div className="user-card user-card--error">
        <p>⚠️ {error}</p>
      </div>
    );
  }

  if (!user) return null;

  return (
    <div className="user-card">
      <div className="user-card__avatar">
        {user.name.charAt(0).toUpperCase()}
      </div>
      <div className="user-card__info">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
        <a href={`https://${user.website}`} target="_blank" rel="noreferrer">
          {user.website}
        </a>
      </div>
    </div>
  );
}

export default UserCard;
```

---

### Example 2 — Intermediate: Multi-State Dashboard Widget

```jsx
// File: components/DashboardWidget/DashboardWidget.jsx
import { useState } from 'react';
import clsx from 'clsx';

const WIDGET_STATES = {
  IDLE:    'idle',
  LOADING: 'loading',
  SUCCESS: 'success',
  ERROR:   'error',
  EMPTY:   'empty',
};

function DashboardWidget({ title, fetchData, renderContent }) {
  const [widgetState, setWidgetState] = useState(WIDGET_STATES.IDLE);
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  async function handleLoad() {
    setWidgetState(WIDGET_STATES.LOADING);
    try {
      const result = await fetchData();
      if (!result || (Array.isArray(result) && result.length === 0)) {
        setWidgetState(WIDGET_STATES.EMPTY);
      } else {
        setData(result);
        setWidgetState(WIDGET_STATES.SUCCESS);
      }
    } catch (err) {
      setError(err.message);
      setWidgetState(WIDGET_STATES.ERROR);
    }
  }

  // Object lookup for widget state rendering
  const stateContent = {
    [WIDGET_STATES.IDLE]: (
      <div className="widget__idle">
        <p>Click to load data</p>
        <button className="btn btn--primary" onClick={handleLoad}>Load</button>
      </div>
    ),
    [WIDGET_STATES.LOADING]: (
      <div className="widget__loading">
        <div className="spinner" aria-label="Loading..." />
        <p>Fetching data...</p>
      </div>
    ),
    [WIDGET_STATES.SUCCESS]: (
      <div className="widget__success">
        {renderContent(data)}
      </div>
    ),
    [WIDGET_STATES.ERROR]: (
      <div className="widget__error">
        <p>❌ {error}</p>
        <button className="btn btn--outline" onClick={handleLoad}>Retry</button>
      </div>
    ),
    [WIDGET_STATES.EMPTY]: (
      <div className="widget__empty">
        <p>No data available.</p>
        <button className="btn btn--ghost" onClick={handleLoad}>Refresh</button>
      </div>
    ),
  };

  return (
    <div className={clsx('widget', `widget--${widgetState}`)}>
      <div className="widget__header">
        <h3>{title}</h3>
        {widgetState === WIDGET_STATES.SUCCESS && (
          <button
            className="btn btn--icon"
            onClick={handleLoad}
            aria-label="Refresh"
          >
            🔄
          </button>
        )}
      </div>
      <div className="widget__body">
        {stateContent[widgetState]}
      </div>
    </div>
  );
}

export default DashboardWidget;

// Usage
<DashboardWidget
  title="Recent Orders"
  fetchData={() => fetch('/api/orders/recent').then(r => r.json())}
  renderContent={(orders) => (
    <ul>
      {orders.map(order => (
        <li key={order.id}>#{order.id} — ₹{order.total}</li>
      ))}
    </ul>
  )}
/>
```

---

### Example 3 — Advanced: Role-Based UI System

```jsx
// File: systems/RBACSystem.jsx
import { createContext, useContext } from 'react';

// ─── Permissions System ──────────────────────────────────
const PERMISSIONS = {
  // Format: 'resource:action'
  'posts:read':    ['viewer', 'editor', 'admin'],
  'posts:create':  ['editor', 'admin'],
  'posts:delete':  ['admin'],
  'users:manage':  ['admin'],
  'billing:view':  ['admin'],
  'comments:post': ['viewer', 'editor', 'admin'],
};

function hasPermission(userRole, permission) {
  const allowedRoles = PERMISSIONS[permission] ?? [];
  return allowedRoles.includes(userRole);
}

// ─── Context ──────────────────────────────────────────────
const AuthContext = createContext(null);

function AuthProvider({ user, children }) {
  function can(permission) {
    return hasPermission(user?.role, permission);
  }

  return (
    <AuthContext.Provider value={{ user, can }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}

// ─── Guard Components ────────────────────────────────────
function Can({ do: permission, fallback = null, children }) {
  const { can } = useAuth();
  return can(permission) ? children : fallback;
}

function Cannot({ do: permission, children }) {
  const { can } = useAuth();
  return !can(permission) ? children : null;
}

// ─── Usage ───────────────────────────────────────────────
function BlogPost({ post }) {
  const { user } = useAuth();

  return (
    <article className="blog-post">
      <h1>{post.title}</h1>
      <p>{post.content}</p>

      {/* Only editors and admins see the edit button */}
      <Can do="posts:create">
        <a href={`/posts/${post.id}/edit`} className="btn btn--outline">
          Edit Post
        </a>
      </Can>

      {/* Only admins see delete */}
      <Can
        do="posts:delete"
        fallback={
          <Cannot do="posts:delete">
            <p className="hint">Contact an admin to delete posts.</p>
          </Cannot>
        }
      >
        <button className="btn btn--danger" onClick={() => deletePost(post.id)}>
          Delete Post
        </button>
      </Can>

      {/* Comments visible to all with read permission */}
      <Can do="posts:read">
        <CommentSection postId={post.id} />
      </Can>

      {/* Viewers see an upgrade prompt */}
      <Cannot do="posts:create">
        <div className="upgrade-banner">
          <p>Want to write posts? <a href="/upgrade">Upgrade to Editor</a></p>
        </div>
      </Cannot>
    </article>
  );
}

export { AuthProvider, useAuth, Can, Cannot };
```

---

## 15. Real-World Use Cases

### 15.1 Authentication-Based Routing

```jsx
function AppRouter() {
  const { user, isLoading } = useAuth();

  // Show nothing until auth state is known
  if (isLoading) return <SplashScreen />;

  return (
    <Routes>
      {/* Public routes — always accessible */}
      <Route path="/login"  element={
        user ? <Navigate to="/dashboard" /> : <LoginPage />
      } />
      <Route path="/signup" element={
        user ? <Navigate to="/dashboard" /> : <SignupPage />
      } />

      {/* Protected routes — redirect if not authenticated */}
      <Route path="/dashboard" element={
        user ? <Dashboard /> : <Navigate to="/login" state={{ from: '/dashboard' }} />
      } />

      {/* Admin routes — redirect if not admin */}
      <Route path="/admin" element={
        !user ? <Navigate to="/login" /> :
        user.role !== 'admin' ? <AccessDenied /> :
        <AdminPanel />
      } />

      <Route path="*" element={<NotFoundPage />} />
    </Routes>
  );
}
```

### 15.2 Progressive Disclosure Form

```jsx
function ShippingForm() {
  const [sameAsBilling, setSameAsBilling] = useState(true);
  const [shippingMethod, setShippingMethod] = useState('standard');
  const [addGiftMessage, setAddGiftMessage] = useState(false);

  return (
    <form>
      <h2>Shipping Details</h2>

      {/* Toggle billing address */}
      <label>
        <input
          type="checkbox"
          checked={sameAsBilling}
          onChange={e => setSameAsBilling(e.target.checked)}
        />
        Same as billing address
      </label>

      {/* Only show shipping address fields if different from billing */}
      {!sameAsBilling && (
        <fieldset>
          <legend>Shipping Address</legend>
          <input name="street" placeholder="Street address" />
          <input name="city" placeholder="City" />
          <input name="pincode" placeholder="PIN Code" />
        </fieldset>
      )}

      {/* Shipping method */}
      <fieldset>
        <legend>Shipping Method</legend>
        {['standard', 'express', 'overnight'].map(method => (
          <label key={method}>
            <input
              type="radio"
              value={method}
              checked={shippingMethod === method}
              onChange={e => setShippingMethod(e.target.value)}
            />
            {method.charAt(0).toUpperCase() + method.slice(1)}
          </label>
        ))}
      </fieldset>

      {/* Extra cost warning for overnight */}
      {shippingMethod === 'overnight' && (
        <p className="notice">⚡ Overnight shipping adds ₹499 to your order.</p>
      )}

      {/* Gift message option */}
      <label>
        <input
          type="checkbox"
          checked={addGiftMessage}
          onChange={e => setAddGiftMessage(e.target.checked)}
        />
        Add gift message
      </label>

      {addGiftMessage && (
        <textarea
          placeholder="Write your gift message here..."
          maxLength={200}
          rows={3}
        />
      )}
    </form>
  );
}
```

### 15.3 Notification Toast System

```jsx
function ToastContainer({ toasts, onDismiss }) {
  if (toasts.length === 0) return null;

  return (
    <div className="toast-container" aria-live="polite" aria-atomic="false">
      {toasts.map(toast => (
        <div
          key={toast.id}
          className={`toast toast--${toast.type}`}
          role={toast.type === 'error' ? 'alert' : 'status'}
        >
          <span className="toast__icon">
            {toast.type === 'success' && '✅'}
            {toast.type === 'error' && '❌'}
            {toast.type === 'warning' && '⚠️'}
            {toast.type === 'info' && 'ℹ️'}
          </span>

          <div className="toast__content">
            {toast.title && <strong>{toast.title}</strong>}
            <p>{toast.message}</p>
          </div>

          {toast.action && (
            <button
              className="toast__action"
              onClick={() => {
                toast.action.onClick();
                onDismiss(toast.id);
              }}
            >
              {toast.action.label}
            </button>
          )}

          <button
            className="toast__close"
            onClick={() => onDismiss(toast.id)}
            aria-label="Dismiss notification"
          >
            ✕
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## 16. Best Practices

### 16.1 Use Early Returns for Guard Clauses

Handle loading, error, and empty states with early returns at the top of your component. The main render should be the "happy path".

```jsx
// ✅ Clean pattern
function ProductPage({ productId }) {
  const { data, isLoading, error } = useFetch(`/api/products/${productId}`);

  if (isLoading) return <ProductSkeleton />;
  if (error)     return <ErrorPage message={error} />;
  if (!data)     return null;

  return <ProductDetails product={data} />;
}
```

### 16.2 Avoid Deeply Nested Ternaries

```jsx
// ❌ Deeply nested — unreadable
{a ? (b ? <C /> : <D />) : (e ? <F /> : <G />)}

// ✅ Extract to a variable or function
function renderContent() {
  if (a && b)  return <C />;
  if (a && !b) return <D />;
  if (!a && e) return <F />;
  return <G />;
}
return <div>{renderContent()}</div>;
```

### 16.3 Fix the && Zero Bug Proactively

Always use explicit boolean comparisons when using `&&` with potentially zero/falsy values:

```jsx
// ✅ Explicit comparisons
{count > 0 && <Badge count={count} />}
{items.length > 0 && <List items={items} />}
{typeof value === 'number' && <Display value={value} />}
```

### 16.4 Extract Complex Conditions to Named Variables

```jsx
// ❌ Inline condition — hard to understand
{user && user.subscription && user.subscription.tier === 'premium' && !user.subscription.isCancelled && <PremiumBadge />}

// ✅ Named variable — self-documenting
const isActivePremiumUser =
  user?.subscription?.tier === 'premium' &&
  !user?.subscription?.isCancelled;

{isActivePremiumUser && <PremiumBadge />}
```

### 16.5 Use clsx for Conditional CSS

```jsx
// ❌ Manual string concatenation — messy
className={`btn ${variant ? `btn--${variant}` : ''} ${isActive ? 'btn--active' : ''}`}

// ✅ clsx — clean and composable
className={clsx('btn', variant && `btn--${variant}`, isActive && 'btn--active')}
```

### 16.6 Use Component-Based Patterns for Repeated Conditions

If you find yourself writing the same conditional check in many places, extract it into a component:

```jsx
// ❌ Same condition repeated in many places
{user?.role === 'admin' && <AdminButton />}
{user?.role === 'admin' && <AdminStats />}
{user?.role === 'admin' && <AdminUserList />}

// ✅ Encapsulate in a component
<AdminOnly>
  <AdminButton />
  <AdminStats />
  <AdminUserList />
</AdminOnly>
```

---

## 17. Common Mistakes

### Mistake 1: The && Zero Bug

```jsx
// ❌ Renders "0" when notifications array is empty
{notifications.length && <NotificationBell count={notifications.length} />}

// ✅ Explicit boolean check
{notifications.length > 0 && <NotificationBell count={notifications.length} />}
```

### Mistake 2: Deeply Nested Ternaries

```jsx
// ❌ Completely unreadable
return isA ? <A /> : isB ? <B /> : isC ? <C /> : <D />;

// ✅ Use early returns or object lookup
if (isA) return <A />;
if (isB) return <B />;
if (isC) return <C />;
return <D />;
```

### Mistake 3: Overusing && for Two-Way Conditions

```jsx
// ❌ Two separate && for a two-way condition — confusing
{isLoggedIn && <Dashboard />}
{!isLoggedIn && <LoginPage />}

// ✅ One ternary — explicit two-way condition
{isLoggedIn ? <Dashboard /> : <LoginPage />}
```

### Mistake 4: Conditional Hook Calls

```jsx
// ❌ ILLEGAL — hooks must not be conditional
function BadComponent({ shouldFetch }) {
  if (shouldFetch) {
    const data = useFetchData(); // Hook inside condition — breaks Rules of Hooks
  }
}

// ✅ Call hooks unconditionally, condition inside the hook or its effect
function GoodComponent({ shouldFetch }) {
  const data = useFetchData(shouldFetch ? url : null); // hook decides what to do
}
```

### Mistake 5: Not Handling All States

```jsx
// ❌ Only handles success — what about loading and error?
function UserList() {
  const { users } = useUsers();
  return users.map(user => <UserCard key={user.id} user={user} />);
}

// ✅ Handle all async states
function UserList() {
  const { users, isLoading, error } = useUsers();
  if (isLoading) return <Spinner />;
  if (error)     return <ErrorMessage error={error} />;
  if (!users?.length) return <EmptyState />;
  return users.map(user => <UserCard key={user.id} user={user} />);
}
```

### Mistake 6: Rendering Heavy Components Conditionally in a Hot Path

```jsx
// ❌ HeavyChart mounts/unmounts on every tiny interaction
function Dashboard() {
  const [activeTab, setActiveTab] = useState('overview');
  return (
    <div>
      <Tabs onChange={setActiveTab} />
      {activeTab === 'charts' && <HeavyChart />} {/* Destroyed and recreated on tab switch */}
    </div>
  );
}

// ✅ Keep mounted, use CSS to hide
{/* Or lazy-load and keep mounted after first load */}
function Dashboard() {
  const [activeTab, setActiveTab] = useState('overview');
  const [chartLoaded, setChartLoaded] = useState(false);

  return (
    <div>
      <Tabs onChange={(tab) => {
        setActiveTab(tab);
        if (tab === 'charts') setChartLoaded(true);
      }} />
      {chartLoaded && (
        <div style={{ display: activeTab === 'charts' ? 'block' : 'none' }}>
          <HeavyChart />
        </div>
      )}
    </div>
  );
}
```

### Mistake 7: Forgetting Empty State

```jsx
// ❌ When orders is [], renders an empty ul — ugly
function OrderList({ orders }) {
  return (
    <ul>
      {orders.map(order => <OrderItem key={order.id} order={order} />)}
    </ul>
  );
}

// ✅ Always handle empty state explicitly
function OrderList({ orders }) {
  if (orders.length === 0) {
    return <EmptyState message="No orders yet." ctaText="Start shopping" ctaHref="/shop" />;
  }
  return (
    <ul>
      {orders.map(order => <OrderItem key={order.id} order={order} />)}
    </ul>
  );
}
```

---

## 18. Performance Considerations

### 18.1 Avoid Expensive Computations in Conditional Render Paths

```jsx
// ❌ processData runs even when !isReady
{isReady && <Chart data={processData(rawData)} />}

// ✅ Memoize the expensive computation
const processedData = useMemo(() => processData(rawData), [rawData]);
{isReady && <Chart data={processedData} />}
```

### 18.2 Use React.lazy for Conditionally Rendered Heavy Components

```jsx
const HeavyModal = lazy(() => import('./HeavyModal'));

function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsModalOpen(true)}>Open</button>
      {isModalOpen && (
        <Suspense fallback={<ModalSkeleton />}>
          <HeavyModal onClose={() => setIsModalOpen(false)} />
        </Suspense>
      )}
    </>
  );
}
```

### 18.3 Keep Conditional Components Stable

When conditions change the component type at the same position in the tree, React fully unmounts and remounts:

```jsx
// ❌ React unmounts/remounts whenever condition changes — state is lost
{isAdmin ? <AdminInput defaultValue={value} /> : <UserInput defaultValue={value} />}

// ✅ Same component, conditional props
<Input isAdmin={isAdmin} defaultValue={value} />
```

---

## 19. Interview Questions

### Q1: What is conditional rendering in React and what JavaScript features make it possible?

**Answer:** Conditional rendering is displaying different UI based on conditions. It's possible because JSX is JavaScript — you can use any JS expression to determine what gets rendered. The main patterns are: `if/else` statements for full component-level conditions, the ternary operator (`? :`) for inline two-way choices, the logical AND (`&&`) for "show only if" cases, the logical OR (`||`) for fallback values, and early returns for guard clauses. React renders whatever your component function returns, so controlling the return value controls what appears on screen.

---

### Q2: What is the && zero bug and how do you fix it?

**Answer:** When you use `&&` in JSX with a left operand that evaluates to the number `0`, JavaScript short-circuits and returns `0` — not `false`. React treats `0` as a renderable value (unlike `false`, `null`, or `undefined`) and prints it on screen. Example: `{items.length && <List />}` renders `"0"` when `items` is empty. Fixes: use an explicit boolean comparison (`items.length > 0 && <List />`), double-negate (`!!items.length && <List />`), or use a ternary (`items.length ? <List /> : null`).

---

### Q3: What is the difference between conditional rendering and toggling CSS display?

**Answer:** Conditional rendering mounts/unmounts the component — when it renders false/null, React removes it from the DOM entirely, destroying its state and running cleanup effects. CSS `display: none` keeps the component mounted but visually hidden — state is preserved, effects keep running. Use conditional rendering when you want proper cleanup, state reset, or to keep the DOM clean. Use CSS hiding when state needs to be preserved across show/hide cycles, when animation is needed, or when the component is expensive to mount repeatedly.

---

### Q4: What are early returns in React components and why are they useful?

**Answer:** Early returns are `return` statements at the top of a component that exit before the main JSX, used to handle edge cases like loading, error, and empty states. They're useful because they eliminate deeply nested ternary or conditional blocks, making components flat and readable. The pattern is: list guard clauses (loading → error → empty → invalid data) at the top, then write the main "happy path" render at the bottom without any nesting.

---

### Q5: When should you use a ternary vs the && operator for conditional rendering?

**Answer:** Use a ternary (`? :`) when you have two distinct alternatives — show A or show B. It's explicit about both cases. Use `&&` when you only want to show something optionally with no alternative — "show this or nothing". Never use `&&` with potentially falsy non-boolean values like numbers or strings without an explicit comparison, due to the zero bug. For more than two cases, prefer `if/else` with early returns, a `switch` statement, or an object lookup map.

---

### Q6: What is an object/map lookup pattern in conditional rendering?

**Answer:** An object lookup replaces `if/else` chains or `switch` statements by storing the different UI variants in a JavaScript object keyed by the condition value. For example, storing components or configuration objects keyed by a `status` string, then accessing the object with `obj[status]`. It's more declarative, easier to extend (just add a new key), and avoids deeply nested logic. It's especially powerful for mapping enum-like string values to components or configuration.

---

### Q7: How do you render nothing in React? What values cause React to render nothing?

**Answer:** Return or include `null`, `undefined`, or `false` in JSX to render nothing. A component can `return null` to produce no DOM output while still being mounted (still has state and can run effects). In JSX expressions, `null`, `undefined`, `false`, and `''` (empty string) all render nothing. Important exception: the number `0` renders as the character "0" on screen, which is the source of the common `&&` zero bug.

---

### Q8: What are component-based conditional patterns? Give examples.

**Answer:** Component-based patterns encapsulate conditional logic in reusable components rather than scattering it through JSX. Examples: (1) `<CanAccess requiredRole="admin">` — wraps children and renders them only if the user has the required role; (2) `<Feature flag="new_checkout">` — renders children only if a feature flag is enabled; (3) `<Show when={isLoggedIn} fallback={<Login />}>` — a generic conditional wrapper; (4) `<RequireAuth>` — redirects to login if not authenticated. These make conditions reusable, testable, and centralized.

---

### Q9: What is the difference between `||` and `??` for fallback rendering?

**Answer:** `||` (OR) returns the right side when the left is any falsy value — `false`, `0`, `''`, `null`, `undefined`, `NaN`. `??` (nullish coalescing) returns the right side only when the left is `null` or `undefined` — it treats `0`, `false`, and `''` as valid values. For UI fallbacks, `??` is often safer: `price ?? 'Free'` correctly shows `0` as "0", not "Free". Use `||` when you want any falsy value to trigger the fallback.

---

### Q10: How do you handle multiple async states (loading, error, empty, success) cleanly in a component?

**Answer:** The cleanest pattern is the **early return / guard clause** approach: at the top of the component, return the loading state first, then the error state, then the empty state, then the "invalid data" state. The main JSX at the bottom is the success state — it runs only when all checks pass, with no nesting required. Some teams use a state machine approach (storing a status string like `'idle' | 'loading' | 'success' | 'error'`) combined with an object lookup to render each state's component — this is especially clean for complex widgets.

---

## 20. Practice Tasks

### Task 1 — Beginner: Weather Widget

Build a `WeatherWidget` component with four visual states.

**Requirements:**
- State: `status` can be `'idle'`, `'loading'`, `'success'`, or `'error'`
- `'idle'` state: Show a button "Get Weather" with a cloud icon
- `'loading'` state: Show an animated spinner and "Fetching weather..."
- `'error'` state: Show an error icon, the error message, and a "Try Again" button
- `'success'` state: Show temperature, city name, weather description, and an appropriate weather icon (☀️ 🌧️ ⛅ ❄️) based on the description
- Simulate API call with `setTimeout` (1.5 second delay)
- Randomly succeed or fail (50/50) to test both states
- Use the object lookup pattern to map weather conditions to icons
- Use `clsx` to apply different border colors per state

---

### Task 2 — Intermediate: Permissions-Based Navigation Menu

Build a `NavigationMenu` component driven by a permissions system.

**Requirements:**
- Menu items: Home, Dashboard, Products, Analytics, Users, Settings, Admin Panel, Billing
- Each menu item has a required permission: `public`, `user`, `editor`, `admin`
- Accept a `userRole` prop: `'guest'`, `'user'`, `'editor'`, `'admin'`
- Only show menu items the current role has access to
- For the currently active route (accept `activePath` prop), highlight the item
- Guest users see a "Login to unlock more" banner below the menu
- Admin users see a red "Admin" badge next to their restricted items
- If a user tries to navigate to a restricted route (pass as prop), show an "Access Denied" screen instead of the content
- Extract the permission logic into a `usePermissions(role)` custom hook

---

### Task 3 — Advanced: Multi-Step Checkout Flow

Build a complete `CheckoutFlow` component with full conditional rendering at every step.

**Requirements:**
- 4 steps: Cart Review → Shipping → Payment → Confirmation
- Each step has its own component with different content
- Conditionally show/hide the "Back" button on step 1
- Conditionally show "Place Order" vs "Next" on the last vs other steps
- If cart is empty on step 1, show an empty state with a "Continue Shopping" button instead of proceeding
- If user is not logged in (prop), show a login prompt on step 2 instead of the shipping form
- Payment step: conditionally show different form fields based on selected payment method (Card shows card number/CVV/expiry; UPI shows UPI ID; Net Banking shows bank dropdown)
- Confirmation step: show different messages based on payment success/failure simulation
- Order summary sidebar: conditionally show discount row only if coupon is applied
- Use early returns for guard clauses in each step component
- Show a step progress indicator that conditionally marks completed steps with a checkmark

---

## 21. Summary

### Key Takeaways

| Pattern | Best Used When |
|---------|---------------|
| `if/else` | Full component returns, multiple branches, complex logic |
| Ternary `? :` | Two alternatives inline in JSX |
| `&&` | Optional content with no fallback — use explicit boolean check |
| `\|\|` and `??` | Fallback values — `??` is safer for 0 and false |
| Early return | Guard clauses — loading, error, empty, invalid states |
| `switch` | Multiple cases based on a single value |
| Object lookup | Enum-based conditions, component maps, status configs |
| Component-based | Reusable, repeated conditions — auth guards, feature flags |
| `return null` | Hide component completely — no DOM output |
| CSS display:none | Preserve state, animation needs, expensive remounting |

### Conditional Rendering Decision Tree

```
How many branches?
  ├── 1 branch (show or nothing)
  │     └── Use && (with explicit boolean check for non-boolean values)
  │
  ├── 2 branches (A or B)
  │     └── Use ternary ? :
  │
  ├── 3+ branches based on ONE value (status, role, step)
  │     ├── Simple → switch statement
  │     └── Maps to components/config → object lookup map
  │
  └── Multiple guard clauses (loading, error, empty)
        └── Early returns at component top

Is the condition reused in many places?
  └── YES → Extract to a wrapper component (Can, Show, Feature, RequireAuth)
```

### The Zero Bug — Quick Reference

```jsx
// These all render nothing (safe with &&):
null && <X />        // → null
false && <X />       // → false (React ignores)
undefined && <X />   // → undefined (React ignores)

// This renders "0" on screen (dangerous!):
0 && <X />           // → 0 (React renders it!)

// Fix — always use explicit boolean comparison:
count > 0 && <X />   // ✅ safe
!!count && <X />     // ✅ safe
Boolean(count) && <X /> // ✅ safe
```

---

> **Next Topic:** `08-lists-and-keys.md` — Rendering dynamic lists in React, the critical role of the key prop, performance implications, and advanced list manipulation patterns.
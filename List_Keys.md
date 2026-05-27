# 08 — Lists & Keys in React (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** Lists & Keys
> **Level:** Beginner → Advanced
> **Prerequisites:** JSX (Module 02), State & useState (Module 04), Conditional Rendering (Module 07)

---

## Table of Contents

1. [What are Lists in React?](#1-what-are-lists-in-react)
2. [How React Renders Lists Internally](#2-how-react-renders-lists-internally)
3. [The key Prop — Deep Dive](#3-the-key-prop--deep-dive)
4. [Rendering Lists with .map()](#4-rendering-lists-with-map)
5. [Filtering and Transforming Lists](#5-filtering-and-transforming-lists)
6. [Nested Lists](#6-nested-lists)
7. [Keys in Depth — What Makes a Good Key?](#7-keys-in-depth--what-makes-a-good-key)
8. [List State Management Patterns](#8-list-state-management-patterns)
9. [Virtualization — Rendering Large Lists](#9-virtualization--rendering-large-lists)
10. [Code Examples (Beginner → Advanced)](#10-code-examples-beginner--advanced)
11. [Real-World Use Cases](#11-real-world-use-cases)
12. [Best Practices](#12-best-practices)
13. [Common Mistakes](#13-common-mistakes)
14. [Performance Considerations](#14-performance-considerations)
15. [Interview Questions](#15-interview-questions)
16. [Practice Tasks](#16-practice-tasks)
17. [Summary](#17-summary)

---

## 1. What are Lists in React?

### Simple Explanation

A **list** in React is a collection of similar UI elements rendered from an array of data. Product grids, comment feeds, todo items, dropdown options, navigation links — almost every real app renders lists.

React gives you the full power of JavaScript array methods to create, filter, sort, and transform lists before rendering them.

```jsx
// Simplest possible list
const fruits = ['Apple', 'Banana', 'Mango'];

function FruitList() {
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}
```

### Technical Explanation

In JSX, you can embed **any JavaScript array** of React elements as children. React flattens the array and renders each element in order. The `.map()` method is the primary tool because it transforms every item in a data array into a React element, returning a new array of elements.

```jsx
// JSX accepts arrays of elements directly
const elements = [<li key="1">One</li>, <li key="2">Two</li>, <li key="3">Three</li>];

function List() {
  return <ul>{elements}</ul>; // React renders all three li elements
}
```

---

## 2. How React Renders Lists Internally

### The Reconciliation Algorithm and Lists

React uses a **diffing algorithm** (part of its reconciler called "Fiber") to compare the previous render output with the new render output. For lists, this is especially important.

When a list changes (item added, removed, or reordered), React needs to figure out the minimum number of DOM operations to update the UI. This is where the `key` prop becomes critical.

### Without Keys — The Problem

Imagine React renders this list:

```
Render 1:   [<li>Alice</li>, <li>Bob</li>, <li>Charlie</li>]
```

Now Alice is removed:

```
Render 2:   [<li>Bob</li>, <li>Charlie</li>]
```

Without keys, React compares by **position**:
- Position 0: "Alice" → "Bob" — React updates text content
- Position 1: "Bob" → "Charlie" — React updates text content
- Position 2: "Charlie" → nothing — React removes this element

React made **two updates and one removal** when it could have made **one removal**. Worse, if items have internal state (like a checked checkbox), React assigns that state to the wrong item.

### With Keys — The Solution

With unique keys, React compares by **identity**, not position:
- Key "alice": present in render 1, missing in render 2 → **remove**
- Key "bob": present in both → **keep, no update needed**
- Key "charlie": present in both → **keep, no update needed**

React made **one removal** — exactly the minimum work.

### The Fiber Reconciler's List Diffing Strategy

React's diffing for lists works in two passes:

```
Pass 1 — Scan forward from start:
  Compare old[0] with new[0] by key — same? update. different? stop.

Pass 2 — Scan backward from end:
  Compare old[last] with new[last] by key — same? update. different? stop.

Remaining middle section:
  Build a key→index map of remaining old items
  For each remaining new item, look it up in the old map
    Found? → move or update
    Not found? → create new
  Items left in old map that weren't matched → delete
```

This is why inserting at the end is cheaper than inserting at the beginning — the second pass catches it immediately.

---

## 3. The key Prop — Deep Dive

### What is a key?

The `key` prop is a **special React attribute** (not a regular prop) that gives each list item a stable identity. React uses it to track items across renders.

```jsx
// key is a special prop — it's NOT accessible inside the component via props
function UserItem({ user }) {
  console.log(props.key); // undefined! key is consumed by React, not passed down
  return <li>{user.name}</li>;
}

// You must pass id separately if you need it inside the component
<UserItem key={user.id} id={user.id} user={user} />
```

### Rules of Keys

1. Keys must be **unique among siblings** (not globally)
2. Keys must be **stable** — the same item should have the same key across renders
3. Keys must be **strings or numbers**
4. Keys do **not** need to be globally unique — same key in different lists is fine
5. Keys are **not passed** to components as props

```jsx
// ✅ Same key in different lists is fine
<ul>
  <li key="1">Item 1 in List A</li>  {/* key="1" */}
</ul>
<ul>
  <li key="1">Item 1 in List B</li>  {/* key="1" — completely separate, no conflict */}
</ul>
```

### What React Does When a Key Changes

When an item's key changes, React treats it as a completely **different element**:
- Old element is **unmounted** (state destroyed, cleanup effects run)
- New element is **mounted** (fresh state, setup effects run)

This behavior can actually be used intentionally to **reset a component's state**:

```jsx
// Force UserProfile to fully reset when userId changes
// (re-mount instead of update)
<UserProfile key={userId} userId={userId} />
```

---

## 4. Rendering Lists with .map()

### Basic .map() Pattern

```jsx
const users = [
  { id: 1, name: 'Priya Sharma', role: 'Admin' },
  { id: 2, name: 'Rahul Verma', role: 'Editor' },
  { id: 3, name: 'Sneha Patel', role: 'Viewer' },
];

function UserList() {
  return (
    <ul className="user-list">
      {users.map(user => (
        <li key={user.id} className="user-list__item">
          <strong>{user.name}</strong>
          <span>{user.role}</span>
        </li>
      ))}
    </ul>
  );
}
```

### .map() with Index (When Acceptable)

```jsx
// Using index as key — acceptable for STATIC, NEVER-REORDERED lists
const steps = ['Install Node', 'Create React App', 'Start Development Server'];

function SetupGuide() {
  return (
    <ol>
      {steps.map((step, index) => (
        <li key={index}>{step}</li>  {/* OK here — list never changes */}
      ))}
    </ol>
  );
}
```

### Extracting the List Item Component

```jsx
// ❌ Inline — JSX gets cluttered for complex items
function ProductGrid({ products }) {
  return (
    <div className="grid">
      {products.map(product => (
        <div key={product.id} className="product-card">
          <img src={product.image} alt={product.name} />
          <h3>{product.name}</h3>
          <p>₹{product.price}</p>
          <button>Add to Cart</button>
        </div>
      ))}
    </div>
  );
}

// ✅ Extracted component — clean and reusable
function ProductCard({ product }) {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>₹{product.price}</p>
      <button>Add to Cart</button>
    </div>
  );
}

function ProductGrid({ products }) {
  return (
    <div className="grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Where the key Goes

The key belongs on the **outermost element returned by .map()**, not nested inside:

```jsx
// ✅ key on outermost element returned by map
{users.map(user => (
  <li key={user.id}>
    <Avatar src={user.avatar} />
    <span>{user.name}</span>
  </li>
))}

// ✅ key on the component when a component is the outermost
{users.map(user => (
  <UserCard key={user.id} user={user} />
))}

// ❌ key inside nested element — wrong placement
{users.map(user => (
  <li>
    <div key={user.id}> {/* Wrong! Key should be on <li> */}
      {user.name}
    </div>
  </li>
))}
```

### .map() with Fragment and Key

When you need to return multiple sibling elements per item without a wrapper, use `React.Fragment` with a `key` (shorthand `<>` does NOT support `key`):

```jsx
import { Fragment } from 'react';

function DefinitionList({ terms }) {
  return (
    <dl>
      {terms.map(term => (
        // Fragment with key — required when returning multiple siblings
        <Fragment key={term.id}>
          <dt>{term.word}</dt>
          <dd>{term.definition}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

---

## 5. Filtering and Transforming Lists

### Filtering with .filter()

```jsx
function ActiveUserList({ users }) {
  const activeUsers = users.filter(user => user.isActive);

  if (activeUsers.length === 0) {
    return <p>No active users.</p>;
  }

  return (
    <ul>
      {activeUsers.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Chaining .filter() and .map()

```jsx
function FilteredProductGrid({ products, category, maxPrice, sortBy }) {
  const displayProducts = products
    .filter(p => category === 'all' || p.category === category)
    .filter(p => p.price <= maxPrice)
    .sort((a, b) => {
      if (sortBy === 'price-asc')  return a.price - b.price;
      if (sortBy === 'price-desc') return b.price - a.price;
      if (sortBy === 'name')       return a.name.localeCompare(b.name);
      return 0;
    })
    .map(product => (
      <ProductCard key={product.id} product={product} />
    ));

  return (
    <div className="product-grid">
      {displayProducts.length > 0
        ? displayProducts
        : <EmptyState message="No products match your filters." />
      }
    </div>
  );
}
```

### .reduce() for Grouped Lists

```jsx
function GroupedContactList({ contacts }) {
  // Group contacts by first letter
  const grouped = contacts.reduce((acc, contact) => {
    const letter = contact.name[0].toUpperCase();
    if (!acc[letter]) acc[letter] = [];
    acc[letter].push(contact);
    return acc;
  }, {});

  const sortedLetters = Object.keys(grouped).sort();

  return (
    <div className="contact-list">
      {sortedLetters.map(letter => (
        <div key={letter} className="contact-group">
          <h3 className="contact-group__letter">{letter}</h3>
          <ul>
            {grouped[letter].map(contact => (
              <li key={contact.id} className="contact-item">
                <Avatar name={contact.name} />
                <div>
                  <strong>{contact.name}</strong>
                  <span>{contact.phone}</span>
                </div>
              </li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

### flatMap for Normalized Data

```jsx
// Data: categories with nested products
const categories = [
  { id: 1, name: 'Electronics', products: [{ id: 101, name: 'Phone' }, { id: 102, name: 'Tablet' }] },
  { id: 2, name: 'Clothing',    products: [{ id: 201, name: 'Shirt' }, { id: 202, name: 'Jeans'  }] },
];

function AllProductsList() {
  // flatMap — map then flatten one level deep
  const allProducts = categories.flatMap(cat =>
    cat.products.map(product => ({ ...product, categoryName: cat.name }))
  );

  return (
    <ul>
      {allProducts.map(product => (
        <li key={product.id}>
          [{product.categoryName}] {product.name}
        </li>
      ))}
    </ul>
  );
}
```

---

## 6. Nested Lists

### Basic Nested List

```jsx
function CategoryMenu({ categories }) {
  return (
    <nav>
      <ul className="menu">
        {categories.map(category => (
          <li key={category.id} className="menu__item">
            <a href={`/category/${category.slug}`}>{category.name}</a>

            {/* Nested sub-categories */}
            {category.children && category.children.length > 0 && (
              <ul className="menu__submenu">
                {category.children.map(child => (
                  <li key={child.id} className="menu__subitem">
                    <a href={`/category/${child.slug}`}>{child.name}</a>
                  </li>
                ))}
              </ul>
            )}
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

### Recursive List (Arbitrary Depth)

```jsx
// Recursive component for tree data of any depth
function TreeNode({ node, depth = 0 }) {
  const [isExpanded, setIsExpanded] = useState(depth < 2); // expand first 2 levels

  const hasChildren = node.children && node.children.length > 0;

  return (
    <li
      className="tree-node"
      style={{ paddingLeft: depth * 20 }}
    >
      <div
        className="tree-node__label"
        onClick={() => hasChildren && setIsExpanded(prev => !prev)}
        style={{ cursor: hasChildren ? 'pointer' : 'default' }}
      >
        {hasChildren && (
          <span className="tree-node__toggle">
            {isExpanded ? '▼' : '▶'}
          </span>
        )}
        <span className="tree-node__name">{node.name}</span>
        {node.badge && <span className="badge">{node.badge}</span>}
      </div>

      {hasChildren && isExpanded && (
        <ul className="tree-node__children">
          {node.children.map(child => (
            <TreeNode
              key={child.id}   // key on the component, not inside it
              node={child}
              depth={depth + 1}
            />
          ))}
        </ul>
      )}
    </li>
  );
}

function FileTree({ data }) {
  return (
    <ul className="file-tree">
      {data.map(node => (
        <TreeNode key={node.id} node={node} />
      ))}
    </ul>
  );
}
```

### Key Uniqueness in Nested Lists

```jsx
// Keys only need to be unique AMONG SIBLINGS — not globally
// These keys are all valid:

<ul>
  <li key="1">Parent 1       {/* key="1" in parent list */}
    <ul>
      <li key="1">Child 1-1</li>  {/* key="1" in child list — OK! Different list */}
      <li key="2">Child 1-2</li>
    </ul>
  </li>
  <li key="2">Parent 2       {/* key="2" in parent list */}
    <ul>
      <li key="1">Child 2-1</li>  {/* key="1" again — OK! Different list */}
    </ul>
  </li>
</ul>
```

---

## 7. Keys in Depth — What Makes a Good Key?

### The Ideal Key: Stable, Unique, from Your Data

```jsx
// ✅ Database IDs — perfect keys
{users.map(user => <UserCard key={user.id} user={user} />)}

// ✅ Unique slugs
{posts.map(post => <PostCard key={post.slug} post={post} />)}

// ✅ Compound key when no single unique field
{orderItems.map(item => (
  <OrderItem key={`${item.orderId}-${item.productId}`} item={item} />
))}

// ✅ UUID generated at creation time (not at render time)
// Store in state: { id: crypto.randomUUID(), text: '' }
{todos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
```

### When Index as Key is Acceptable

Index keys are **safe** only when ALL three conditions are true:

1. The list is **static** — items are never reordered, inserted at the middle, or removed
2. Items have **no internal state** (no input fields, checkboxes, or local component state)
3. The list is **never re-rendered differently** (not filtered or sorted)

```jsx
// ✅ Index key is fine here — static ordered list, no item state
const steps = ['Install', 'Configure', 'Deploy'];
steps.map((step, i) => <Step key={i} text={step} />)

// ❌ Index key WRONG here — list can be reordered, items have state
todos.map((todo, i) => <TodoItem key={i} todo={todo} />)
// Reordering visually moves items but React reuses DOM nodes incorrectly
```

### Why Index Keys Break Stateful Lists — Illustrated

```
INITIAL STATE:
  key=0: <input value="Buy milk" />    [0] → Alice   (has typed "Buy milk")
  key=1: <input value="" />            [1] → Bob
  key=2: <input value="" />            [2] → Charlie

DELETE ALICE (index 0):
  New array: [Bob, Charlie]

  key=0: <input value="Buy milk" />    [0] → Bob     ← Bug! Bob gets Alice's state
  key=1: <input value="" />            [1] → Charlie

React matched by position (key), not by item identity.
Bob now has Alice's typed text in its input.
```

### Generating Stable IDs for New Items

```jsx
// ✅ Generate ID when creating the item — store in state
function TodoApp() {
  const [todos, setTodos] = useState([]);

  function addTodo(text) {
    setTodos(prev => [
      ...prev,
      {
        id: crypto.randomUUID(), // stable ID generated once at creation
        text,
        completed: false,
      }
    ]);
  }

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />  // stable ID used as key
      ))}
    </ul>
  );
}
```

---

## 8. List State Management Patterns

### Adding Items

```jsx
function ShoppingList() {
  const [items, setItems] = useState([]);
  const [inputValue, setInputValue] = useState('');

  function addItem() {
    if (!inputValue.trim()) return;

    setItems(prev => [
      ...prev,
      { id: crypto.randomUUID(), name: inputValue.trim(), quantity: 1 }
    ]);
    setInputValue('');
  }

  function handleKeyDown(e) {
    if (e.key === 'Enter') addItem();
  }

  return (
    <div>
      <div className="add-item">
        <input
          value={inputValue}
          onChange={e => setInputValue(e.target.value)}
          onKeyDown={handleKeyDown}
          placeholder="Add item..."
        />
        <button onClick={addItem}>Add</button>
      </div>
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.name} × {item.quantity}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Removing Items

```jsx
// Remove by id using filter
function removeItem(id) {
  setItems(prev => prev.filter(item => item.id !== id));
}

// Remove by index (when no stable id available)
function removeAt(index) {
  setItems(prev => prev.filter((_, i) => i !== index));
}
```

### Updating Items

```jsx
// Update a field of a specific item
function updateItem(id, updates) {
  setItems(prev =>
    prev.map(item =>
      item.id === id ? { ...item, ...updates } : item
    )
  );
}

// Toggle a boolean field
function toggleComplete(id) {
  setItems(prev =>
    prev.map(item =>
      item.id === id ? { ...item, completed: !item.completed } : item
    )
  );
}
```

### Reordering Items

```jsx
// Move item up in the list
function moveUp(index) {
  if (index === 0) return;
  setItems(prev => {
    const newItems = [...prev];
    [newItems[index - 1], newItems[index]] = [newItems[index], newItems[index - 1]];
    return newItems;
  });
}

// Move item down in the list
function moveDown(index) {
  setItems(prev => {
    if (index === prev.length - 1) return prev;
    const newItems = [...prev];
    [newItems[index], newItems[index + 1]] = [newItems[index + 1], newItems[index]];
    return newItems;
  });
}

// Insert at specific position
function insertAt(index, newItem) {
  setItems(prev => [
    ...prev.slice(0, index),
    newItem,
    ...prev.slice(index)
  ]);
}
```

### Bulk Operations

```jsx
function TaskManager() {
  const [tasks, setTasks] = useState([]);
  const [selectedIds, setSelectedIds] = useState(new Set());

  function toggleSelect(id) {
    setSelectedIds(prev => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  }

  function selectAll() {
    setSelectedIds(new Set(tasks.map(t => t.id)));
  }

  function clearSelection() {
    setSelectedIds(new Set());
  }

  function deleteSelected() {
    setTasks(prev => prev.filter(task => !selectedIds.has(task.id)));
    setSelectedIds(new Set());
  }

  function markSelectedComplete() {
    setTasks(prev =>
      prev.map(task =>
        selectedIds.has(task.id) ? { ...task, completed: true } : task
      )
    );
    setSelectedIds(new Set());
  }

  return (
    <div>
      <div className="bulk-actions">
        <span>{selectedIds.size} selected</span>
        <button onClick={selectAll}>Select All</button>
        <button onClick={clearSelection}>Clear</button>
        <button onClick={deleteSelected} disabled={selectedIds.size === 0}>
          Delete Selected
        </button>
        <button onClick={markSelectedComplete} disabled={selectedIds.size === 0}>
          Mark Complete
        </button>
      </div>
      <ul>
        {tasks.map(task => (
          <li key={task.id}>
            <input
              type="checkbox"
              checked={selectedIds.has(task.id)}
              onChange={() => toggleSelect(task.id)}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
              {task.title}
            </span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 9. Virtualization — Rendering Large Lists

### The Problem with Long Lists

Rendering 10,000 list items creates 10,000 DOM nodes — all at once, even if only 20 are visible on screen. This causes:
- Slow initial render
- High memory usage
- Janky scrolling

### What is Virtualization?

**Virtualization** (also called windowing) renders only the items **currently visible in the viewport** plus a small buffer above and below. As you scroll, items that leave the viewport are removed from the DOM, and new items entering are added.

```
DOM at any time ≈ 20-30 items
Data: 10,000 items

Result: Fast initial render, constant memory, smooth scrolling
```

### react-window (Lightweight)

```bash
npm install react-window
```

```jsx
import { FixedSizeList, VariableSizeList } from 'react-window';

// ─── Fixed Height Items ────────────────────────────────
function VirtualUserList({ users }) {
  const Row = ({ index, style }) => (
    // MUST apply style to the row — it controls the absolute positioning
    <div style={style} className="virtual-row">
      <img src={users[index].avatar} alt="" />
      <span>{users[index].name}</span>
      <span>{users[index].email}</span>
    </div>
  );

  return (
    <FixedSizeList
      height={600}          // visible container height (px)
      width="100%"          // container width
      itemCount={users.length}
      itemSize={72}         // height of each row (px) — fixed
    >
      {Row}
    </FixedSizeList>
  );
}

// ─── Variable Height Items ─────────────────────────────
function VirtualPostList({ posts }) {
  // Function that returns the height for each item
  function getItemSize(index) {
    return posts[index].hasImage ? 200 : 80;
  }

  const Row = ({ index, style }) => (
    <div style={style} className="post-row">
      <h3>{posts[index].title}</h3>
      {posts[index].hasImage && <img src={posts[index].image} alt="" />}
    </div>
  );

  return (
    <VariableSizeList
      height={600}
      width="100%"
      itemCount={posts.length}
      itemSize={getItemSize}
    >
      {Row}
    </VariableSizeList>
  );
}
```

### TanStack Virtual (More Flexible)

```bash
npm install @tanstack/react-virtual
```

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

function VirtualTable({ rows, columns }) {
  const parentRef = useRef(null);

  const rowVirtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 48,       // estimated row height
    overscan: 5,                  // render 5 extra rows above/below visible area
  });

  return (
    <div
      ref={parentRef}
      style={{ height: '600px', overflow: 'auto' }}
    >
      {/* Total scrollable height */}
      <div style={{ height: rowVirtualizer.getTotalSize(), position: 'relative' }}>
        {rowVirtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {/* Render the actual row content */}
            {columns.map(col => (
              <span key={col.key}>{rows[virtualRow.index][col.key]}</span>
            ))}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### When to Virtualize

| List Size | Recommendation |
|-----------|---------------|
| < 100 items | No virtualization needed |
| 100–500 items | Consider if items are complex (images, videos) |
| 500–2000 items | Virtualize if performance is noticeable |
| > 2000 items | Always virtualize |

---

## 10. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Favorite Movies List

```jsx
// File: components/MovieList/MovieList.jsx
import { useState } from 'react';

const INITIAL_MOVIES = [
  { id: 1, title: '3 Idiots',        year: 2009, genre: 'Comedy',  rating: 8.4 },
  { id: 2, title: 'Dil Chahta Hai',  year: 2001, genre: 'Drama',   rating: 8.1 },
  { id: 3, title: 'Lagaan',          year: 2001, genre: 'Drama',   rating: 8.1 },
  { id: 4, title: 'Sholay',          year: 1975, genre: 'Action',  rating: 8.2 },
  { id: 5, title: 'Andhadhun',       year: 2018, genre: 'Thriller',rating: 8.2 },
];

function MovieCard({ movie, onRemove }) {
  return (
    <li className="movie-card">
      <div className="movie-card__info">
        <h3>{movie.title}</h3>
        <p>{movie.year} · {movie.genre}</p>
        <div className="movie-card__rating">
          {'★'.repeat(Math.round(movie.rating / 2))} {movie.rating}/10
        </div>
      </div>
      <button
        className="movie-card__remove"
        onClick={() => onRemove(movie.id)}
        aria-label={`Remove ${movie.title}`}
      >
        ✕
      </button>
    </li>
  );
}

function MovieList() {
  const [movies, setMovies] = useState(INITIAL_MOVIES);
  const [sortBy, setSortBy] = useState('title');

  const sortedMovies = [...movies].sort((a, b) => {
    if (sortBy === 'title')  return a.title.localeCompare(b.title);
    if (sortBy === 'year')   return b.year - a.year;
    if (sortBy === 'rating') return b.rating - a.rating;
    return 0;
  });

  function removeMovie(id) {
    setMovies(prev => prev.filter(m => m.id !== id));
  }

  return (
    <div className="movie-list">
      <div className="movie-list__header">
        <h2>My Favourite Movies ({movies.length})</h2>
        <select
          value={sortBy}
          onChange={e => setSortBy(e.target.value)}
          aria-label="Sort movies by"
        >
          <option value="title">Sort by Title</option>
          <option value="year">Sort by Year</option>
          <option value="rating">Sort by Rating</option>
        </select>
      </div>

      {sortedMovies.length === 0 ? (
        <p className="empty-state">No movies in your list yet.</p>
      ) : (
        <ul className="movie-list__items">
          {sortedMovies.map(movie => (
            <MovieCard
              key={movie.id}
              movie={movie}
              onRemove={removeMovie}
            />
          ))}
        </ul>
      )}
    </div>
  );
}

export default MovieList;
```

---

### Example 2 — Intermediate: Paginated Data Table

```jsx
// File: components/DataTable/DataTable.jsx
import { useState, useMemo } from 'react';

const ITEMS_PER_PAGE_OPTIONS = [10, 25, 50, 100];

function DataTable({ data = [], columns = [] }) {
  const [currentPage, setCurrentPage] = useState(1);
  const [itemsPerPage, setItemsPerPage] = useState(10);
  const [sortConfig, setSortConfig] = useState({ key: null, direction: 'asc' });
  const [searchQuery, setSearchQuery] = useState('');

  // Step 1: Filter
  const filteredData = useMemo(() => {
    if (!searchQuery.trim()) return data;
    const q = searchQuery.toLowerCase();
    return data.filter(row =>
      columns.some(col =>
        String(row[col.key]).toLowerCase().includes(q)
      )
    );
  }, [data, columns, searchQuery]);

  // Step 2: Sort
  const sortedData = useMemo(() => {
    if (!sortConfig.key) return filteredData;
    return [...filteredData].sort((a, b) => {
      const aVal = a[sortConfig.key];
      const bVal = b[sortConfig.key];
      const cmp = typeof aVal === 'number'
        ? aVal - bVal
        : String(aVal).localeCompare(String(bVal));
      return sortConfig.direction === 'asc' ? cmp : -cmp;
    });
  }, [filteredData, sortConfig]);

  // Step 3: Paginate
  const totalPages = Math.ceil(sortedData.length / itemsPerPage);
  const paginatedData = useMemo(() => {
    const start = (currentPage - 1) * itemsPerPage;
    return sortedData.slice(start, start + itemsPerPage);
  }, [sortedData, currentPage, itemsPerPage]);

  function handleSort(key) {
    setSortConfig(prev => ({
      key,
      direction: prev.key === key && prev.direction === 'asc' ? 'desc' : 'asc',
    }));
    setCurrentPage(1);
  }

  function handleSearch(e) {
    setSearchQuery(e.target.value);
    setCurrentPage(1);
  }

  function handleItemsPerPage(e) {
    setItemsPerPage(Number(e.target.value));
    setCurrentPage(1);
  }

  const sortIcon = (key) => {
    if (sortConfig.key !== key) return '↕';
    return sortConfig.direction === 'asc' ? '↑' : '↓';
  };

  return (
    <div className="data-table">
      {/* Controls */}
      <div className="data-table__controls">
        <input
          type="search"
          value={searchQuery}
          onChange={handleSearch}
          placeholder="Search all columns..."
          aria-label="Search table"
        />
        <label>
          Rows per page:
          <select value={itemsPerPage} onChange={handleItemsPerPage}>
            {ITEMS_PER_PAGE_OPTIONS.map(n => (
              <option key={n} value={n}>{n}</option>
            ))}
          </select>
        </label>
      </div>

      {/* Table */}
      <div className="data-table__wrapper" role="region" aria-label="Data table" tabIndex={0}>
        <table>
          <thead>
            <tr>
              {columns.map(col => (
                <th
                  key={col.key}
                  onClick={() => col.sortable !== false && handleSort(col.key)}
                  style={{ cursor: col.sortable !== false ? 'pointer' : 'default' }}
                  aria-sort={
                    sortConfig.key === col.key
                      ? sortConfig.direction === 'asc' ? 'ascending' : 'descending'
                      : 'none'
                  }
                >
                  {col.label}
                  {col.sortable !== false && (
                    <span aria-hidden="true"> {sortIcon(col.key)}</span>
                  )}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {paginatedData.length > 0 ? (
              paginatedData.map((row, rowIndex) => (
                <tr key={row.id ?? rowIndex}>
                  {columns.map(col => (
                    <td key={col.key}>
                      {col.render ? col.render(row[col.key], row) : row[col.key]}
                    </td>
                  ))}
                </tr>
              ))
            ) : (
              <tr>
                <td colSpan={columns.length} className="data-table__empty">
                  No results found.
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>

      {/* Pagination */}
      <div className="data-table__pagination">
        <span>
          Showing {Math.min((currentPage - 1) * itemsPerPage + 1, sortedData.length)}–
          {Math.min(currentPage * itemsPerPage, sortedData.length)} of {sortedData.length}
        </span>
        <div className="pagination-controls">
          <button onClick={() => setCurrentPage(1)}          disabled={currentPage === 1}>«</button>
          <button onClick={() => setCurrentPage(p => p - 1)} disabled={currentPage === 1}>‹</button>

          {Array.from({ length: Math.min(5, totalPages) }, (_, i) => {
            const page = Math.max(1, Math.min(currentPage - 2, totalPages - 4)) + i;
            return (
              <button
                key={page}
                onClick={() => setCurrentPage(page)}
                className={currentPage === page ? 'active' : ''}
                aria-current={currentPage === page ? 'page' : undefined}
              >
                {page}
              </button>
            );
          })}

          <button onClick={() => setCurrentPage(p => p + 1)} disabled={currentPage === totalPages}>›</button>
          <button onClick={() => setCurrentPage(totalPages)}  disabled={currentPage === totalPages}>»</button>
        </div>
      </div>
    </div>
  );
}

export default DataTable;

// ─── Usage ────────────────────────────────────────────
const userColumns = [
  { key: 'id',    label: 'ID',    sortable: false },
  { key: 'name',  label: 'Name'  },
  { key: 'email', label: 'Email' },
  { key: 'role',  label: 'Role',
    render: (val) => <span className={`badge badge--${val}`}>{val}</span>
  },
  { key: 'status', label: 'Status',
    render: (val) => <span className={val === 'active' ? 'green' : 'gray'}>{val}</span>
  },
];
```

---

### Example 3 — Advanced: Virtualized Infinite Scroll Feed

```jsx
// File: components/InfiniteFeed/InfiniteFeed.jsx
import { useState, useEffect, useRef, useCallback } from 'react';
import { FixedSizeList } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';

const PAGE_SIZE = 20;

function InfiniteFeed({ fetchPosts }) {
  const [posts, setPosts] = useState([]);
  const [hasMore, setHasMore] = useState(true);
  const [isLoadingMore, setIsLoadingMore] = useState(false);
  const totalLoadedRef = useRef(0);

  const isItemLoaded = useCallback(
    index => !hasMore || index < posts.length,
    [hasMore, posts.length]
  );

  const loadMoreItems = useCallback(async (startIndex, stopIndex) => {
    if (isLoadingMore) return;
    setIsLoadingMore(true);
    try {
      const page = Math.floor(startIndex / PAGE_SIZE) + 1;
      const newPosts = await fetchPosts(page, PAGE_SIZE);
      if (newPosts.length < PAGE_SIZE) setHasMore(false);
      setPosts(prev => [...prev, ...newPosts]);
      totalLoadedRef.current += newPosts.length;
    } finally {
      setIsLoadingMore(false);
    }
  }, [fetchPosts, isLoadingMore]);

  // Initial load
  useEffect(() => {
    loadMoreItems(0, PAGE_SIZE - 1);
  }, []); // eslint-disable-line

  const itemCount = hasMore ? posts.length + 1 : posts.length;

  function PostRow({ index, style }) {
    if (!isItemLoaded(index)) {
      return (
        <div style={style} className="feed-item feed-item--skeleton">
          <div className="skeleton skeleton--avatar" />
          <div className="skeleton-content">
            <div className="skeleton skeleton--line" />
            <div className="skeleton skeleton--line skeleton--short" />
          </div>
        </div>
      );
    }

    const post = posts[index];
    if (!post) return null;

    return (
      <div style={style} className="feed-item">
        <img src={post.author.avatar} alt={post.author.name} className="feed-item__avatar" />
        <div className="feed-item__content">
          <div className="feed-item__header">
            <strong>{post.author.name}</strong>
            <time dateTime={post.createdAt}>
              {new Date(post.createdAt).toLocaleDateString()}
            </time>
          </div>
          <p className="feed-item__text">{post.content}</p>
          <div className="feed-item__actions">
            <button>👍 {post.likes}</button>
            <button>💬 {post.comments}</button>
            <button>↗ Share</button>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="infinite-feed">
      <InfiniteLoader
        isItemLoaded={isItemLoaded}
        itemCount={itemCount}
        loadMoreItems={loadMoreItems}
        threshold={5} // start loading when 5 items from bottom
      >
        {({ onItemsRendered, ref }) => (
          <FixedSizeList
            ref={ref}
            height={window.innerHeight - 120}
            width="100%"
            itemCount={itemCount}
            itemSize={140}
            onItemsRendered={onItemsRendered}
          >
            {PostRow}
          </FixedSizeList>
        )}
      </InfiniteLoader>

      {!hasMore && (
        <p className="feed__end">You've reached the end! 🎉</p>
      )}
    </div>
  );
}

export default InfiniteFeed;
```

---

## 11. Real-World Use Cases

### 11.1 Search Results with Highlighting

```jsx
function highlightMatch(text, query) {
  if (!query.trim()) return text;
  const regex = new RegExp(`(${query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi');
  const parts = text.split(regex);

  return parts.map((part, i) =>
    regex.test(part)
      ? <mark key={i} className="highlight">{part}</mark>
      : part
  );
}

function SearchResults({ results, query }) {
  if (results.length === 0) {
    return (
      <div className="search-empty">
        <p>No results for "<strong>{query}</strong>"</p>
        <p>Try a different search term.</p>
      </div>
    );
  }

  return (
    <ul className="search-results">
      <p className="search-results__count">{results.length} result(s) found</p>
      {results.map(result => (
        <li key={result.id} className="search-result">
          <h3>{highlightMatch(result.title, query)}</h3>
          <p>{highlightMatch(result.excerpt, query)}</p>
          <span className="search-result__category">{result.category}</span>
        </li>
      ))}
    </ul>
  );
}
```

### 11.2 Drag to Reorder with Keyboard Support

```jsx
function SortableList({ items, onReorder }) {
  const [draggedId, setDraggedId] = useState(null);
  const [dragOverId, setDragOverId] = useState(null);

  function handleDragStart(e, id) {
    setDraggedId(id);
    e.dataTransfer.effectAllowed = 'move';
  }

  function handleDragOver(e, id) {
    e.preventDefault();
    if (id !== draggedId) setDragOverId(id);
  }

  function handleDrop(e, targetId) {
    e.preventDefault();
    if (draggedId === targetId) return;

    const draggedIndex = items.findIndex(i => i.id === draggedId);
    const targetIndex  = items.findIndex(i => i.id === targetId);
    const newItems     = [...items];
    const [moved]      = newItems.splice(draggedIndex, 1);
    newItems.splice(targetIndex, 0, moved);

    onReorder(newItems);
    setDraggedId(null);
    setDragOverId(null);
  }

  function handleDragEnd() {
    setDraggedId(null);
    setDragOverId(null);
  }

  return (
    <ul className="sortable-list">
      {items.map((item, index) => (
        <li
          key={item.id}
          draggable
          onDragStart={e => handleDragStart(e, item.id)}
          onDragOver={e => handleDragOver(e, item.id)}
          onDrop={e => handleDrop(e, item.id)}
          onDragEnd={handleDragEnd}
          className={`
            sortable-item
            ${draggedId === item.id ? 'sortable-item--dragging' : ''}
            ${dragOverId === item.id ? 'sortable-item--over' : ''}
          `}
          aria-roledescription="sortable item"
        >
          <span className="drag-handle" aria-hidden="true">⠿</span>
          <span>{item.label}</span>
        </li>
      ))}
    </ul>
  );
}
```

---

## 12. Best Practices

### 12.1 Always Use Stable, Unique Keys from Data

```jsx
// ✅ Best — database ID
{items.map(item => <Card key={item.id} item={item} />)}

// ✅ Good — compound key when needed
{items.map(item => <Row key={`${item.userId}-${item.productId}`} item={item} />)}

// ⚠️ Acceptable — index, only for static non-stateful lists
{staticLabels.map((label, i) => <Label key={i} text={label} />)}

// ❌ Never — Math.random() or Date.now() at render time
{items.map(item => <Card key={Math.random()} item={item} />)}
```

### 12.2 Extract List Item as a Component

Every list item that's more than 2-3 lines of JSX should be its own component.

### 12.3 Compute Transforms Outside JSX

```jsx
// ✅ Sort and filter before the return/JSX — not inline
const displayItems = useMemo(() =>
  items
    .filter(i => i.isActive)
    .sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

return (
  <ul>
    {displayItems.map(item => <Item key={item.id} item={item} />)}
  </ul>
);
```

### 12.4 Always Handle Empty State

```jsx
// ✅ Dedicated empty state — never render an empty list silently
if (items.length === 0) {
  return <EmptyState message="No items found." />;
}
return <ul>{items.map(...)}</ul>;
```

### 12.5 Paginate or Virtualize Large Lists

- < 100 items: render normally
- 100–500: paginate or memoize items
- 500+: virtualize with react-window or TanStack Virtual

### 12.6 Memoize Expensive List Items

```jsx
// Wrap heavy list item components in React.memo
const HeavyListItem = React.memo(function HeavyListItem({ item, onAction }) {
  return (
    <li>
      {/* Complex render with charts, images, calculations */}
    </li>
  );
});
```

---

## 13. Common Mistakes

### Mistake 1: Using Index as Key for Dynamic Lists

```jsx
// ❌ Items have internal state (checkboxes) — reordering causes bugs
{todos.map((todo, index) => <TodoItem key={index} todo={todo} />)}

// ✅ Stable ID from data
{todos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
```

### Mistake 2: Generating Keys During Render

```jsx
// ❌ New key every render → component unmounts and remounts every render
{items.map(item => <Card key={Math.random()} item={item} />)}
{items.map(item => <Card key={Date.now()} item={item} />)}

// ✅ Key comes from data, generated once at item creation
{items.map(item => <Card key={item.id} item={item} />)}
```

### Mistake 3: Missing key Prop

```jsx
// ❌ React warning + potential bugs
{users.map(user => <UserCard user={user} />)}

// ✅ Always provide key
{users.map(user => <UserCard key={user.id} user={user} />)}
```

### Mistake 4: Putting key on the Wrong Element

```jsx
// ❌ Key on inner element
{items.map(item => (
  <div>
    <span key={item.id}>{item.name}</span>  {/* WRONG */}
  </div>
))}

// ✅ Key on outermost returned element
{items.map(item => (
  <div key={item.id}>
    <span>{item.name}</span>
  </div>
))}
```

### Mistake 5: Mutating Arrays in State

```jsx
// ❌ Mutates state array directly
function addItem(newItem) {
  items.push(newItem);     // Direct mutation
  setItems(items);         // Same reference — React may not re-render
}

// ✅ Create new array
function addItem(newItem) {
  setItems(prev => [...prev, newItem]);
}
```

### Mistake 6: Not Memoizing Expensive List Computations

```jsx
// ❌ Re-runs expensive filter/sort on every render
function ProductGrid({ products, filters }) {
  const filtered = products
    .filter(complexFilterFn)
    .sort(expensiveSortFn);

  return filtered.map(p => <ProductCard key={p.id} product={p} />);
}

// ✅ Memoize with useMemo
const filtered = useMemo(() =>
  products.filter(complexFilterFn).sort(expensiveSortFn),
  [products, filters]
);
```

### Mistake 7: Not Using Fragment When Returning Multiple Siblings from map

```jsx
// ❌ Returns invalid JSX (two siblings without wrapper)
{data.map(item => (
  <dt>{item.term}</dt>
  <dd>{item.definition}</dd>
))}

// ✅ Use Fragment with key
{data.map(item => (
  <Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.definition}</dd>
  </Fragment>
))}
```

---

## 14. Performance Considerations

### 14.1 useMemo for Filtering and Sorting

```jsx
const sortedAndFiltered = useMemo(() => {
  return products
    .filter(p => p.category === selectedCategory)
    .sort((a, b) => a.price - b.price);
}, [products, selectedCategory]);
```

### 14.2 React.memo for List Items

```jsx
const ProductCard = React.memo(function ProductCard({ product, onAddToCart }) {
  return (/* ... */);
});

// Now ProductCard only re-renders if product or onAddToCart changes
// Use useCallback for onAddToCart to prevent unnecessary re-renders
const handleAddToCart = useCallback((id) => {
  addToCart(id);
}, []);
```

### 14.3 Stable Callback Props with useCallback

```jsx
// ❌ New function on every parent render → all list items re-render
{items.map(item => (
  <Item key={item.id} onDelete={() => deleteItem(item.id)} />
))}

// ✅ Stable reference with useCallback — only re-renders if deleteItem changes
const handleDelete = useCallback((id) => {
  deleteItem(id);
}, [deleteItem]);

{items.map(item => (
  <Item key={item.id} id={item.id} onDelete={handleDelete} />
))}
```

### 14.4 Pagination Over Virtualization for Simpler Cases

Pagination is simpler to implement and sufficient for most cases. Only reach for virtualization when you genuinely have thousands of items that must all be browseable without paging.

### 14.5 Key Stability Prevents Expensive Remounting

```jsx
// ❌ Changing keys causes full unmount/remount on every sort
{sortedItems.map((item, index) => (
  <HeavyCard key={index} item={item} />  // key=0 was A, now key=0 is B → remount!
))}

// ✅ Stable keys — React reorders DOM nodes cheaply instead of remounting
{sortedItems.map(item => (
  <HeavyCard key={item.id} item={item} />  // same id = same component instance
))}
```

---

## 15. Interview Questions

### Q1: Why does React need a key prop when rendering lists?

**Answer:** React uses keys to identify which items in a list have changed, been added, or removed between renders. Without keys, React compares list items by their position in the array. This causes bugs when items are reordered or inserted in the middle — React reuses the wrong DOM nodes and assigns state to the wrong components. With stable unique keys, React can correctly track each item regardless of its position, performing the minimum number of DOM operations and preserving component state correctly.

---

### Q2: Why is using array index as a key bad for dynamic lists?

**Answer:** Index keys break when items are reordered, inserted, or removed. React identifies components by their key, so if index 0 was "Alice" and after deletion index 0 becomes "Bob", React thinks the same component is still at position 0 and just updates its content — instead of recognizing that Alice was deleted and Bob stayed. This causes state bugs: if Alice had an active input field, Bob inherits that state. Use stable unique IDs from your data as keys instead.

---

### Q3: What makes a good key? What makes a bad key?

**Answer:** A good key is **stable** (same item always has the same key), **unique among siblings** (no two siblings share a key), and **from your data** (database ID, slug, or compound key). A bad key is a random value generated at render time like `Math.random()` (causes full remount every render), or array index for dynamic lists (causes state assignment bugs on reorder/delete). The best keys are database IDs. For new client-created items without a server ID yet, use `crypto.randomUUID()` at the time of item creation.

---

### Q4: Can two different lists have items with the same key?

**Answer:** Yes. Keys only need to be unique **among siblings within the same list**. Two completely separate lists can each have an item with `key="1"` without any conflict. React's reconciler tracks keys per-parent, not globally. The same key in different lists is perfectly valid and will not cause any issues.

---

### Q5: What happens when you change a component's key?

**Answer:** When a key changes, React treats the element as a completely different component — it unmounts the old one (destroying all state and running cleanup effects) and mounts a brand new one (creating fresh state and running setup effects). This is actually a useful technique for **intentionally resetting component state**: if you want a component to fully reset when a certain prop changes, give it `key={thatProp}`. For example, `<UserProfile key={userId} userId={userId} />` ensures a clean state for every user.

---

### Q6: How does React's reconciliation algorithm handle list changes?

**Answer:** React's diffing algorithm for lists uses keys to match elements across renders. It does a two-pointer scan from both ends of the list looking for matching keys. For the unmatched middle section, it builds a key-to-index map of the old items, then for each new item checks if it exists in the old map. Matched items are moved or updated in place (cheaply). Items in the old map not matched to new items are removed. Items in the new list not found in the old map are created. This is why stable keys make list updates O(n) instead of O(n²).

---

### Q7: When is it acceptable to use array index as a key?

**Answer:** Index keys are acceptable only when all three conditions are true: (1) the list is static and will never be reordered, filtered differently, or have items inserted/removed; (2) list items have no internal state (no input fields, checkboxes, or `useState`); (3) items have no stable unique identifier. A static read-only numbered list of fixed steps is a valid use case. A todo list, search results, or any list that can be modified is not.

---

### Q8: What is list virtualization and when should you use it?

**Answer:** Virtualization (windowing) is the technique of rendering only the items currently visible in the viewport plus a small buffer, rather than all items at once. As the user scrolls, items leaving the viewport are removed from the DOM and new ones are added. This keeps DOM node count constant regardless of data size. Use virtualization when a list has 500+ items, especially if items are complex (images, nested components). Libraries like `react-window` and `TanStack Virtual` implement this. For simpler cases, pagination achieves the same performance goal with less complexity.

---

### Q9: How do you correctly add, remove, and update items in a list stored in state?

**Answer:** All list state operations must return a **new array** — never mutate the existing one. Add: `setItems(prev => [...prev, newItem])`. Remove by id: `setItems(prev => prev.filter(item => item.id !== id))`. Update by id: `setItems(prev => prev.map(item => item.id === id ? { ...item, ...updates } : item))`. Sorting: `setItems(prev => [...prev].sort(compareFn))` — always spread first to create a copy before sorting since `.sort()` mutates. These patterns guarantee React detects the change and re-renders correctly.

---

### Q10: How do you render a list of items where each item returns two or more sibling elements?

**Answer:** Use `React.Fragment` with an explicit `key` prop. The shorthand `<>` syntax does not support the `key` attribute, so you must use the long-form `<Fragment key={item.id}>` from `'react'`. This is commonly needed for definition lists (`dt`/`dd` pairs), table rows with groups, or any case where you want to avoid adding an extra wrapper DOM element. The `key` goes on the `Fragment`, not on the inner elements.

---

## 16. Practice Tasks

### Task 1 — Beginner: Contact Book

Build a `ContactBook` component.

**Requirements:**
- Hardcode an initial list of 10 contacts: `{ id, name, phone, email, group }`
- Render each contact as a card with name, phone, email, and group badge
- Allow adding a new contact via a form (name + phone + email + group dropdown)
- Allow deleting a contact via a button on each card (with confirm dialog)
- Sort contacts alphabetically by name automatically
- Group contacts under alphabetical headings (A, B, C...)
- Use stable `crypto.randomUUID()` for new contact IDs
- Show a count: "12 contacts"
- Handle empty state: "No contacts yet. Add your first one!"

---

### Task 2 — Intermediate: Filterable Product Catalog

Build a `ProductCatalog` component.

**Requirements:**
- Start with 20 products: `{ id, name, price, category, brand, rating, inStock, image }`
- Filter bar: category (multi-select checkboxes), price range (min/max inputs), in-stock toggle, minimum rating (1–5 stars selector)
- Sort options: Name A–Z, Name Z–A, Price Low–High, Price High–Low, Highest Rated
- Show result count: "Showing 8 of 20 products"
- Each product card shows: image, name, brand, price, rating stars, in/out of stock badge
- "Add to Cart" button — track cart count in parent state
- "Reset Filters" button restores defaults
- All filtering/sorting computed with `useMemo`
- Empty state when no products match: "No products match your filters."
- Extract `ProductCard` as a memoized component

---

### Task 3 — Advanced: Drag-and-Drop Task Board

Build a full Kanban board with drag-and-drop using only React state (no external DnD library).

**Requirements:**
- Three columns: Backlog, In Progress, Done
- Tasks: `{ id, title, description, priority: 'low'|'medium'|'high', assignee, dueDate }`
- Drag tasks between columns using HTML5 drag and drop events
- Visual drop target highlighting on drag-over
- Prevent dropping in the same column
- Within a column, drag to reorder tasks
- Add new task via inline form at bottom of each column
- Edit task title inline on double-click (contenteditable or input swap)
- Delete task with hold-to-confirm (show "Hold to delete" button, confirm after 1.5s)
- Filter tasks by priority (top-level filter chips: All / Low / Medium / High)
- Persist board to `localStorage` with lazy initialization
- Show task count per column
- Column total tasks and completed percentage in column header
- Memoize each TaskCard with `React.memo`
- Use `useCallback` for all event handlers passed to TaskCards

---

## 17. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| Lists | Use `.map()` to transform data arrays into JSX element arrays |
| key prop | Special prop for React to track identity across renders — not passed to components |
| Good key | Stable, unique among siblings, from your data (database ID, slug) |
| Bad key | `Math.random()`, `Date.now()` at render, array index for dynamic lists |
| Index key | Only safe for static, non-stateful, never-reordered lists |
| Changing key | Causes full unmount + remount — use intentionally to reset state |
| Key placement | Always on the outermost element returned by `.map()` |
| Fragment key | Use `<Fragment key={id}>` not `<>` — shorthand doesn't support key |
| Empty state | Always handle explicitly — never render a silently empty list |
| List mutations | Always return new arrays — spread, filter, map, never push/splice |
| Virtualization | Render only visible items — use for 500+ item lists |
| Performance | useMemo for transforms, React.memo for items, useCallback for handlers |

### List Operations Quick Reference

```jsx
// Add to end
setItems(prev => [...prev, newItem])

// Add to beginning
setItems(prev => [newItem, ...prev])

// Remove by id
setItems(prev => prev.filter(i => i.id !== id))

// Update by id
setItems(prev => prev.map(i => i.id === id ? { ...i, ...updates } : i))

// Toggle field by id
setItems(prev => prev.map(i => i.id === id ? { ...i, done: !i.done } : i))

// Sort (immutably)
setItems(prev => [...prev].sort((a, b) => a.name.localeCompare(b.name)))

// Move item up
setItems(prev => {
  const next = [...prev];
  [next[idx-1], next[idx]] = [next[idx], next[idx-1]];
  return next;
})

// Clear all
setItems([])
```

### The key Prop Decision Tree

```
Does my data have a unique ID field?
  ├── YES → use item.id (or item.slug, item.uuid) as key
  └── NO  → Can I generate a stable ID at item creation?
              ├── YES → use crypto.randomUUID() when creating item, store in state
              └── NO  → Is the list static and items have no state?
                          ├── YES → index key is acceptable
                          └── NO  → build a stable compound key: `${a.userId}-${a.productId}`
```

---

> **Next Topic:** `09-forms-and-validation.md` — Complete guide to building forms in React: controlled inputs, form validation, error handling, custom form hooks, and integrating with validation libraries like Zod and React Hook Form.

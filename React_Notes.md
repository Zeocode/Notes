# React 19 Crash Course – Notes with Code & Examples

*Notes from "React 19 Crash Course for Beginners 2026 (Learn in 90 Minutes)". Code is reconstructed from the examples described in the video.*

## Table of Contents

1. [Prerequisites & Setup](#1-prerequisites--setup)
2. [What is React?](#2-what-is-react)
3. [Components](#3-components)
4. [JSX](#4-jsx)
5. [How React Starts](#5-how-react-starts-indexhtml--maintsx)
6. [Virtual DOM](#6-virtual-dom-very-common-interview-question)
7. [Creating Components](#7-creating-components)
8. [Props](#8-props)
9. [The children Prop](#9-the-children-prop-content-projection)
10. [Event Handlers](#10-event-handlers)
11. [Conditional Rendering](#11-conditional-rendering)
12. [Lists & Looping](#12-lists--looping)
13. [Forms & Input Handling](#13-forms--input-handling)
14. [State with useState](#14-state-with-usestate)
15. [Side Effects with useEffect](#15-side-effects-with-useeffect)
16. [Context API](#16-context-api-solving-prop-drilling)
17. [Routing with React Router](#17-routing-with-react-router)
18. [Demo App Architecture](#18-demo-app-architecture-aimon-generator)
19. [Workshop: Building the AImon App](#19-workshop--building-the-aimon-app-step-by-step)
20. [What's Next](#20-whats-next-react-19--advanced-topics)
21. [Interview Cheat Sheet](#21-interview-cheat-sheet)

---

## 1. Prerequisites & Setup

- **Know first:** HTML, JavaScript ES6 (arrow functions, destructuring, spread), a little CSS, Git, basic programming concepts.
- **Tools:** Node.js, Git, VS Code (or Google Antigravity), ES7 React snippets extension. Vite is not a prerequisite but is needed later.
- **Playground:** stackblitz.com → New project → React + TypeScript (edit `src/App.tsx`).
- **Dev tools:** install the *React Developer Tools* Chrome extension → Inspect → **Components** tab shows the component tree.
- **Tech stack of the demo app ("AImon" generator):** React 19, Vite, Tailwind CSS, Express.js, PostgreSQL, Memcached.

```bash
# slides folder
npm install
npm run dev
```

## 2. What is React?

- A small **JavaScript library** for building UIs, created by Facebook/Meta in 2013.
- **Component-based architecture:** build a piece of UI once (e.g. a card) and reuse it everywhere.
- Used by Airbnb, Netflix, Instagram.

| Library (React) | Framework (e.g. Angular) |
|---|---|
| Small, focused on one job | Full solution: router, HTTP, forms, state, testing |
| Flexible – you choose router, state tools, etc. | Opinionated – projects look alike |
| Teams must define their own standards | Consistency built in |

- Community's "unofficial" router choice: **React Router**.

## 3. Components

- Everything on screen is a component: a card, a header, a settings widget, even a whole page.
- Components nest: `Root → Navbar / List → Item`. Write `Item` once, reuse with different data.
- A component is **a function that returns JSX**.

## 4. JSX

JSX = HTML written inside JavaScript. It is **not a string** – it compiles to `React.createElement` calls:

```jsx
const el = <h1>Hello</h1>;
// compiles to:
const el = React.createElement('h1', null, 'Hello');
```

You never write the compiled form, but it's good to know for interviews.

### Basic component

```tsx
function App() {
  return (
    <div>
      <h1>Hello</h1>
      <p>How is it going</p>
    </div>
  );
}
```

### Rule 1 – One single parent element

```tsx
// ❌ Error: Adjacent JSX elements must be wrapped in an enclosing tag
return (
  <h1>Hello</h1>
  <p>How is it going</p>
);

// ✅ Wrap in a div ...
return (<div><h1>Hello</h1><p>How is it going</p></div>);

// ✅ ... or a Fragment
return (<><h1>Hello</h1><p>How is it going</p></>);
```

**Interview question – div vs Fragment?** A `<div>` is rendered into the DOM as an extra element. A Fragment (`<>...</>`) is **not rendered**; its children appear directly in the parent (inspect `#root` to see the difference). Useful when extra wrappers break semantics, e.g. between a `<ul>` and its `<li>`s.

### Rule 2 – `className` and `htmlFor`

`class` and `for` are reserved JavaScript keywords, so JSX uses different names.

```jsx
<label htmlFor="email" className="label">Email</label>
```

### Rule 3 – Close all tags

```jsx
<img src="photo.png" />
<input type="text" />
<br />
```

Not closing them gives *"Unterminated JSX contents"*.

### Rule 4 – JavaScript expressions in `{ }`

```tsx
function App() {
  const name = 'Essen';
  const items = ['a', 'b', 'c'];
  return (
    <>
      {/* defaultValue = editable initial value; value would lock the input */}
      <input type="text" defaultValue={name} />
      <p>Items: {items.length}</p>
      <p>Sum: {2 + 3}</p>
      <p>Today: {new Date().toDateString()}</p>
    </>
  );
}
```

- Use `defaultValue` (not `value`) for an editable pre-filled input. A hard-coded `value` makes it read-only unless it is tied to state (see Forms).

### Rule 5 – Inline styles are objects with camelCase keys

```tsx
// ❌ <p style="background-color: red">
const styles = { backgroundColor: 'red', color: 'white' };
<p style={styles}>Styled text</p>

// or inline
<p style={{ backgroundColor: 'red', color: 'white' }}>Styled text</p>
```

## 5. How React Starts (index.html → main.tsx)

- Vite's `index.html` has just `<div id="root"></div>` and a script tag pointing to `main.tsx`.
- `main.tsx` tells React where to render:

```tsx
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')!).render(<App />);
```

React finds `#root`, renders `<App />` and all nested components inside it.

## 6. Virtual DOM (very common interview question)

**Problem:** the browser is single-threaded, and DOM updates are costly. Doing e.g. 1000 DOM operations blocks the UI.

**React's solution – virtual DOM:**

1. State changes → React creates a **new lightweight copy** (virtual DOM) of the UI. Real DOM nodes are heavy (styles, classes, properties, functions).
2. It **compares** the old and new virtual DOM → called **diffing**.
3. It calculates the **minimum set of changes**.
4. It **batches** and patches only those parts into the real DOM.

Example: a list `<ul>` with three `<li>`; if only one value changes from `b` to `x`, only that one node is updated.

### Why `key` matters

With 500 list items, React must know *which* item changed. Without unique keys it may delete and re-create all 500 DOM nodes. Always give list items a **unique, stable `key`**.

## 7. Creating Components

### Function component (arrow-function style is the modern way)

```tsx
// Welcome.tsx
export const Welcome = () => {
  return <h1>Hello World</h1>;
};
```

```tsx
// App.tsx
import { Welcome } from './Welcome';

function App() {
  return (
    <>
      <Welcome />
      <Welcome />   {/* reusable – render as many times as you like */}
    </>
  );
}
```

- **Class components** exist (older style) but are no longer used. You may meet them only in ~10-year-old enterprise code.

## 8. Props

Props are values a **parent passes to a child** so the same component can render different output. Rule of thumb: *if the output can differ between uses, it must be a prop.*

```tsx
// Welcome.tsx
type WelcomeProps = {
  name: string;
};

export const Welcome = ({ name }: WelcomeProps) => {
  return <h1>Hello {name}</h1>;
};
```

```tsx
// App.tsx
<Welcome name="Essen" />
<Welcome name="Ayaz" />
```

Without the TypeScript type you get *"implicitly has an 'any' type"*. Destructure `{ name }` from the props object to use `name` directly.

### UserCard example (string, union and boolean props)

```tsx
// UserCard.tsx
type UserCardProps = {
  name: string;
  role: 'admin' | 'user';   // union type: only these two values are accepted
  verified: boolean;
};

export const UserCard = ({ name, role, verified }: UserCardProps) => {
  return (
    <div style={{ display: 'flex', gap: '16px', alignItems: 'center' }}>
      <h2>{name}</h2>
      <span>{role}</span>
      {verified && <span>✔</span>}
    </div>
  );
};
```

```tsx
// App.tsx
<UserCard name="Essen" role="user" verified={true} />
<UserCard name="Ayaz" role="admin" verified={true} />
```

- `role="XYZ"` is a TypeScript error because of the union type; IntelliSense suggests only `admin` / `user`.
- `verified={false}` hides the tick mark (conditional rendering with `&&`).

## 9. The `children` Prop (content projection)

Anything placed between a component's opening and closing tags is available as `children`.

```tsx
// Card.tsx
import React from 'react';

export const Card = ({ children }: React.PropsWithChildren) => {
  return <div className="card">{children}</div>;
};
```

```css
/* App.css */
.card {
  padding: 8px 16px;
  background-color: #333;
  color: white;
  border-radius: 16px;
  margin-bottom: 8px;
}
```

```tsx
// App.tsx – one card with one child, another with two
<Card>
  <Welcome name="Essen" />
</Card>

<Card>
  <UserCard name="Essen" role="user" verified={true} />
  <UserCard name="Ayaz" role="admin" verified={true} />
</Card>
```

The card can also add its own structure around `{children}`:

```tsx
export const Card = ({ children }: React.PropsWithChildren) => (
  <div className="card">
    <button>Click me</button>
    <section>{children}</section>
  </div>
);
```

Every card shares the same styles and structure; only the projected content differs.

## 10. Event Handlers

Bind handlers with camelCase attributes: `onClick`, `onChange`, `onKeyUp`, `onKeyDown`, `onMouseOver`, `onMouseEnter`, `onFocus`, `onBlur`, `onSubmit`. Inside `{ }` pass a **function** (or reference to one).

### Example 1 – basic click (function reference)

```tsx
const Button = () => {
  const handleClick = () => alert('Button clicked');
  return <button onClick={handleClick}>Click me</button>;
};
```

### Example 2 – inline handler

```tsx
<button onClick={() => alert('Inline click')}>Click</button>
```

### Example 3 – the event object

When you pass a function *reference*, React automatically calls it with the event (a *synthetic event*).

```tsx
const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
  console.log(event);        // SyntheticBaseEvent
  console.log(event.target); // which element was clicked
  console.log(event.type);   // "click"
  // event.preventDefault() is also available
};
<button onClick={handleClick}>Click</button>
```

### Example 4 – passing your own arguments

If you write `onClick={greetUser(name)}` the function runs during render. And if you write `onClick={greetUser}` the handler receives the **event** as its argument (you'd see `[object Object]`). To pass your own value, wrap the call in an inline arrow function:

```tsx
export const UserCard = ({ name, role, verified }: UserCardProps) => {
  const greetUser = (username: string) => {
    alert(`Hello ${username}`);
  };

  return (
    <div>
      <h2>{name}</h2>
      <button onClick={() => greetUser(name)}>Show user info</button>
    </div>
  );
};
```

Clicking the button on Essen's card alerts "Hello Essen"; on Ayaz's card it alerts "Hello Ayaz". Update the component once and every instance gets the change.

**Debugging tip:** DevTools → Sources → `Ctrl/Cmd + P` → type the file name → click a line number to set a breakpoint, then check the **Call Stack** to see who called your function.

## 11. Conditional Rendering

Show or hide elements based on conditions (they are actually added to or removed from the DOM).

### Ternary – if / else inside JSX

```tsx
export const Greeting = ({ isLoggedIn }: { isLoggedIn: boolean }) => (
  <h1>{isLoggedIn ? 'Welcome back' : 'Please sign in'}</h1>
);

// <Greeting isLoggedIn={false} /> → "Please sign in"
// <Greeting isLoggedIn={true} />  → "Welcome back"
```

### Logical AND `&&` – render only if truthy (no else)

```tsx
{count > 0 && <p>You have {count} new messages</p>}
```

### Logical OR `||` – show a default value

```tsx
<p>Hello, {username || 'Guest'}</p>   {/* empty string is falsy → "Guest" */}
```

### Multiple conditions – `if / else if` before `return`

```tsx
const StatusMessage = ({ status }: { status: string }) => {
  if (status === 'loading') return <div>Loading...</div>;
  if (status === 'error')   return <div>Error occurred</div>;
  if (status === 'success') return <div>Success!</div>;
  return <div>Unknown status</div>;
};
```

### `switch` statement

```tsx
const UserDashboard = ({ role }: { role: string }) => {
  const renderContent = () => {
    switch (role) {
      case 'admin': return <div>Admin Panel</div>;
      case 'user':  return <div>User Profile</div>;
      default:      return <div>Guest View</div>;
    }
  };
  return <div>{renderContent()}</div>;
};
```

### Live examples from the StackBlitz template

```tsx
// 1. if/else with state: login/logout toggle
const [isLoggedIn, setIsLoggedIn] = useState(false);
<h1>{isLoggedIn ? 'Welcome back' : 'Please sign in'}</h1>
<button onClick={() => setIsLoggedIn(!isLoggedIn)}>
  {isLoggedIn ? 'Logout' : 'Login'}
</button>

// 2. ternary: day / night
const [isDay, setIsDay] = useState(true);
<p>{isDay ? 'Good morning' : 'Good evening'}</p>
<button onClick={() => setIsDay(!isDay)}>Toggle time</button>

// 3. logical AND: notification bell
const [hasNotifications, setHasNotifications] = useState(true);
{hasNotifications && <span>🔔</span>}
<button onClick={() => setHasNotifications(!hasNotifications)}>Toggle notifications</button>

// 4. logical OR: default username (controlled input)
const [username, setUsername] = useState('');
<input value={username} onChange={(e) => setUsername(e.target.value)} />
<p>Hello, {username || 'Guest'}</p>

// 5. multiple conditions: status buttons (pending / success / error)
const [status, setStatus] = useState('pending');
<button onClick={() => setStatus('pending')}>Pending</button>
<button onClick={() => setStatus('success')}>Success</button>
<button onClick={() => setStatus('error')}>Error</button>

// 6. switch: role selector (admin / moderator / user / guest)
const [role, setRole] = useState('user');   // default → "User Profile"
```

## 12. Lists & Looping

Use `.map()` on an array. The callback receives `(item, index)`. **Always add a unique `key`.**

```tsx
const fruits = ['Apple', 'Banana', 'Orange', 'Mango'];

<ul>
  {fruits.map((fruit, index) => (
    <li key={index}>{fruit}</li>
  ))}
</ul>
```

Add or remove items from the array and the UI re-renders automatically.

### Best practice – use a unique ID as the key

```tsx
const users = [
  { id: 1, name: 'Alice', age: 25 },
  { id: 2, name: 'Bob', age: 30 },
  { id: 3, name: 'Charlie', age: 35 },
];

<ul>
  {users.map((user) => (
    <li key={user.id}>{user.name} ({user.age})</li>
  ))}
</ul>
```

Index keys are acceptable for static lists; for dynamic lists use an ID. Duplicate keys trigger a React warning.

### Filtering – to-do example

```tsx
const activeTodos = todos.filter((todo) => !todo.completed);

<ul>
  {activeTodos.map((todo) => <li key={todo.id}>{todo.text}</li>)}
</ul>
```

### Search filter with "no results" message

```tsx
const allItems = ['Apple', 'Banana', 'Cherry', 'Date'];
const [filter, setFilter] = useState('');

const filteredItems = allItems.filter((item) =>
  item.toLowerCase().includes(filter.toLowerCase())
);

<input value={filter} onChange={(e) => setFilter(e.target.value)} />
<ul>
  {filteredItems.map((item) => <li key={item}>{item}</li>)}
</ul>
{filteredItems.length === 0 && <p>No results found</p>}
```

If the filter is empty, `includes('')` is true for everything, so all items show.

### Dynamic list – add and remove

```tsx
const [items, setItems] = useState(['Item 1', 'Item 2', 'Item 3']);
const [text, setText] = useState('');

const addItem = () => {
  setItems([...items, text]);   // new array: copy old items + new one
  setText('');                  // reset the input
};

const removeItem = (indexToRemove: number) => {
  // keep everything whose index is NOT the one being removed
  setItems(items.filter((_, index) => index !== indexToRemove));
};

<input value={text} onChange={(e) => setText(e.target.value)} />
<button onClick={addItem}>Add</button>
<ul>
  {items.map((item, index) => (
    <li key={index}>
      {item} <button onClick={() => removeItem(index)}>✕</button>
    </li>
  ))}
</ul>
```

## 13. Forms & Input Handling

### Controlled input

When an input's `value` is bound to state and `onChange` updates that state, **React controls the input**. (Uncontrolled inputs exist too, but weren't covered.)

```tsx
const [name, setName] = useState('');

<input value={name} onChange={(e) => setName(e.target.value)} />
<p>Hello, {name || 'stranger'}!</p>
```

### Multiple inputs with one object state and one handler

Give each input a `name` attribute that matches a key in the state object, then use `e.target.name` to update the right key.

```tsx
const [formData, setFormData] = useState({ firstName: '', lastName: '', email: '' });

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setFormData({ ...formData, [name]: value });   // computed key + spread
};

<input name="firstName" value={formData.firstName} onChange={handleChange} />
<input name="lastName"  value={formData.lastName}  onChange={handleChange} />
<input name="email"     value={formData.email}     onChange={handleChange} />
<p>{formData.firstName} {formData.lastName}</p>
```

### Form submission

Use `onSubmit` on the `<form>` and a `<button type="submit">`. **Call `e.preventDefault()`**, otherwise the page reloads (we are building a single-page app and want to send data to an API without a reload).

```tsx
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  console.log('Form submitted', formData);   // here you would call your API
};

<form onSubmit={handleSubmit}>
  {/* inputs ... */}
  <button type="submit">Submit</button>
</form>
```

### Checkboxes and radio buttons

| Input | Bound attribute | Read from |
|---|---|---|
| text | `value` | `e.target.value` |
| checkbox | `checked` | `e.target.checked` |
| radio | `checked` (compare to state) + `value` | `e.target.value` |

```tsx
// Checkbox
<input
  type="checkbox"
  name="termsAccepted"
  checked={formData.termsAccepted}
  onChange={(e) => setFormData({ ...formData, termsAccepted: e.target.checked })}
/>

// Radio group
<input
  type="radio" name="gender" value="male"
  checked={formData.gender === 'male'}
  onChange={(e) => setFormData({ ...formData, gender: e.target.value })}
/>
<input
  type="radio" name="gender" value="female"
  checked={formData.gender === 'female'}
  onChange={(e) => setFormData({ ...formData, gender: e.target.value })}
/>
```

The complete registration form in the StackBlitz template also uses the `required` attribute on inputs, so the browser shows validation messages on submit.

## 14. State with `useState`

A **state variable** holds a value that changes over time. `useState` returns `[currentValue, setterFunction]`; the argument is the initial value.

```tsx
import { useState } from 'react';

const [count, setCount] = useState(0);
const [user, setUser] = useState<User | null>(null);   // null until loaded from the API
```

- You **cannot** assign directly (`count = 5`). Always use the setter.
- Calling the setter re-renders the component (and creates a new virtual DOM).

### Gotcha – state updates are batched/asynchronous

```tsx
// ❌ Both lines see count = 0 → result is 1, not 2
setCount(count + 1);
setCount(count + 1);

// ✅ Functional update always receives the latest value → result is 2
setCount((prev) => prev + 1);
setCount((prev) => prev + 1);
```

### Immutability (interview topic)

Never **mutate** state – it won't re-render. Always pass a **new object / new array** (new reference in memory) using the spread operator `...`.

```tsx
// ❌ Mutation – no re-render
user.name = 'B';
items.push('x');

// ✅ New object: copy everything, then override
setUser({ ...user, name: 'B' });         // keeps address & designation

// ✅ New arrays
setItems([...items, 'new item']);                      // add
setItems(items.filter((item) => item !== 'x'));        // remove
```

## 15. Side Effects with `useEffect`

**Side effect** = anything that isn't a direct state update from a user action: fetching data, subscriptions, manually touching the DOM, `localStorage`, timers / `setTimeout`.

Typical use: calling an API when a page loads (e.g. the Gallery page).

```tsx
import { useEffect } from 'react';

useEffect(() => {
  // effect body
  return () => {
    // optional cleanup: runs on unmount and before the effect re-runs
  };
}, [dependencies]);
```

### Three flavours

| Form | When it runs |
|---|---|
| `useEffect(() => {...})` | After **every** render (almost never used) |
| `useEffect(() => {...}, [])` | **Once**, when the component mounts |
| `useEffect(() => {...}, [name])` | On mount **and** whenever `name` changes |

```tsx
const [name, setName] = useState('');

useEffect(() => {
  console.log('The new value of the input is', name);
}, [name]);   // logs on every keystroke (and once on mount)
```

With `[]` it logs only once on load. (You may see it log twice in development – that's **StrictMode**; ignore it.)

### Common mistakes

1. Using a variable inside the effect but **not listing it** in the dependencies.
2. Putting an **object** in the dependencies – a new object each render makes the effect run on **every render**.

## 16. Context API (solving prop drilling)

**Prop drilling:** to give a deeply nested component (e.g. `Avatar`) a value, every component in between (`App → Header → Nav → UserMenu → Avatar`) must accept and forward the prop. Common interview topic.

**Context** lets any component read a value directly. Steps: **create** a context → create a **provider** → **wrap** the app → call **`useContext`**.

```tsx
// UserContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type User = { name: string; id: string };
const UserContext = createContext<User | null>(null);

export const UserProvider = ({ children }: { children: ReactNode }) => {
  const [user] = useState<User>({ name: 'Essen', id: 'abc-123' });
  return <UserContext.Provider value={user}>{children}</UserContext.Provider>;
};

// custom hook wrapping useContext
export const useUser = () => useContext(UserContext);
```

```tsx
// main.tsx – wrap the app
<UserProvider>
  <App />
</UserProvider>

// Avatar.tsx – deep in the tree, no props needed
const Avatar = () => {
  const user = useUser();
  return <img alt={user?.name} />;
};
```

### Best practices

- Create **multiple contexts by concern** (`useUser`, `useTheme`).
- Provide context **only as high as needed** – a change re-renders every consumer.
- **Don't use it for frequently changing state.**
- Wrap `useContext` in **custom hooks**.
- For big apps use a state management library (e.g. **Redux**, which uses context internally).

## 17. Routing with React Router

Why a router in a single-page app: **multiple pages without reload**, working **browser back/forward**, and **shareable URLs** that land on the right page.

```bash
npm install react-router-dom
```

```tsx
// main.tsx – wrap in BrowserRouter
import { BrowserRouter } from 'react-router-dom';

<BrowserRouter>
  <App />
</BrowserRouter>
```

```tsx
// App.tsx – define the routes
import { Routes, Route, Link } from 'react-router-dom';

<>
  <nav>
    <Link to="/">Home</Link>
    <Link to="/about">About</Link>
    <Link to="/contact">Contact</Link>
  </nav>
  <Routes>
    <Route path="/" element={<HomePage />} />
    <Route path="/about" element={<AboutPage />} />
    <Route path="/contact" element={<ContactPage />} />
  </Routes>
</>
```

**Use `<Link to="...">`, not `<a href="...">`** – an anchor tag triggers a full page reload.

## 18. Demo App Architecture (AImon Generator)

- **Front end:** React 19, Vite, Tailwind CSS – draw a doodle on a canvas → Gemini generates an image ("Nano Banana"), name, category, powers and description.
- **Back end:** Node.js + Express, **PostgreSQL** (Neon or Docker), **Memcached** for caching.
- **Flow:** React asks the API for AImons → server checks **Memcached** → *cache hit:* respond (~40–60 ms) → *cache miss:* query Postgres (~800–1000 ms), store in cache, respond.

| Metric | No cache | With Memcached |
|---|---|---|
| Response time | ~1 s | ~50 ms |
| Queries hitting the DB | 100% | ~10% |
| Requests / second | ~200 | ~2,000 |

- Hosting (sponsor, Cloudways) provides Memcached, Redis (sessions/caching) and Varnish (HTTP/asset caching) out of the box. Users enter their own Gemini API key; it is stored in the browser only.

## 19. Workshop – Building the AImon App Step by Step

Repo: use the `client-tutorial` folder and its README (each step is marked `workshop step N` in the code).

### Setup

```bash
cd client-tutorial
cp .env.example .env          # set the API URL to http://localhost:3001
npm install
npm run dev                   # client

# new terminal – backend
cd server
npm install
npm run dev                   # server on port 3001

# new terminal – database + memcached (needs Docker), or use a Neon Postgres URL
docker compose up
# server/.env:  DATABASE_URL=postgresql://<user>:<password>@localhost:5432/<db>
```

Get a Gemini API key from **Google AI Studio**, then click *Set Gemini API Key* in the app.

### Step 1 – Routing

1. `main.jsx`: import `BrowserRouter` and wrap `<App />` with it.
2. `App.jsx`: define `<Routes>` with `/` → Generator page and `/gallery` → Gallery page.
3. `Header`: replace plain anchors/divs with `<Link>` / nav links.

### Step 2 – `useState` on the Generator page

```jsx
const [isGenerating, setIsGenerating] = useState(false);
const [error, setError] = useState(null);
const [lastResult, setLastResult] = useState(null);

const handleGenerate = async (/* canvas data */) => {
  setIsGenerating(true);   // show loader
  setError(null);          // clear previous error
  try {
    const response = await fetch(`${API_URL}/api/generate`, { /* ... */ });
    const data = await response.json();
    setLastResult(data);
  } catch (err) {
    setError(err.message);
  } finally {
    setIsGenerating(false);
  }
};
```

- Disable the canvas/button while `isGenerating`.
- Conditional rendering: no result and not generating → default message; generating → loader; result and not generating → image, name, category, description, powers; `error` → error message.

### Step 3 – Data fetching with `useEffect` (Gallery page)

```jsx
const [data, setData] = useState([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

useEffect(() => {
  fetchGallery();
}, []);   // run once on mount

const fetchGallery = async () => {
  setLoading(true);
  setError(null);
  try {
    const res = await fetch(`${API_URL}/api/gallery`);
    setData(await res.json());
  } catch (err) {
    setError(err.message);
  } finally {
    setLoading(false);
  }
};
```

Render a grid with `data.map(...)` (image, name, type, description, powers, a **Like** button). If `data.length === 0`, show an empty state.

### Step 4 – Custom hook

Move the `data / loading / error` state and the `useEffect` into a reusable hook and return `refetch` so the Like button can refresh the list:

```jsx
// hooks/usePokemonGallery.js
export function usePokemonGallery() {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchGallery = async () => { /* same as step 3 */ };
  useEffect(() => { fetchGallery(); }, []);

  return { data, loading, error, refetch: fetchGallery };
}

// GalleryPage.jsx – now cleaner (no useState / useEffect)
const { data, loading, error, refetch } = usePokemonGallery();
// after liking: await likeRequest(id); refetch();
```

### Step 5 – Context API for the theme

1. Replace `context/ThemeContext.jsx` with the provided `ThemeProvider` + `useTheme` hook.
2. `main.jsx`: wrap `<BrowserRouter>` in `<ThemeProvider>`.
3. `Header.jsx`: `const { theme, toggleTheme } = useTheme();` and uncomment the toggle button.

Check it in React DevTools → Components: `ThemeProvider → BrowserRouter → App → Header / Routes`. Changing the provider's state in DevTools updates the UI live.

### Step 6 – Filtering & sorting with `useMemo`

```jsx
import { useMemo, useState } from 'react';

const [sortBy, setSortBy] = useState('newest');     // 'newest' | 'likes' | 'name'
const [filterType, setFilterType] = useState('all');

const processedData = useMemo(() => {
  let result = [...data];
  if (filterType !== 'all') result = result.filter((p) => p.type === filterType);
  if (sortBy === 'likes') result.sort((a, b) => b.likes - a.likes);
  else if (sortBy === 'name') result.sort((a, b) => a.name.localeCompare(b.name));
  else result.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
  return result;
}, [data, sortBy, filterType]);   // recalculated only when these change

// unique types for the dropdown (Set removes duplicates)
const availableTypes = useMemo(
  () => [...new Set(data.map((p) => p.type))],
  [data]
);
```

Add `<select>` controls (sort: Newest / Most liked / Name; filter: All types + `availableTypes.map(...)` options), then **render `processedData` instead of `data`**.

### Step 7 – Lazy loading + Suspense

```jsx
import { lazy, Suspense } from 'react';

const GeneratorPage = lazy(() => import('./pages/GeneratorPage'));
const GalleryPage   = lazy(() => import('./pages/GalleryPage'));

<Suspense fallback={<Spinner />}>
  <Routes>
    <Route path="/" element={<GeneratorPage />} />
    <Route path="/gallery" element={<GalleryPage />} />
  </Routes>
</Suspense>
```

Code splitting: the Gallery code is only downloaded when you visit it (check DevTools → Network → JS: ~30 requests drop to ~28 initially).

### Step 8 – Error boundary

An error boundary catches rendering errors so the whole app doesn't go blank. Wrap the app in it (top level), or wrap just the page content under `<Suspense>` so the **header stays usable** and only the broken page shows the error.

```jsx
<ApiKeyProvider>
  <ErrorBoundary>
    <Header />
    <Suspense fallback={<Spinner />}>
      <Routes>{/* ... */}</Routes>
    </Suspense>
  </ErrorBoundary>
</ApiKeyProvider>
```

(To test it, temporarily add `throw new Error('Something went terrible')` in the Gallery page.)

## 20. What's Next (React 19 & advanced topics)

- **`use` hook** for reading promises.
- **`useActionState`** for forms.
- **`useOptimistic`** for instant/optimistic UI (e.g. show "liked" immediately, revert if the request fails).
- **Server Components & Server Actions** – advanced; learn after building a few apps.
- **React Compiler** – auto-optimizes, so you rarely need to write `useMemo` by hand.
- The repo contains a bonus markdown file on React 19 features.

## 21. Interview Cheat Sheet

- **What is React?** A small component-based JS library (not a full framework).
- **Virtual DOM?** Lightweight copy; diff old vs new, then patch only the minimum to the real DOM.
- **Why `key`?** Lets React identify which list items changed instead of re-rendering the whole list.
- **Fragment vs div?** A fragment adds no extra DOM node.
- **Props vs state?** Props come from the parent (read-only input); state is owned by the component and changes over time via its setter.
- **Why `setCount(prev => prev + 1)`?** State updates are batched; the functional form gets the latest value.
- **Why not mutate state?** React compares references; mutation doesn't trigger a re-render. Use spread to create new objects/arrays.
- **`useEffect` dependency forms?** none = every render, `[]` = mount only, `[x]` = mount + when `x` changes.
- **Prop drilling?** Passing props through many layers; solved by Context API.
- **Why `Link` instead of `a`?** Avoids a full page reload.
- **Controlled input?** `value` bound to state with `onChange` updating it.

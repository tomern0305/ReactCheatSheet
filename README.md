# ⚛️ React + TypeScript + Tailwind CSS — Cheat Sheet

> For CS students who are new to the stack but not new to thinking.

---

## Table of Contents

1. [Project Setup](#1-project-setup)
2. [TypeScript Essentials (React-relevant)](#2-typescript-essentials-react-relevant)
3. [React Core Concepts](#3-react-core-concepts)
4. [Hooks](#4-hooks)
5. [Component Patterns](#5-component-patterns)
6. [Tailwind CSS](#6-tailwind-css)
7. [Combining All Three](#7-combining-all-three)
8. [Common Gotchas](#8-common-gotchas)
9. [File Structure Convention](#9-file-structure-convention)
10. [Quick Reference Card](#10-quick-reference-card)

---

## 1. Project Setup

```bash
# Recommended: Vite (fast, modern)
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install

# Add Tailwind
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**`tailwind.config.js`** — tell Tailwind where your files are:

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: { extend: {} },
  plugins: [],
};
```

**`src/index.css`** — add these three lines (required):

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Make sure `index.css` is imported in `main.tsx`:

```tsx
import "./index.css";
```

---

## 2. TypeScript Essentials (React-relevant)

TypeScript is JavaScript with a type system. You already know types from Java/C++ — same idea, different syntax.

### Basic Types

```ts
let name: string = "Alice";
let age: number = 21;
let active: boolean = true;
let ids: number[] = [1, 2, 3];       // array
let pair: [string, number] = ["a", 1]; // tuple
```

### Union Types

```ts
type Status = "loading" | "success" | "error"; // only these 3 strings are valid
let s: Status = "loading"; // ✅
let s2: Status = "oops";   // ❌ compile error
```

### Interfaces vs Type Aliases

Both define the shape of an object. Prefer `interface` for objects, `type` for unions/primitives.

```ts
interface User {
  id: number;
  name: string;
  email?: string; // ? = optional
}

type Point = { x: number; y: number };

// Extending
interface Admin extends User {
  role: "admin" | "superadmin";
}
```

### Generics

Like Java generics — parameterize a type.

```ts
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello"); // returns string
identity<number>(42);      // returns number

// Common in React: useState<T>
```

### Non-null Assertion & Optional Chaining

```ts
// Optional chaining (safe navigation)
const city = user?.address?.city; // undefined if any part is null/undefined

// Non-null assertion (you're sure it's not null — use sparingly)
const el = document.getElementById("root")!;
```

---

## 3. React Core Concepts

### What is a Component?

A component is a **function that returns JSX**. JSX is HTML-like syntax that compiles to `React.createElement(...)` calls.

```tsx
// Simplest possible component
function Hello() {
  return <h1>Hello, world!</h1>;
}
```

### Props — Passing Data Down

Props are like function arguments. Always type them with an interface.

```tsx
interface GreetingProps {
  name: string;
  age?: number; // optional
}

function Greeting({ name, age = 18 }: GreetingProps) {
  return (
    <p>
      Hello, {name}! You are {age} years old.
    </p>
  );
}

// Usage
<Greeting name="Alice" />
<Greeting name="Bob" age={25} />
```

### JSX Rules

```tsx
// 1. Single root element (or use a Fragment)
return (
  <>
    <h1>Title</h1>
    <p>Body</p>
  </>
);

// 2. className, not class
<div className="text-red-500">...</div>

// 3. JS expressions in {}
<p>{2 + 2}</p>
<p>{isLoggedIn ? "Welcome" : "Please log in"}</p>

// 4. Self-close empty tags
<img src="..." alt="..." />
<br />

// 5. Events are camelCase
<button onClick={handleClick}>Click me</button>
```

### Conditional Rendering

```tsx
// Ternary
{isLoading ? <Spinner /> : <Content />}

// Short-circuit (renders nothing if false)
{isLoggedIn && <Dashboard />}

// If-else outside JSX
function Component({ show }: { show: boolean }) {
  if (!show) return null;
  return <div>Visible!</div>;
}
```

### Rendering Lists

```tsx
const fruits = ["apple", "banana", "cherry"];

// key must be unique and stable — do NOT use index if list can reorder
<ul>
  {fruits.map((fruit) => (
    <li key={fruit}>{fruit}</li>
  ))}
</ul>
```

---

## 4. Hooks

Hooks are functions that let you "hook into" React features. **Rules**: only call at the top level of a component (not inside loops/conditions), and only in React functions.

### `useState` — Local State

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState<number>(0);
  //     ^state  ^setter    ^hook    ^initial value

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount((prev) => prev - 1)}>-</button>
      {/* Use functional update when new state depends on old state */}
    </div>
  );
}
```

State with objects — always replace, never mutate:

```tsx
const [user, setUser] = useState({ name: "Alice", age: 20 });

// ✅ Correct: spread the old state
setUser((prev) => ({ ...prev, age: 21 }));

// ❌ Wrong: mutating directly won't trigger re-render
user.age = 21;
```

### `useEffect` — Side Effects

Run code after render. Think of it as a lifecycle hook.

```tsx
import { useEffect } from "react";

function Component() {
  // Runs after every render
  useEffect(() => {
    console.log("rendered");
  });

  // Runs once (on mount)
  useEffect(() => {
    fetchData();
  }, []); // empty dependency array

  // Runs when `userId` changes
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);

  // Cleanup (like componentWillUnmount)
  useEffect(() => {
    const timer = setInterval(() => tick(), 1000);
    return () => clearInterval(timer); // cleanup fn
  }, []);
}
```

### `useRef` — Mutable Reference / DOM Access

Doesn't trigger re-render when changed.

```tsx
import { useRef } from "react";

function InputFocus() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={() => inputRef.current?.focus()}>
        Focus the input
      </button>
    </>
  );
}
```

### `useContext` — Avoid Prop Drilling

Share data across the tree without passing props manually at every level.

```tsx
import { createContext, useContext, useState } from "react";

// 1. Create context
const ThemeContext = createContext<"light" | "dark">("light");

// 2. Provide it high up in the tree
function App() {
  const [theme] = useState<"light" | "dark">("dark");
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}

// 3. Consume it anywhere below
function DeepChild() {
  const theme = useContext(ThemeContext);
  return <p>Current theme: {theme}</p>;
}
```

### `useMemo` & `useCallback` — Performance

```tsx
// useMemo: memoize an expensive computed value
const sorted = useMemo(
  () => items.sort((a, b) => a.price - b.price),
  [items] // recompute only when items changes
);

// useCallback: memoize a function (useful when passing callbacks to children)
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

> Don't reach for these first. Optimize when you have a measured performance problem.

---

## 5. Component Patterns

### Lifting State Up

When two sibling components need to share state, move it to their common parent.

```tsx
function Parent() {
  const [value, setValue] = useState("");

  return (
    <>
      <Input value={value} onChange={setValue} />
      <Display value={value} />
    </>
  );
}
```

### Children Prop

```tsx
interface CardProps {
  title: string;
  children: React.ReactNode; // anything renderable
}

function Card({ title, children }: CardProps) {
  return (
    <div className="border rounded p-4">
      <h2 className="font-bold">{title}</h2>
      {children}
    </div>
  );
}

// Usage
<Card title="My Card">
  <p>This is the body content.</p>
</Card>
```

### Custom Hooks — Extract & Reuse Logic

Any function starting with `use` that calls other hooks.

```tsx
// useFetch.ts
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((d) => setData(d))
      .catch((e) => setError(e))
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// Usage
const { data, loading } = useFetch<User[]>("/api/users");
```

### Event Handler Typing

```tsx
// Input change
function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  setValue(e.target.value);
}

// Form submit
function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
  // ...
}

// Button click
function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
  // ...
}
```

---

## 6. Tailwind CSS

Tailwind is **utility-first** — you compose design by stacking class names. No custom CSS files needed for most things.

### The Mental Model

```
Traditional CSS:       Tailwind:
.btn {                 <button class="px-4 py-2 bg-blue-500
  padding: 16px;                      text-white rounded
  background: blue;                   hover:bg-blue-600">
  border-radius: 4px;
}
```

### Layout

```html
<!-- Flexbox -->
<div class="flex items-center justify-between gap-4">...</div>
<div class="flex flex-col gap-2">...</div>

<!-- Grid -->
<div class="grid grid-cols-3 gap-4">...</div>
<div class="grid grid-cols-[1fr_2fr_1fr]">...</div> <!-- custom columns -->

<!-- Centering (the classic) -->
<div class="flex items-center justify-center h-screen">...</div>
```

### Spacing

Tailwind uses a 4px base unit. `p-1` = 4px, `p-4` = 16px, `p-8` = 32px.

```
p-{n}   → padding all sides
px-{n}  → padding left & right
py-{n}  → padding top & bottom
pt/pr/pb/pl → individual sides

m-{n}   → margin (same pattern)
gap-{n} → gap in flex/grid
space-x-{n} → horizontal space between children
```

### Sizing

```
w-full    → width: 100%
w-1/2     → width: 50%
w-64      → width: 256px (64 * 4)
w-[300px] → arbitrary value
h-screen  → height: 100vh
max-w-xl  → max-width: 36rem
```

### Typography

```
text-sm / text-base / text-lg / text-xl / text-2xl ... text-9xl
font-thin / font-normal / font-medium / font-semibold / font-bold
text-center / text-left / text-right
tracking-tight / tracking-wide   → letter-spacing
leading-tight / leading-loose    → line-height
text-gray-500 / text-blue-600    → color
uppercase / lowercase / capitalize
truncate                         → overflow ellipsis
```

### Colors

Tailwind has a full color palette: `slate`, `gray`, `zinc`, `red`, `orange`, `yellow`, `green`, `blue`, `indigo`, `purple`, `pink`, etc.

Each color has shades 50–950:

```
bg-blue-50    → very light blue background
bg-blue-500   → medium blue background
bg-blue-900   → very dark blue background
text-red-600  → red text
border-gray-200
```

### Borders & Shadows

```
border          → 1px border
border-2        → 2px border
border-gray-300 → border color
rounded         → border-radius: 4px
rounded-md      → 6px
rounded-lg      → 8px
rounded-full    → 9999px (circle/pill)

shadow-sm / shadow / shadow-md / shadow-lg / shadow-xl
```

### Hover, Focus, Other States

```
hover:bg-blue-600   → on hover
focus:outline-none  → on focus
active:scale-95     → on click
disabled:opacity-50 → when disabled
```

### Responsive Design (Mobile-First)

Breakpoints: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px), `2xl` (1536px).

```html
<!-- Default (mobile), then override at md and lg -->
<div class="text-sm md:text-base lg:text-lg">...</div>
<div class="flex flex-col md:flex-row">...</div>
<div class="hidden md:block">Only on md+</div>
```

### Dark Mode

```js
// tailwind.config.js
export default {
  darkMode: "class", // toggle by adding 'dark' class to <html>
};
```

```html
<div class="bg-white text-black dark:bg-gray-900 dark:text-white">...</div>
```

### Arbitrary Values

When the built-in scale isn't enough:

```html
<div class="w-[372px] top-[117px] bg-[#1a1a2e] text-[13px]">...</div>
```

---

## 7. Combining All Three

### A Typed, Styled Component

```tsx
interface ButtonProps {
  label: string;
  variant?: "primary" | "secondary" | "danger";
  onClick: () => void;
  disabled?: boolean;
}

const variantClasses: Record<ButtonProps["variant"] & string, string> = {
  primary: "bg-blue-600 hover:bg-blue-700 text-white",
  secondary: "bg-gray-200 hover:bg-gray-300 text-gray-800",
  danger: "bg-red-600 hover:bg-red-700 text-white",
};

function Button({
  label,
  variant = "primary",
  onClick,
  disabled = false,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`
        px-4 py-2 rounded-md font-medium transition-colors
        disabled:opacity-50 disabled:cursor-not-allowed
        ${variantClasses[variant]}
      `}
    >
      {label}
    </button>
  );
}
```

### A Typed Fetch + Display Component

```tsx
interface Post {
  id: number;
  title: string;
  body: string;
}

function PostList() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/posts?_limit=5")
      .then((r) => r.json())
      .then((data: Post[]) => {
        setPosts(data);
        setLoading(false);
      });
  }, []);

  if (loading) {
    return <p className="text-center text-gray-500 mt-8">Loading...</p>;
  }

  return (
    <ul className="space-y-4 max-w-xl mx-auto mt-8">
      {posts.map((post) => (
        <li key={post.id} className="border border-gray-200 rounded-lg p-4 shadow-sm">
          <h2 className="font-semibold text-lg capitalize">{post.title}</h2>
          <p className="text-gray-600 text-sm mt-1">{post.body}</p>
        </li>
      ))}
    </ul>
  );
}
```

### Controlled Form

```tsx
function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit} className="flex flex-col gap-4 max-w-sm mx-auto">
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        className="border border-gray-300 rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        className="border border-gray-300 rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        type="submit"
        className="bg-blue-600 text-white py-2 rounded hover:bg-blue-700 transition-colors"
      >
        Log In
      </button>
    </form>
  );
}
```

---

## 8. Common Gotchas

| Gotcha | Why it happens | Fix |
|--------|---------------|-----|
| State update not reflected immediately | `setState` is async | Use the updated state in the next render, or use a callback |
| Infinite loop in `useEffect` | Object/array in deps changes every render | Memoize with `useMemo`/`useCallback`, or use primitives in deps |
| `key` warning on lists | React needs keys to track list items | Add `key={uniqueId}` to each list element |
| `Cannot read property of null` | Accessing state before data is loaded | Guard with `if (!data) return null` or optional chaining |
| TypeScript `any` creeping in | Lazy typing | Always type API responses with an interface |
| Tailwind classes not applying | Not in `content` paths, or typo in class name | Check `tailwind.config.js` content; use VS Code Tailwind IntelliSense |
| `className` vs `class` | JSX is not HTML | Always use `className` in JSX |
| Mutating state directly | React doesn't detect mutation | Always return a new object/array |
| Missing `return` in component | Component renders nothing | Add explicit `return` or use arrow function body |
| Stale closure in `useEffect` | Closure captures old value of variable | Add variable to deps array |

---

## 9. File Structure Convention

```
src/
├── assets/            # Images, fonts, static files
├── components/        # Reusable UI components
│   ├── Button.tsx
│   ├── Card.tsx
│   └── Navbar.tsx
├── hooks/             # Custom hooks
│   ├── useFetch.ts
│   └── useAuth.ts
├── pages/             # Page-level components (if using routing)
│   ├── Home.tsx
│   └── About.tsx
├── types/             # Shared TypeScript interfaces/types
│   └── index.ts
├── utils/             # Pure helper functions
│   └── formatDate.ts
├── App.tsx
├── main.tsx
└── index.css
```

**Naming conventions:**
- Components: `PascalCase` (`UserCard.tsx`)
- Hooks: `camelCase` starting with `use` (`useLocalStorage.ts`)
- Utilities: `camelCase` (`formatDate.ts`)
- Types/interfaces: `PascalCase` (`interface UserProfile`)

---

## 10. Quick Reference Card

### TypeScript

```ts
// Types
string | number | boolean | null | undefined | void | any | unknown | never

// Utility types
Partial<T>         // all fields optional
Required<T>        // all fields required
Readonly<T>        // all fields readonly
Record<K, V>       // object with keys K and values V
Pick<T, 'a'|'b'>  // subset of T
Omit<T, 'a'>      // T without 'a'
Array<T> = T[]
```

### React Hooks at a Glance

```ts
useState<T>(init)                    → [value, setter]
useEffect(fn, deps?)                 → void (side effects)
useRef<T>(init)                      → { current: T }
useContext(MyContext)                 → context value
useMemo(() => compute, deps)         → memoized value
useCallback(fn, deps)                → memoized function
useReducer(reducer, init)            → [state, dispatch]
```

### Tailwind Cheatsheet

```
LAYOUT:    flex grid block inline hidden
POSITION:  relative absolute fixed sticky
FLEX:      flex-row flex-col items-center justify-between flex-wrap gap-4
GRID:      grid-cols-2 grid-cols-3 col-span-2
SPACING:   p-4 px-6 py-2 m-auto mt-4 space-y-2 gap-4
SIZING:    w-full w-1/2 w-64 h-screen max-w-lg min-h-0
TEXT:      text-sm text-xl font-bold text-center text-gray-700 truncate
COLOR:     bg-blue-500 text-white border-gray-300
BORDER:    border rounded-lg border-2 border-blue-500
SHADOW:    shadow shadow-md shadow-lg
EFFECTS:   opacity-50 transition-colors duration-200 cursor-pointer
STATES:    hover:bg-blue-700 focus:ring-2 active:scale-95 disabled:opacity-50
RESPONSIVE: sm:flex md:grid lg:hidden xl:text-lg
DARK:      dark:bg-gray-900 dark:text-white
```

---

> **Tip:** Install the [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss) VS Code extension — it gives you autocomplete, hover previews, and linting for Tailwind classes. Also grab the [TypeScript Error Translator](https://marketplace.visualstudio.com/items?itemName=mattpocock.ts-error-translator) extension to make TS errors human-readable.

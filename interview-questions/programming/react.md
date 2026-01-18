# Complete React Interview Guide

Master React with 45 real-world interview questions covering debugging, performance, hooks, testing, and architecture. Practice interactive coding challenges that mirror actual senior engineer scenarios.

**Companies that ask these questions:** Meta | Google | Amazon | Netflix | Airbnb | Stripe

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | A critical component is crashing in production. How would yo... | Debugging | Debugging & Tools |
| 2 | This statically generated page is showing stale data. How do... | Practical | SSG & Data Fetching |
| 3 | This component is failing its test. Fix the component to mak... | Practical | Testing |
| 4 | This form doesn't reset correctly. Fix the bug and refactor ... | Debugging | State Management |
| 5 | An expensive component is re-rendering unnecessarily. How wo... | Architecture | Performance |
| 6 | This data fetching component has a race condition. How do yo... | Debugging | Data Fetching |
| 7 | This component causes a hydration mismatch error. Why and ho... | Debugging | SSR & Hydration |
| 8 | This component has hardcoded text. Refactor it to support mu... | Practical | Internationalization |
| 9 | This component is not reusable. Refactor it using the compos... | Architecture | Component Patterns |
| 10 | This legacy class component mixes logic and UI. Refactor it ... | Practical | Refactoring |
| 11 | Why does this component log the old state value after callin... | Conceptual | React Fundamentals |
| 12 | A child component is failing to update the parent's state. W... | Debugging | Props & State |
| 13 | Clarify the core React concepts: Element, Component, and Nod... | Conceptual | React Fundamentals |
| 14 | Your app's state management is a mess. How would Flux archit... | Architecture | State Management |
| 15 | Your UI freezes during large updates. How does React Fiber h... | Conceptual | React Internals |
| 16 | This list re-renders inefficiently. How does React's reconci... | Debugging | Performance |
| 17 | This component makes duplicate API calls in development. How... | Debugging | Debugging & Tools |
| 18 | Your team is evaluating React. How would you explain its cor... | Conceptual | React Fundamentals |
| 19 | This component has accessibility ID issues. Fix it with `use... | Practical | Hooks & Patterns |
| 20 | This modal is clipped by its parent. Fix it using a React Po... | Practical | Component Patterns |
| 21 | This component re-renders unnecessarily due to context objec... | Debugging | Performance |
| 22 | This button should increment twice, but only increments once... | Debugging | State Management |
| 23 | An expensive component is re-rendering unnecessarily. How wo... | Debugging | Performance |
| 24 | This component flickers when resizing. Fix it using the corr... | Practical | Hooks & Patterns |
| 25 | This data fetching component is causing an infinite loop. Ho... | Debugging | Data Fetching |
| 26 | This component is crashing with a JSX syntax error. How do y... | Debugging | React Fundamentals |
| 27 | This component is breaking layout. Fix it using a React Frag... | Practical | Component Patterns |
| 28 | This component is trying to modify a prop directly. How do y... | Debugging | Props & State |
| 29 | This application suffers from 'prop drilling.' How do you fi... | Practical | State Management |
| 30 | This component is crashing due to a 'Rules of Hooks' violati... | Debugging | Hooks & Patterns |
| 31 | This custom component is not forwarding its ref. How do you ... | Practical | Component Patterns |
| 32 | This `useEffect` is logging a stale value. How do you fix it... | Debugging | Hooks & Patterns |
| 33 | This component's UI is not updating. Why is direct state mut... | Debugging | State Management |
| 34 | This component has issues managing a timer ID. Fix it with `... | Practical | Hooks & Patterns |
| 35 | This optimized component re-renders unnecessarily. How do yo... | Practical | Performance |
| 36 | This large component is slowing down initial load. Implement... | Practical | Performance |
| 37 | This component is crashing the entire app. Implement an Erro... | Practical | Debugging & Tools |
| 38 | This component performs expensive calculations unnecessarily... | Practical | Performance |
| 39 | This component re-renders unnecessarily. How do you prevent ... | Debugging | Performance |
| 40 | This component's state logic is complex. Refactor it with `u... | Refactoring | State Management |
| 41 | These components duplicate logic. Refactor them using a High... | Refactoring | Component Patterns |
| 42 | These components duplicate logic. Refactor them using the Re... | Refactoring | Component Patterns |
| 43 | These components duplicate stateful logic. Refactor them usi... | Refactoring | Hooks & Patterns |
| 44 | This component manages loading state imperatively. Refactor ... | Refactoring | Component Patterns |
| 45 | This form uses an uncontrolled input. Refactor it to be a co... | Refactoring | Forms & Input |

---

## What You'll Learn

- Debug production crashes using Error Boundaries and React DevTools
- Fix race conditions and infinite loops in data fetching
- Optimize Context performance and prevent unnecessary re-renders
- Resolve SSR hydration mismatches and accessibility issues
- Refactor class components to modern Hooks patterns
- Implement advanced patterns: HOCs, Render Props, Custom Hooks
- Master component composition and code splitting
- Write and fix failing React tests
- Understand React internals: Fiber, Reconciliation, Strict Mode
- Handle forms, refs, portals, and Suspense

---

## Interview Questions

### Question 1: A critical component is crashing in production. How would you debug it?

**Type:** Debugging | **Category:** Debugging & Tools

**Step 1: Containment with Error Boundaries**

My first step isn't to fix the bug—it's to stop the bleeding and gather intel.

- **Action:** Wrap the dashboard in an **Error Boundary**
- **Why:** It catches errors in the component tree, renders a fallback UI, and logs the exact error + stack trace to monitoring (Sentry, Datadog)

**Step 2: Inspection with React DevTools**

With error logs coming in, I reproduce the bug locally.

- **Components Tab:** Inspect props and state of the crashing component. Look for `undefined` values or impossible states.
- **Profiler:** Record the rendering sequence to see what's changing on each commit.

**Step 3: Targeted Analysis with Breakpoints**

Once I have a theory, I dive into the code.

- Use `debugger;` or browser breakpoints instead of console.log
- Inspect the call stack, variable values, and step through line-by-line

This methodical process—Contain, Gather, Inspect, Analyze—is how senior engineers debug complex production issues.

---

### Question 2: This statically generated page is showing stale data. How do you fetch and display live data on the client?

**Type:** Practical | **Category:** SSG & Data Fetching

A team is using Static Site Generation (SSG) for their blog to ensure the pages are fast and scalable. The blog post content is generated at build time.

They want to add a "Live Viewers" count to each post. This number should be fresh, client-side data. A junior developer was tasked with this, but the viewer count is always a stale number that was generated at build time; it never updates to a live count.

You've been given the `BlogPost` component. It receives `postData` as a prop, which represents the data that was fetched and rendered into static HTML at build time.

Your task is to fix the component so that it fetches the *live* viewer count from a (mock) API when the page loads on the client, and updates the UI to show the fresh data.

<SmartCodeRunner mode="run" title="BlogPost - Fetch Live Data on SSG Page">
  <Fragment slot="code">
```jsx


// Mock API to get the live viewer count
const getLiveViewerCount = (postId) => {
  console.log(`Fetching live count for post ${postId}...`);
  return new Promise(resolve => {
    setTimeout(() => {
      const liveCount = Math.floor(Math.random() * 1000) + 50;
      resolve(liveCount);
    }, 1200);
  });
};

// This component receives data generated at build time.
function BlogPost({ postData }) {
  // The bug: The component only uses the stale `viewers` prop.
  // It never fetches the live data from the client.
  const [post, setPost] = useState(postData);

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <hr />
      <p>Live Viewers: {post.viewers}</p>
    </div>
  );
}

// To simulate the SSG environment, we'll define the static data
// that would have been generated at build time.
export default function App() {
  const staticData = {
    id: 1,
    title: 'My Fast SSG Blog Post',
    content: 'This content was generated at build time for maximum speed!',
    viewers: 10, // This is the stale viewer count from the build
  };
  return <BlogPost postData={staticData} />;
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the fix
const code = document.querySelector('[data-editor]').textContent;

// Check that useEffect is used to fetch data
if (!code.match(/BlogPost[\s\S]*?useEffect/)) {
  throw new Error('Test failed: Use useEffect in BlogPost to fetch live data on the client.');
}

// Check that getLiveViewerCount is called
if (!code.match(/useEffect[\s\S]*?getLiveViewerCount/)) {
  throw new Error('Test failed: Call getLiveViewerCount inside useEffect to fetch the live viewer count.');
}

// Check that state is updated with live data
if (!code.match(/useEffect[\s\S]*?setPost/)) {
  throw new Error('Test failed: Update the post state with the live viewer count inside useEffect.');
}

// Success!
console.log('✓ All tests passed! SSG page now fetches live client-side data.');
console.log('✓ The viewer count will update after the API call completes.');
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer="I would switch from SSG to SSR (Server-Side Rendering) so that the data is always fresh on each request."
  seniorAnswer="I would keep SSG for the main content (fast initial load) but use `useEffect` with an empty dependency array to fetch the live viewer count on the client after the page loads. This hybrid approach gives us the best of both worlds: instant static content delivery with dynamic, client-side data updates for parts that need to be fresh.">

The issue here is a misunderstanding of how Static Site Generation (SSG) works with dynamic data. The entire page, including the `viewers: 10` data, is pre-rendered into a static HTML file at build time. This makes the initial load incredibly fast.

However, because it's static, that viewer count will be stale forever until the site is rebuilt.

To display live, dynamic data on a static page, you must fetch it from the client-side after the initial page load. The perfect tool for this is the `useEffect` hook. By adding a `useEffect` that runs once on component mount (`[]`), we can call our API, get the fresh data, and update the component's state, which triggers a re-render to show the live viewer count.

This approach gives you the best of both worlds: a super-fast initial page load from the static HTML, supplemented by dynamic, live data fetched on the client.

Here is the corrected implementation:

```jsx


const getLiveViewerCount = (postId) => {
  console.log(`Fetching live count for post ${postId}...`);
  return new Promise(resolve => {
    setTimeout(() => {
      const liveCount = Math.floor(Math.random() * 1000) + 50;
      resolve(liveCount);
    }, 1200);
  });
};

function BlogPost({ postData }) {
  const [post, setPost] = useState(postData);

  // FIX: Fetch live data on the client after the static page loads.
  useEffect(() => {
    getLiveViewerCount(post.id).then(liveCount => {
      // Update the state with the new viewer count
      setPost(currentPost => ({ ...currentPost, viewers: liveCount }));
    });
  }, [post.id]); // Depend on post.id to re-fetch if the post changes

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <hr />
      <p>Live Viewers: {post.viewers}</p>
    </div>
  );
}

export default function App() {
  const staticData = {
    id: 1,
    title: 'My Fast SSG Blog Post',
    content: 'This content was generated at build time for maximum speed!',
    viewers: 10,
  };
  return <BlogPost postData={staticData} />;
}
```

</JuniorVsSenior>

---

### Question 3: This component is failing its test. Fix the component to make the test pass.

**Type:** Practical | **Category:** Testing

A developer wrote a simple `Accordion` component. It should display a title, and when the user clicks that title, it should reveal the hidden content.

They also wrote a unit test for this component using React Testing Library. The test codifies the exact requirements. However, the test is failing, which means the component has a bug.

This is a test-driven development (TDD) challenge. You are given the buggy component and the failing test.

Your task is to **read the test to understand the requirements**, then fix the `Accordion` component so that the test would pass.

<SmartCodeRunner mode="run" title="Accordion Component - Fix to Pass the Test">
  <Fragment slot="code">
```jsx


// The buggy component
function Accordion({ title, children }) {
  // The state seems to be here, but is it used correctly?
  const [isOpen, setIsOpen] = useState(false);

  const handleToggle = () => {
    // THE BUG: This handler is not implemented correctly.
    // It should change the value of `isOpen`.
  };

  return (
    <div>
      <button onClick={handleToggle}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}

// This is a wrapper to demonstrate the component visually.
export default function App() {
  return (
    <Accordion title="Click to Open">
      <p>Here is the hidden content.</p>
    </Accordion>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the fix
const code = document.querySelector('[data-editor]').textContent;

// Check that handleToggle updates the state
if (!code.includes('setIsOpen(!isOpen)') && !code.includes('setIsOpen(prev => !prev)')) {
  throw new Error('Test failed: handleToggle should toggle the isOpen state using setIsOpen(!isOpen).');
}

// Check that the toggle function is implemented
const handleToggleMatch = code.match(/handleToggle\s*=\s*\(\s*\)\s*=>\s*{[\s\S]*?setIsOpen/);
if (!handleToggleMatch) {
  throw new Error('Test failed: handleToggle must call setIsOpen to update state.');
}

// Success!
console.log('✓ All tests passed! The Accordion component now toggles content correctly.');
console.log('✓ Clicking the button will show/hide the content as expected.');
```
  </Fragment>
</SmartCodeRunner>

 {
    // THE BUG: This handler is not implemented correctly.
    // It should change the value of \`isOpen\`.
  };

  return (
    <div>
      <button onClick={handleToggle}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}

export default function App() {
  return (
    <Accordion title="Click to Open">
      <p>Here is the hidden content.</p>
    </Accordion>
  );
}`}
  rightAnswer={`import React, { useState } from 'react';

function Accordion({ title, children }) {
  const [isOpen, setIsOpen] = useState(false);

  const handleToggle = () => {
    // FIX: Update the state to the opposite of its current value.
    setIsOpen(!isOpen);
  };

  return (
    <div>
      <button onClick={handleToggle}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}

export default function App() {
  return (
    <Accordion title="Click to Open">
      <p>Here is the hidden content.</p>
    </Accordion>
  );
}`}
  wrongExplanation="The failing test acts as a perfect specification for our component. The test expects: 1) The accordion's content should be hidden by default (useState(false) handles this). 2) After clicking the title button, the content should appear. The bug is in the handleToggle function. It's empty, so clicking the button does nothing to the isOpen state."
  rightExplanation="To make the test pass, we simply need to implement the state-toggling logic in the handleToggle function. When the button is clicked, we need to set the isOpen state to the opposite of its current value using setIsOpen(!isOpen). With this change, clicking the button will toggle the isOpen state, the component will re-render, and the content will be correctly displayed or hidden, making the test pass."
  client:load
/>

---

### Question 4: This form doesn't reset correctly. Fix the bug and refactor to a more robust solution.

**Type:** Debugging | **Category:** State Management

A developer has built a `SurveyForm` component. The form has several pieces of state: `name`, `email`, and `rating`.

After the user clicks "Submit", the form should completely reset to its initial, empty state. However, a bug was reported: after submission, the `name` and `email` fields clear, but the `rating` field remains filled.

The current implementation uses a manual `handleReset` function, which is becoming hard to maintain as more fields are added.

This is a two-part challenge:

1. **Fix the Bug:** Identify why the `rating` field is not resetting and apply the immediate fix.
2. **Refactor:** The manual reset approach is brittle. Refactor the application to use a more declarative and robust React pattern for resetting the component's entire state at once, without needing to manually reset each state variable.

<SmartCodeRunner mode="run" title="Survey Form - Fix Reset Bug and Refactor">
  <Fragment slot="code">
```jsx


// The form component with a buggy reset logic
function SurveyForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [rating, setRating] = useState(0);

  const handleReset = () => {
    setName('');
    setEmail('');
    // THE BUG: The developer forgot to reset the rating state.
  };

  return (
    <div>
      <form>
        <input
          type="text"
          placeholder="Name"
          value={name}
          onChange={e => setName(e.target.value)}
        />
        <br />
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
        <br />
        <select value={rating} onChange={e => setRating(e.target.value)}>
          <option value={0}>Select rating...</option>
          <option value={1}>1 Star</option>
          <option value={2}>2 Stars</option>
          <option value={3}>3 Stars</option>
        </select>
      </form>
      <button onClick={handleReset}>Reset Form</button>
    </div>
  );
}

export default function App() {
  return <SurveyForm />;
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the refactored solution using the key prop pattern
const code = document.querySelector('[data-editor]').textContent;

// Check that the solution uses the key prop pattern (best practice)
if (!code.includes('key={') && !code.includes('key =')) {
  throw new Error('Test failed: Refactor to use the key prop pattern for robust form reset. The parent should manage a formKey state.');
}

// Check for formKey or similar state in parent
if (!code.includes('formKey') && !code.includes('resetKey') && !code.includes('componentKey')) {
  throw new Error('Test failed: Create a state variable (formKey) in the parent component to use as the key prop.');
}

// Verify the key changes to trigger reset
if (!code.includes('setFormKey') && !code.includes('setResetKey') && !code.includes('setComponentKey')) {
  throw new Error('Test failed: The parent should update the key state to trigger a reset.');
}

// Success!
console.log('✓ All tests passed! Form reset refactored using the key prop pattern.');
console.log('✓ This declarative approach is more robust and scales better.');
```
  </Fragment>
</SmartCodeRunner>

 {
    setName('');
    setEmail('');
    // THE BUG: The developer forgot to reset the rating state.
  };

  return (
    <div>
      <form>
        <input
          type="text"
          placeholder="Name"
          value={name}
          onChange={e => setName(e.target.value)}
        />
        <br />
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
        <br />
        <select value={rating} onChange={e => setRating(e.target.value)}>
          <option value={0}>Select rating...</option>
          <option value={1}>1 Star</option>
          <option value={2}>2 Stars</option>
          <option value={3}>3 Stars</option>
        </select>
      </form>
      <button onClick={handleReset}>Reset Form</button>
    </div>
  );
}

export default function App() {
  return <SurveyForm />;
}`}
  rightAnswer={`import React, { useState } from 'react';

// The form component no longer needs to know how to reset itself.
// It's just a simple, self-contained form.
function SurveyForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [rating, setRating] = useState(0);

  return (
    <form>
      <input
        type="text"
        placeholder="Name"
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <br />
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      <br />
      <select value={rating} onChange={e => setRating(e.target.value)}>
        <option value={0}>Select rating...</option>
        <option value={1}>1 Star</option>
        <option value={2}>2 Stars</option>
        <option value={3}>3 Stars</option>
      </select>
    </form>
  );
}

// The parent component now fully controls the reset behavior.
export default function App() {
  // 1. Create a state variable to use as the key.
  const [formKey, setFormKey] = useState(0);

  const handleReset = () => {
    console.log('Form reset!');
    // 2. To reset the form, just change the key.
    setFormKey(currentKey => currentKey + 1);
  };

  return (
    <div>
      {/* 3. Pass the state variable as the key prop. */}
      <SurveyForm key={formKey} />
      <button onClick={handleReset}>Reset Form</button>
    </div>
  );
}`}
  wrongExplanation="The immediate bug is simple: the developer forgot to add setRating(0) to the handleReset function. This highlights why this manual, imperative approach is a bad practice. As the form grows, it's easy to forget to update the reset handler, leading to bugs."
  rightExplanation="A much more robust and declarative way to reset a component's state is to change its key prop. In React, the key prop is a special instruction. When a component's key changes, React will throw away the old component instance (destroying its DOM node and all of its internal state) and create a brand new one. This gives you a 'fresh start' for the component, which is exactly what you want when resetting a form. This approach is far superior. The SurveyForm is simpler, and the parent can reliably reset it without knowing anything about its internal state. If you add 20 more fields to the form, the reset logic in the parent never has to change."
  client:load
/>

---

### Question 5: An expensive component is re-rendering unnecessarily. How would you fix this state management issue?

**Type:** Architecture | **Category:** Performance

## The Scenario

A junior developer built a new feature using React Context to manage the application's global state. The state object contains everything: the current user's info, and the UI theme (light/dark mode).

They've noticed a performance problem. The `UserWelcome` component, which is expensive to render, re-renders every single time the theme is changed. This is making the UI feel sluggish.

## The Challenge

You've been given the code that demonstrates this issue.

1. Explain why the `UserWelcome` component re-renders every time the "Toggle Theme" button is clicked.
2. Describe two different strategies to fix this performance issue so that `UserWelcome` only re-renders when the `user` data it cares about actually changes.

<SmartCodeRunner mode="run" title="Context Performance Issue - Fix Unnecessary Re-renders">
  <Fragment slot="code">
```jsx


// The monolithic context holding all global state
const AppContext = createContext();

// A component that is "expensive" to render
function UserWelcome() {
  const { user } = useContext(AppContext);
  console.log('Re-rendering UserWelcome (expensive)...');
  return <h1>Welcome, {user.name}!</h1>;
}

// A component to change the theme
function ThemeToggler() {
  const { theme, setTheme } = useContext(AppContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </button>
  );
}

// The main App provider
export default function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  const contextValue = { user, setUser, theme, setTheme };

  return (
    <AppContext.Provider value={contextValue}>
      <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff', padding: '20px' }}>
        <ThemeToggler />
        <hr />
        <UserWelcome />
      </div>
    </AppContext.Provider>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the fix
const code = document.querySelector('[data-editor]').textContent;

// Check for split contexts (best solution)
const hasThemeContext = code.includes('ThemeContext') && code.includes('createContext()');
const hasUserContext = code.includes('UserContext') && code.includes('createContext()');

if (hasThemeContext && hasUserContext) {
  console.log('✓ Excellent! You split the contexts - this is the best practice solution.');
  console.log('✓ UserWelcome now only subscribes to UserContext and won\'t re-render on theme changes.');
} else {
  throw new Error('Test failed: Split the monolithic AppContext into separate ThemeContext and UserContext.');
}

// Verify components use the correct context
if (!code.match(/UserWelcome[\s\S]*?useContext\(UserContext\)/)) {
  throw new Error('Test failed: UserWelcome should use useContext(UserContext), not AppContext.');
}

if (!code.match(/ThemeToggler[\s\S]*?useContext\(ThemeContext\)/)) {
  throw new Error('Test failed: ThemeToggler should use useContext(ThemeContext), not AppContext.');
}

console.log('✓ All tests passed! Context performance issue fixed correctly.');
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: The Problem with Monolithic Context

<JuniorVsSenior
  juniorAnswer={{
    title: "Monolithic Context (Everything in One)",
    code: `const AppContext = createContext();

function UserWelcome() {
  const { user } = useContext(AppContext);
  return <h1>Welcome, {user.name}!</h1>;
}

function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  const contextValue = { user, setUser, theme, setTheme };

  return (
    <AppContext.Provider value={contextValue}>
      <UserWelcome />
    </AppContext.Provider>
  );
}`,
    explanation: "Combining unrelated state (user and theme) in one context causes all consumers to re-render when any part of the context changes. UserWelcome re-renders even though it only needs user data."
  }}
  seniorAnswer={{
    title: "Split Contexts (Separation of Concerns)",
    code: `const ThemeContext = createContext();
const UserContext = createContext();

function UserWelcome() {
  const { user } = useContext(UserContext);
  return <h1>Welcome, {user.name}!</h1>;
}

function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <UserWelcome />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}`,
    explanation: "Separating unrelated state into different contexts allows components to subscribe only to what they need. UserWelcome only subscribes to UserContext, so theme changes don't trigger re-renders."
  }}
/>

### The Performance Issue

The performance issue is a classic pitfall of using React Context. When a component consumes a context (with `useContext`), it subscribes to **all** changes in that context's `value`.

In this code, we have a single, monolithic `AppContext` that holds both `user` and `theme`. The `UserWelcome` component only needs `user`, but because it's subscribed to the entire `AppContext`, React will re-render it whenever *any* value in the context provider's `value` object changes. Clicking "Toggle Theme" creates a new `contextValue` object with a new `theme`, triggering a re-render in every component that consumes `AppContext`, including the expensive `UserWelcome`.

Here are two ways to solve this:

### Solution 1: Split the Contexts (Best Practice)

The most robust solution is to separate unrelated state into different contexts. Components can then subscribe only to the state they care about.

```jsx


// Create two separate contexts
const ThemeContext = createContext();
const UserContext = createContext();

function UserWelcome() {
  // Subscribe ONLY to the UserContext
  const { user } = useContext(UserContext);
  console.log('Re-rendering UserWelcome (expensive)...');
  return <h1>Welcome, {user.name}!</h1>;
}

function ThemeToggler() {
  // Subscribe ONLY to the ThemeContext
  const { theme, setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </button>
  );
}

export default function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  // Nest the providers
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff', padding: '20px' }}>
          <ThemeToggler />
          <hr />
          <UserWelcome />
        </div>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}
```

Now, when the theme changes, only the `ThemeContext.Provider` gets a new value, and only its consumers (`ThemeToggler` and the `div`) will re-render. `UserWelcome` is unaffected.

### Solution 2: Memoization

A quicker, but often more brittle, solution is to use `React.memo` to wrap the expensive component. `React.memo` is a higher-order component that prevents a component from re-rendering if its props haven't changed.

```jsx
// Wrap the expensive component in React.memo
const MemoizedUserWelcome = React.memo(function UserWelcome() {
  const { user } = useContext(AppContext);
  console.log('Re-rendering UserWelcome (expensive)...');
  return <h1>Welcome, {user.name}!</h1>;
});

// ... in the App component ...
// <MemoizedUserWelcome />
```

This doesn't work by itself! `React.memo` does a shallow comparison of props, but `MemoizedUserWelcome` has no props. It still re-renders because the context value it depends on changes. To fix this, you have to combine `memo` with extracting the component and passing state down as props. This shows the complexity and why splitting contexts is often better. A better memoization strategy involves splitting the component and memoizing the child.

A more advanced technique is to use `useMemo` to create a stable value for parts of the context, but this can get complicated quickly. Splitting contexts is almost always the cleaner, more scalable solution.

---

### Question 6: This data fetching component has a race condition. How do you fix it?

**Type:** Debugging | **Category:** Data Fetching

**The Race Condition Sequence:**
1. `userId` becomes "2" → fetch starts (2000ms delay)
2. `userId` becomes "3" → new fetch starts (500ms delay)
3. Fetch "3" resolves first → UI shows "Charlie" ✓
4. Fetch "2" resolves late → UI overwrites to "Bob" ✗

**The Fix:** Return a cleanup function from `useEffect` that aborts the previous request when dependencies change.


<SmartCodeRunner mode="run" title="UserDetails Component - Fix the Race Condition">
  <Fragment slot="code">
```jsx


// Mock API with variable delay
const fetchUser = (id) => {
  const users = { '1': { name: 'Alice' }, '2': { name: 'Bob' }, '3': { name: 'Charlie' } };
  const delay = id === '2' ? 2000 : 500;
  return new Promise(resolve => {
    setTimeout(() => resolve(users[id]), delay);
  });
};

export default function UserDetails() {
  const [userId, setUserId] = useState('1');
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    // THE BUG: No cleanup - previous fetch will overwrite state
    fetchUser(userId).then(fetchedUser => {
      setUser(fetchedUser);
      setLoading(false);
    });
  }, [userId]);

  return (
    <div>
      <input type="text" value={userId} onChange={e => setUserId(e.target.value)} />
      <hr />
      {loading ? <p>Loading...</p> : user && <h2>{user.name}</h2>}
    </div>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
const code = document.querySelector('[data-editor]').textContent;
if (!code.includes('AbortController')) throw new Error('Test failed: Use AbortController to cancel requests.');
if (!code.includes('return ()')) throw new Error('Test failed: Add a cleanup function to abort the request.');
if (!code.includes('.abort()')) throw new Error('Test failed: Call controller.abort() in cleanup.');
console.log('✓ All tests passed! Race condition fixed correctly.');
```
  </Fragment>
</SmartCodeRunner>

## The Solution

```jsx
useEffect(() => {
  const controller = new AbortController();

  setLoading(true);
  fetchUser(userId, controller.signal)
    .then(fetchedUser => {
      setUser(fetchedUser);
      setLoading(false);
    })
    .catch(error => {
      if (error.name === 'AbortError') return; // Expected
      setLoading(false);
    });

  // Cleanup: abort when userId changes or unmounts
  return () => controller.abort();
}, [userId]);
```


---

### Quick Check

**In a useEffect hook, what is the primary purpose of the returned cleanup function?**

   A. To run code only once when the component first mounts
-> B. **To cancel side effects (like subscriptions or pending requests) from the previous render before the effect runs again**
   C. To handle errors that occur within the useEffect hook
   D. To update the component's state after the side effect is complete

<details>
<summary>See Answer</summary>

The cleanup function runs before the effect executes again (when dependencies change) and when the component unmounts. This is essential for preventing memory leaks, race conditions, and other issues with asynchronous operations.

</details>

---

### Question 7: This component causes a hydration mismatch error. Why and how to fix it?

**Type:** Debugging | **Category:** SSR & Hydration

**Why It Happens:**
- Server has no `window` → always renders "Desktop User"
- Client has `window` → might render "Mobile User" on first render
- React sees the mismatch and throws a warning

**The Rule:** First client render must be identical to server render. Use `useEffect` for client-only APIs.


<SmartCodeRunner mode="run" title="ResponsiveWelcome - Fix Hydration Mismatch">
  <Fragment slot="code">
```jsx


export default function ResponsiveWelcome() {
  // THE BUG: Server vs client render differently
  const isMobile =
    typeof window !== 'undefined' && window.innerWidth < 768;

  const message = isMobile
    ? 'Welcome, Mobile User!'
    : 'Welcome, Desktop User!';

  return <h1>{message}</h1>;
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
const code = document.querySelector('[data-editor]').textContent;
if (!code.includes('useState(false)')) throw new Error('Test failed: Use useState(false) to match server render.');
if (!code.match(/useEffect[\s\S]*?window\.innerWidth/)) throw new Error('Test failed: Move window check inside useEffect.');
if (!code.match(/useEffect[\s\S]*?setIsMobile/)) throw new Error('Test failed: Call setIsMobile in useEffect.');
console.log('✓ All tests passed! Hydration mismatch fixed correctly.');
```
  </Fragment>
</SmartCodeRunner>

## The Solution

```jsx
export default function ResponsiveWelcome() {
  // 1. Default state matches server render
  const [isMobile, setIsMobile] = useState(false);

  // 2. Client-only check runs after hydration
  useEffect(() => {
    const checkIsMobile = () => window.innerWidth < 768;
    setIsMobile(checkIsMobile());

    // Optional: handle resize
    const handleResize = () => setIsMobile(checkIsMobile());
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  const message = isMobile ? 'Welcome, Mobile User!' : 'Welcome, Desktop User!';
  return <h1>{message}</h1>;
}
```

The initial render uses `isMobile: false`, matching the server. Then `useEffect` runs and updates if needed.

---

### Question 8: This component has hardcoded text. Refactor it to support multiple languages.

**Type:** Practical | **Category:** Internationalization

A developer has built a new `Welcome` component. It works perfectly, but all the text is hardcoded in English. The product team now requires the application to support both English and German.

The current implementation is not scalable for adding new languages.

You've been given the component with the hardcoded text. Your task is to refactor it to support internationalization (i18n).

1. Create a simple translation dictionary to hold the strings for English (`en`) and German (`de`).
2. Implement a way to manage the application's current language (locale).
3. Connect the `LanguageSwitcher` buttons to change the locale.
4. Update the `Welcome` component to display the correct text based on the selected locale.

For this challenge, you don't need a full library like `react-i18next`; implementing a simple system using React state and context is sufficient to demonstrate the concept.

<SmartCodeRunner mode="run" title="Welcome Component - Add Internationalization">
  <Fragment slot="code">
```jsx


// --- Challenge: Refactor this section ---

// 1. The text is hardcoded in English
function Welcome() {
  return (
    <div>
      <h2>Welcome!</h2>
      <p>Thank you for visiting our application.</p>
    </div>
  );
}

// 2. The language switcher is not connected to any logic
function LanguageSwitcher() {
  return (
    <div>
      <button>English</button>
      <button>German</button>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <LanguageSwitcher />
      <hr />
      <Welcome />
    </div>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the i18n implementation
const code = document.querySelector('[data-editor]').textContent;

// Check for translations dictionary
if (!code.includes('translations') || !code.match(/translations\s*=\s*{/)) {
  throw new Error('Test failed: Create a translations object/dictionary with en and de keys.');
}

// Check for LocaleContext
if (!code.includes('LocaleContext') || !code.includes('createContext')) {
  throw new Error('Test failed: Create a LocaleContext using createContext to manage the locale state.');
}

// Check that Welcome uses context
if (!code.match(/Welcome[\s\S]*?useContext/)) {
  throw new Error('Test failed: Welcome component should use useContext to access the locale.');
}

// Check that LanguageSwitcher uses context
if (!code.match(/LanguageSwitcher[\s\S]*?useContext/)) {
  throw new Error('Test failed: LanguageSwitcher should use useContext to access setLocale.');
}

// Check for setLocale calls in buttons
if (!code.includes('setLocale')) {
  throw new Error('Test failed: LanguageSwitcher buttons should call setLocale to change the language.');
}

// Success!
console.log('✓ All tests passed! i18n system implemented correctly.');
console.log('✓ Clicking language buttons will now switch the displayed text.');
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState, useContext, createContext } from 'react';

// 1. The text is hardcoded in English
function Welcome() {
  return (
    <div>
      <h2>Welcome!</h2>
      <p>Thank you for visiting our application.</p>
    </div>
  );
}

// 2. The language switcher is not connected to any logic
function LanguageSwitcher() {
  return (
    <div>
      <button>English</button>
      <button>German</button>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <LanguageSwitcher />
      <hr />
      <Welcome />
    </div>
  );
}`}
  seniorAnswer={`import React, { useState, useContext, createContext } from 'react';

// 1. Create a dictionary for translations
const translations = {
  en: {
    title: 'Welcome!',
    body: 'Thank you for visiting our application.',
  },
  de: {
    title: 'Willkommen!',
    body: 'Danke für Ihren Besuch auf unserer Anwendung.',
  },
};

// 2. Create a context to manage the locale
const LocaleContext = createContext();

// 3. Refactor Welcome to use the context and dictionary
function Welcome() {
  const { locale } = useContext(LocaleContext);
  const { title, body } = translations[locale];

  return (
    <div>
      <h2>{title}</h2>
      <p>{body}</p>
    </div>
  );
}

// 4. Refactor LanguageSwitcher to change the locale
function LanguageSwitcher() {
  const { setLocale } = useContext(LocaleContext);
  return (
    <div>
      <button onClick={() => setLocale('en')}>English</button>
      <button onClick={() => setLocale('de')}>German</button>
    </div>
  );
}

// 5. The App component now provides the context
export default function App() {
  const [locale, setLocale] = useState('en');

  return (
    <LocaleContext.Provider value={{ locale, setLocale }}>
      <div>
        <LanguageSwitcher />
        <hr />
        <Welcome />
      </div>
    </LocaleContext.Provider>
  );
}`}
  juniorExplanation="Hardcoding text directly into components is an anti-pattern for any application that might need to support multiple languages. It tightly couples the component's logic to a single language, making translation a nightmare of finding and replacing strings."
  seniorExplanation="The standard approach is to abstract the text into a centralized 'dictionary' or set of translation files. The components then reference a key, and the system provides the correct string for that key based on the currently selected language. For this challenge, we can build a minimal i18n system using React Context to provide the current locale and a function to change it. This pattern decouples the components from the text content. Now, to add a new language, we only need to add a new entry to the translations object and a button in the LanguageSwitcher—we don't have to touch the Welcome component at all."
  client:load
/>

<InterviewQuiz
  question="For an application that needs to be translated, what is the main problem with hardcoding text strings directly into your components?"
  options={[
    "It makes the components render more slowly.",
    "It tightly couples the component to a single language, making it difficult to manage and scale translations.",
    "It increases the memory usage of the application.",
    "It prevents the use of functional components."
  ]}
  correctAnswer={1}
  explanation="Hardcoding text strings directly in components creates tight coupling between the component logic and a specific language. This makes translation management extremely difficult as you need to hunt through code to find and update strings. A centralized translation system with a dictionary/keys approach decouples the text from components, making it easy to add new languages and manage translations at scale."
  client:load
/>

---

### Question 9: This component is not reusable. Refactor it using the composition pattern.

**Type:** Architecture | **Category:** Component Patterns

A junior developer has created a `Dashboard` component. Inside, they've hardcoded three different types of panels: a `WelcomePanel`, a `ChartPanel`, and a `SettingsPanel`.

The code is very repetitive. Each panel has the same border, title styling, and padding, but the content inside is different. This makes the component hard to maintain and the panel styling is not reusable anywhere else.

Your task is to refactor this `Dashboard` component. Create a single, reusable `Panel` component that uses React's composition pattern (specifically the `children` prop) to render different content inside.

The final UI should look identical, but the code should be much cleaner, more modular, and follow the "Don't Repeat Yourself" (DRY) principle.

<SmartCodeRunner mode="run" title="Dashboard - Refactor Using Composition">
  <Fragment slot="code">
```jsx


// The styles for the panels are defined here for clarity.
const panelStyles = {
  border: '1px solid #ccc',
  borderRadius: '8px',
  padding: '16px',
  marginBottom: '16px',
  boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
};

const titleStyles = {
  margin: '0 0 10px 0',
  borderBottom: '1px solid #eee',
  paddingBottom: '10px',
};

// The anti-pattern: A monolithic component with repeated structure.
export default function Dashboard() {
  return (
    <div>
      {/* Welcome Panel */}
      <div style={panelStyles}>
        <h2 style={titleStyles}>Welcome</h2>
        <p>Welcome to your dashboard, Admin!</p>
      </div>

      {/* Chart Panel */}
      <div style={panelStyles}>
        <h2 style={titleStyles}>Analytics</h2>
        <p>Here are your user engagement charts.</p>
        <img src="https://via.placeholder.com/300x100.png?text=Chart" alt="Chart" />
      </div>

      {/* Settings Panel */}
      <div style={panelStyles}>
        <h2 style={titleStyles}>Settings</h2>
        <p>Configure your notification preferences.</p>
        <button>Edit Settings</button>
      </div>
    </div>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the refactor
const code = document.querySelector('[data-editor]').textContent;

// Check for Panel component
if (!code.includes('Panel')) {
  throw new Error('Test failed: Create a reusable Panel component.');
}

// Check that Panel uses children prop
if (!code.match(/Panel[\s\S]*?children/)) {
  throw new Error('Test failed: Panel component should accept and render the children prop.');
}

// Check that Panel uses title prop
if (!code.match(/Panel[\s\S]*?title/)) {
  throw new Error('Test failed: Panel component should accept a title prop.');
}

// Check that Dashboard uses Panel component at least 3 times
const panelMatches = code.match(/<Panel/g);
if (!panelMatches || panelMatches.length < 3) {
  throw new Error('Test failed: Dashboard should use the Panel component for all three panels.');
}

// Check that repeated div structure is removed
const divMatches = code.match(/<div style={panelStyles}>/g);
if (divMatches && divMatches.length > 1) {
  throw new Error('Test failed: Remove repeated div structures. The Panel component should encapsulate this.');
}

// Success!
console.log('✓ All tests passed! Dashboard refactored using composition pattern.');
console.log('✓ Panel component is now reusable across the application.');
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer="I would extract the panel styling into a CSS class and apply it to each div, which would reduce some duplication."
  seniorAnswer="I would create a reusable `Panel` component that accepts a `title` prop and uses `children` to compose its content. This leverages React's composition pattern, making the Panel independently reusable and testable. The Dashboard then becomes declarative, using `<Panel title='...'>content</Panel>` for each section, eliminating structural duplication while maintaining flexibility.">

The original code violates the DRY (Don't Repeat Yourself) principle. The container `div` with its `style={panelStyles}` and the `h2` title are repeated for every panel. This is a classic sign that a component should be extracted.

The best way to fix this is by creating a generic `Panel` component that encapsulates the repeated structure and styling. We can then use `props.children` to place any content we want inside this panel. This is a core React concept called **composition**. The `Panel` component "contains" the children you pass to it.

This makes our `Dashboard` component much cleaner and gives us a `Panel` component we can reuse anywhere in our application.

Here is the corrected implementation:

```jsx


const panelStyles = {
  border: '1px solid #ccc',
  borderRadius: '8px',
  padding: '16px',
  marginBottom: '16px',
  boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
};

const titleStyles = {
  margin: '0 0 10px 0',
  borderBottom: '1px solid #eee',
  paddingBottom: '10px',
};

// The reusable Panel component using composition
function Panel({ title, children }) {
  return (
    <div style={panelStyles}>
      <h2 style={titleStyles}>{title}</h2>
      {children}
    </div>
  );
}

// The refactored Dashboard, now clean and declarative
export default function Dashboard() {
  return (
    <div>
      <Panel title="Welcome">
        <p>Welcome to your dashboard, Admin!</p>
      </Panel>

      <Panel title="Analytics">
        <p>Here are your user engagement charts.</p>
        <img src="https://via.placeholder.com/300x100.png?text=Chart" alt="Chart" />
      </Panel>

      <Panel title="Settings">
        <p>Configure your notification preferences.</p>
        <button>Edit Settings</button>
      </Panel>
    </div>
  );
}
```

</JuniorVsSenior>


---

### Quick Check

**What is the primary advantage of using the composition pattern over inheritance in React?**

   A. It makes components run faster.
-> B. **It allows for greater flexibility and reusability by creating loosely coupled components.**
   C. It's the only way to pass data between components.
   D. It reduces the amount of JavaScript code that needs to be written.

<details>
<summary>See Answer</summary>

React favors composition over inheritance. The composition pattern using props.children and other props allows you to create flexible, reusable components that are loosely coupled. This makes them easier to maintain, test, and reuse across different parts of your application. Components become building blocks that can be combined in various ways without tight coupling.

</details>

---

### Question 10: This legacy class component mixes logic and UI. Refactor it to a modern functional component using Hooks.

**Type:** Practical | **Category:** Refactoring

You've been tasked with modernizing a part of a legacy React codebase. You find an old class component, `UserListContainer`, that fetches a list of users and renders them.

This component mixes data fetching, state management, and rendering logic all in one place. This makes the UI difficult to reuse or test in isolation. It follows the old "Container Component" pattern, but it's time for an update.

Refactor this legacy component. Your goal is to separate the logic from the presentation using modern React Hooks.

1. Create a **custom Hook** called `useUsers` to encapsulate the logic for fetching and managing the user data.
2. Create a purely **presentational component** called `UserList` that receives the users and loading state as props and is only responsible for rendering the UI.
3. Create a new `UserPage` component that uses your `useUsers` hook and your `UserList` component to render the final output.

The final UI should look and function identically to the original.

<SmartCodeRunner mode="run" title="Refactor Class Component to Hooks">
  <Fragment slot="code">
```jsx


// Mock API call
const fetchUsers = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' },
        { id: 3, name: 'Charlie' },
      ]);
    }, 1000);
  });
};

// The legacy class component that mixes concerns
export default class UserListContainer extends Component {
  state = {
    users: [],
    loading: true,
  };

  componentDidMount() {
    fetchUsers().then(users => {
      this.setState({ users, loading: false });
    });
  }

  render() {
    const { users, loading } = this.state;

    if (loading) {
      return <div>Loading users...</div>;
    }

    return (
      <div>
        <h2>Users</h2>
        <ul>
          {users.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      </div>
    );
  }
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the refactor
const code = document.querySelector('[data-editor]').textContent;

// Check for custom hook
if (!code.includes('useUsers') || !code.includes('const useUsers =') && !code.includes('function useUsers')) {
  throw new Error('Test failed: Create a custom hook called useUsers to encapsulate the data fetching logic.');
}

// Check that hook uses useState
if (!code.match(/useUsers[\s\S]*?useState/)) {
  throw new Error('Test failed: The useUsers hook should use useState to manage users and loading state.');
}

// Check that hook uses useEffect
if (!code.match(/useUsers[\s\S]*?useEffect/)) {
  throw new Error('Test failed: The useUsers hook should use useEffect to fetch data on mount.');
}

// Check for presentational component
if (!code.includes('UserList')) {
  throw new Error('Test failed: Create a UserList presentational component that receives users and loading as props.');
}

// Check that no class component
if (code.includes('extends Component') || code.includes('extends React.Component')) {
  throw new Error('Test failed: Refactor from class component to functional components with hooks.');
}

// Success!
console.log('✓ All tests passed! Successfully refactored to modern React patterns.');
console.log('✓ Logic is separated from presentation using custom hooks.');
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer="I would convert the class component to a functional component and move all the code into the function body, using useState and useEffect for the state and lifecycle methods."
  seniorAnswer="I would extract the stateful logic into a reusable custom hook called `useUsers` that encapsulates the data fetching with useState and useEffect. Then, I'd create a pure presentational component `UserList` that only handles rendering. Finally, a `UserPage` component would compose these together, achieving proper separation of concerns and making both the logic and UI independently reusable and testable.">

The original code uses the **Container and Presentational Component Pattern**. The `UserListContainer` is a "smart" container that knows how to fetch data and manage state. It then renders the UI directly. The problem is that the UI (the `ul` and `li` elements) is not reusable without the data-fetching logic.

With the introduction of React Hooks, this pattern has evolved. We can achieve a much cleaner separation of concerns by extracting all the stateful logic into a **custom Hook**.

1. **`useUsers` Hook**: This hook is our new "container." It handles the `useState` for `users` and `loading`, and the `useEffect` for fetching the data. It returns the stateful values. This logic is now reusable by any component.
2. **`UserList` Component**: This becomes our "dumb" presentational component. It receives `users` and `loading` as props and does nothing but render the UI. It's now easy to test and reuse.
3. **`UserPage` Component**: This component brings the two together. It calls the `useUsers` hook to get the data and then passes that data to the `UserList` component to be rendered.

This approach is more flexible, composable, and aligns with modern React best practices.

Here is the corrected implementation:

```jsx


const fetchUsers = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' },
        { id: 3, name: 'Charlie' },
      ]);
    }, 1000);
  });
};

// 1. The custom Hook for logic
const useUsers = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsers().then(fetchedUsers => {
      setUsers(fetchedUsers);
      setLoading(false);
    });
  }, []); // Runs once on mount

  return { users, loading };
};

// 2. The dumb presentational component
const UserList = ({ users, loading }) => {
  if (loading) {
    return <div>Loading users...</div>;
  }

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};

// 3. The final component bringing them together
export default function UserPage() {
  const { users, loading } = useUsers();
  return <UserList users={users} loading={loading} />;
}
```

</JuniorVsSenior>

---

### Question 11: Why does this component log the old state value after calling a useState setter?

**Type:** Conceptual | **Category:** React Fundamentals

<JuniorVsSenior
  juniorAnswer="The console.log should show the new value since we just updated it with setCount."
  seniorAnswer="State updates in React are asynchronous. When you call setCount, React schedules an update—it doesn't happen immediately. The count variable still holds the value from the current render, so the console.log shows the old value.">

To log the new value, you need to use `useEffect` with the state variable in its dependency array. This runs *after* the re-render when the new value is available.

**Key insight:** The functional update form `setCount(c => c + 1)` prevents bugs from batched updates, but doesn't solve this logging issue—you still need `useEffect` to react to state changes.
</JuniorVsSenior>

<SmartCodeRunner mode="run" title="Counter Component - Fix the Console Log">
  <Fragment slot="code">
```jsx


export default function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // Increment the state
    setCount(count + 1);

    // THE BUG: Try to log the new state value immediately
    console.log(`The new count is: ${count}`);
  };

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={handleClick}>Increment</button>
      <p>Check the console after clicking the button.</p>
    </div>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the fix
const code = document.querySelector('[data-editor]').textContent;

// Check that useEffect is imported and used
if (!code.includes('useEffect')) {
  throw new Error('Test failed: useEffect hook not imported or used. You need useEffect to log the state after it updates.');
}

// Check that count is in the dependency array
if (!code.includes('[count]')) {
  throw new Error('Test failed: count must be in the useEffect dependency array to track state changes.');
}

// Check that console.log is inside useEffect
const useEffectMatch = code.match(/useEffect\s*\(\s*\(\s*\)\s*=>\s*{[\s\S]*?console\.log/);
if (!useEffectMatch) {
  throw new Error('Test failed: console.log should be moved inside useEffect to log the updated state.');
}

// Success!
console.log('✓ All tests passed! The component now correctly logs the updated state value.');
console.log('✓ useEffect will run after each state update, giving you access to the new count.');
```
  </Fragment>
</SmartCodeRunner>

## The Solution

```jsx


export default function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  // FIX: This effect runs *after* the re-render with the new count
  useEffect(() => {
    if (count > 0) {
      console.log(`The new count is: ${count}`);
    }
  }, [count]); // The dependency array is key!

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

---

### Question 12: A child component is failing to update the parent's state. Why is this happening and how do you fix it?

**Type:** Debugging | **Category:** Props & State

A junior developer has built a `UserProfile` component that displays a username. They've also created a `ChangeUsername` child component that should update the username in the parent when a button is clicked.

However, clicking the button does nothing. The parent's state isn't updating.

Your task is to identify the bug that breaks React's one-way data flow and fix the code so that clicking the button correctly updates the username displayed in the `UserProfile` component.

<SmartCodeRunner mode="run" title="UserProfile - Fix Parent State Update">
  <Fragment slot="code">
```jsx


// Child component that tries to modify the prop directly
function ChangeUsername({ username }) {
  const changeName = () => {
    // This is the bug! Props are read-only.
    // This will not update the parent's state.
    username = 'Bob';
    console.log(`Username in child changed to: ${username}`);
  };

  return (
    <button onClick={changeName}>
      Change Username to Bob
    </button>
  );
}

// Parent component
export default function UserProfile() {
  const [username, setUsername] = useState('Alice');

  return (
    <div>
      <h1>Welcome, {username}!</h1>
      <ChangeUsername username={username} />
    </div>
  );
}
```
  </Fragment>

  <Fragment slot="test-cases">
```javascript
// Validate the fix
const code = document.querySelector('[data-editor]').textContent;

// Check that child receives a callback function prop
if (!code.includes('onUsernameChange') && !code.includes('onChange') && !code.includes('onUpdate')) {
  throw new Error('Test failed: Pass a callback function as a prop (e.g., onUsernameChange) to the child component.');
}

// Check that parent defines a handler function
if (!code.includes('handleUsernameChange') && !code.includes('handleChange') && !code.includes('handleUpdate')) {
  throw new Error('Test failed: Create a handler function in the parent that calls setUsername.');
}

// Check that setUsername is called
if (!code.includes('setUsername')) {
  throw new Error('Test failed: The parent should use setUsername to update the state.');
}

// Success!
console.log('✓ All tests passed! Child can now update parent state via callback.');
console.log('✓ This follows React\'s one-way data flow pattern correctly.');
```
  </Fragment>
</SmartCodeRunner>


The bug occurs because the child component tries to directly mutate the `username` prop. In React, this is an anti-pattern. Data flows in one direction: from parent to child. Props are read-only and cannot be modified by the child component.

The `console.log` shows the local `username` variable inside the `changeName` function being reassigned, but this does not affect the `username` state in the `UserProfile` parent component. React has no knowledge of this mutation, so it does not trigger a re-render.

Here is the corrected implementation:

```jsx


// Child component now receives a function to call
function ChangeUsername({ onUsernameChange }) {
  return (
    <button onClick={() => onUsernameChange('Bob')}>
      Change Username to Bob
    </button>
  );
}

// Parent component
export default function UserProfile() {
  const [username, setUsername] = useState('Alice');

  // The parent manages the state and the logic to update it
  const handleUsernameChange = (newName) => {
    setUsername(newName);
    console.log(`Username updated to: ${newName}`);
  };

  return (
    <div>
      <h1>Welcome, {username}!</h1>
      <ChangeUsername onUsernameChange={handleUsernameChange} />
    </div>
  );
}
```

---

### Question 13: Clarify the core React concepts: Element, Component, and Node.

**Type:** Conceptual | **Category:** React Fundamentals

<JuniorVsSenior
  juniorAnswer="They're all basically the same thing—they're all parts of React that make up the UI."
  seniorAnswer="They're distinct concepts with specific meanings:">

**React Element**
A plain JavaScript object that describes what you want to see on the screen. It's a lightweight, immutable blueprint—not the actual DOM node.

```jsx
// A React Element describing a div
const divElement = <div>Hello!</div>;

// What JSX compiles to:
// React.createElement('div', null, 'Hello!')
```

**React Component**
A function or class that takes props and returns a React Element. Components are the reusable building blocks you write.

```jsx
// A Function Component - returns a React Element
function MyComponent({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

**React Node**
The broadest term—anything React can render:
- React Elements (`<div />`, `<MyComponent />`)
- Strings (`"Hello"`)
- Numbers (`123`)
- Booleans, `null`, `undefined` (ignored but valid)
- Arrays of React Nodes

```jsx
// All of these are valid React Nodes
<ParentComponent>
  <div>Element node</div>
  {"String node"}
  {123}
  {[<p key="1">Array item</p>]}
</ParentComponent>
```

**The Relationship:**
- You **write** a React **Component**
- A Component **returns** a React **Element**
- An **Element** is a description of what to render
- A **Node** is anything React can render
</JuniorVsSenior>

---

### Question 14: Your app's state management is a mess. How would Flux architecture help?

**Type:** Architecture | **Category:** State Management

<JuniorVsSenior
  juniorAnswer="Just use Redux or Context—they fix state management problems."
  seniorAnswer="The problem isn't the tool, it's the data flow. Flux enforces unidirectional data flow, which makes state predictable and debuggable:">

**The Flux Flow: Action → Dispatcher → Store → View**

Unlike traditional MVC where data flows in multiple directions, Flux ensures data always flows in one direction.

**1. Actions**
Plain objects describing *what happened* (`ADD_TO_CART`, `USER_LOGIN`). Centralizes all possible events.

**2. Dispatcher**
Central hub that receives Actions and dispatches to all Stores. Processes one at a time—no race conditions.

**3. Stores**
Hold state and business logic. Update in response to Actions, emit "change" events. Single source of truth for their domain.

**4. Views (React Components)**
Listen for Store changes, re-render when state updates. Can trigger new Actions on user interaction.

**Why this solves the "spaghetti" problem:**
- **Predictable state:** You know the exact path every update takes
- **Easier debugging:** Trace the sequence of Actions that led to any state
- **Clear separation:** Each part has a distinct responsibility
- **Scalability:** Adding features means adding Actions/Stores, but the flow stays consistent

This pattern directly influenced Redux, MobX, and other modern state management solutions.
</JuniorVsSenior>

---

### Question 15: Your UI freezes during large updates. How does React Fiber help?

**Type:** Conceptual | **Category:** React Internals

<JuniorVsSenior
  juniorAnswer="Fiber is some internal React thing that makes it faster."
  seniorAnswer="Fiber is a complete rewrite of React's reconciliation algorithm that transforms rendering from a blocking, synchronous process into an interruptible, prioritized one:">

**The Problem Before Fiber**

React's old reconciliation was synchronous and uninterruptible. A 500ms update = 500ms UI freeze. No user input, no animations, nothing.

**How Fiber Solves It**

**1. Interruptible Rendering**
Fiber breaks rendering into small units of work. It can pause, yield control to the browser (for user input, repaints), then resume. No more blocking the main thread.

**2. Prioritization (Time Slicing)**
Not all updates are equal. Fiber assigns priorities:
- User interactions (typing, clicking) → **High priority**
- Background data fetches → **Low priority**

If a high-priority update arrives during low-priority work, React pauses the low-priority work and handles the urgent update first.

**3. Concurrent Rendering**
Fiber enables working on multiple tasks simultaneously or interleaving them. This is the foundation for Suspense and Concurrent Mode.

**Practical Benefits:**
- **Smoother UX:** Fewer freezes during heavy computations
- **Better interactions:** Typing and clicking feel immediate
- **Better animations:** Animation frames render on time
- **Foundation for features:** Suspense, Concurrent Mode, Transitions
</JuniorVsSenior>

---

### Question 16: This list re-renders inefficiently. How does React's reconciliation algorithm explain why?

**Type:** Debugging | **Category:** Performance

## The Scenario

A developer has a simple `ItemList` component that renders a list of items. They've noticed that when they add a new item to the *beginning* of the list, the entire list seems to re-render, even though most of the existing items haven't changed. This is causing a performance bottleneck in their application, especially with very long lists.

Each list item logs a "Rendering Item..." message when it renders.

## The Challenge

You've been given the `ItemList` component.

1.  Explain how React's **reconciliation algorithm** works and why this specific scenario (adding an item to the beginning of a list using array indices as keys) causes an inefficient re-render of all items.
2.  Fix the component to ensure that only the newly added item (and any truly changed items) re-renders, optimizing the list's performance.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// A component that logs when it renders
function ListItem({ item }) {
  console.log(`Rendering Item: ${item.id} - ${item.text}`);
  return <li>{item.text}</li>;
}

export default function ItemList() {
  const [items, setItems] = useState([
    { id: 1, text: 'Item A' },
    { id: 2, text: 'Item B' },
    { id: 3, text: 'Item C' },
  ]);

  const addItemToBeginning = () => {
    const newItem = { id: Date.now(), text: `New Item ${items.length + 1}` };
    setItems(prevItems => [newItem, ...prevItems]);
  };

  return (
    <div>
      <button onClick={addItemToBeginning}>Add Item to Beginning</button>
      <ul>
        {items.map((item, index) => (
          // THE BUG: Using array index as key.
          // This causes inefficient re-renders when items are reordered.
          <ListItem key={index} item={item} />
        ))}
      </ul>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code. You will see "Rendering Item: 1 - Item A", etc.
// 2. Click the "Add Item to Beginning" button.
// 3. With the bug, you will see "Rendering Item..." logged for *all* items,
//    including the existing ones, even though their content hasn't changed.
//
// After your fix:
// 1. When you click "Add Item to Beginning", only the *new* item should log
//    "Rendering Item...". The existing items should not re-render.

console.log("Goal: Prevent unnecessary re-renders of existing list items when a new item is added to the beginning.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: The Importance of Stable Keys in Reconciliation

 (
  <ListItem key={index} item={item} />
))}`,
    explanation: "When using array indices as keys, adding an item to the beginning causes all keys to shift. React thinks every item is new because the key for each position changed, forcing a complete re-render of all items."
  }}
  rightAnswer={{
    title: "Using Stable, Unique IDs",
    code: `{items.map((item) => (
  <ListItem key={item.id} item={item} />
))}`,
    explanation: "Using stable, unique IDs (like item.id) ensures that each item's key remains constant regardless of position. React can correctly identify that existing items are just reordered, only rendering the new item."
  }}
/>

### How React's Reconciliation Algorithm Works

When a component's state or props change, React builds a new tree of React elements (the Virtual DOM). It then compares this new tree with the previous one to figure out what changes need to be made to the actual DOM. This comparison process is called **diffing**.

When diffing lists, React uses `key` props to identify which items have changed, are added, or are removed.
*   If an item's `key` remains the same, React assumes it's the same component instance and tries to update it.
*   If an item's `key` changes, React treats it as a new component instance and unmounts the old one, then mounts the new one.

### The Bug: Using Array Index as `key`

In the buggy code, `index` is used as the `key` prop (`<ListItem key={index} item={item} />`). When a new item is added to the *beginning* of the `items` array:
*   The new item gets `key=0`.
*   The original item that had `key=0` now has `key=1`.
*   The original item that had `key=1` now has `key=2`, and so on.

React sees that the `key` for *every existing item* has changed. It doesn't realize that the items themselves are the same, just shifted. Instead, it thinks that the item with `key=0` is now a completely different item, and so on for the entire list. This forces React to re-render (and potentially re-mount) every single `ListItem` component, leading to inefficient updates.

### The Fix: Using a Stable, Unique ID as `key`

The solution is to use a **stable, unique identifier** for each item as its `key` prop. This ID should be unique among siblings and should not change across re-renders. Often, this comes from your data (e.g., a database ID).

```jsx


function ListItem({ item }) {
  console.log(`Rendering Item: ${item.id} - ${item.text}`);
  return <li>{item.text}</li>;
}

export default function ItemList() {
  const [items, setItems] = useState([
    { id: 1, text: 'Item A' },
    { id: 2, text: 'Item B' },
    { id: 3, text: 'Item C' },
  ]);

  const addItemToBeginning = () => {
    const newItem = { id: Date.now(), text: `New Item ${items.length + 1}` };
    setItems(prevItems => [newItem, ...prevItems]);
  };

  return (
    <div>
      <button onClick={addItemToBeginning}>Add Item to Beginning</button>
      <ul>
        {items.map((item) => (
          // FIX: Use a stable, unique ID from the item data as the key.
          <ListItem key={item.id} item={item} />
        ))}
      </ul>
    </div>
  );
}
```
Now, when a new item is added to the beginning, it gets a new unique `id` (e.g., `Date.now()`). The existing items retain their original `id`s. React's reconciliation algorithm can correctly identify that the existing items are the same instances, just reordered, and only the new item needs to be rendered.

---

### Question 17: This component makes duplicate API calls in development. How does Strict Mode help identify the bug?

**Type:** Debugging | **Category:** Debugging & Tools

## The Scenario

A developer is working on a component that sets up a `setInterval` to log a message every second. They've noticed that in development mode, the message appears twice as often as expected (every 0.5 seconds instead of every 1 second). This behavior is confusing, and they suspect something is wrong with their `useEffect` setup.

They've also noticed that this only happens in development mode, and not in production.

## The Challenge

You've been given the `Timer` component, which is wrapped in `React.StrictMode`.

1.  Explain why the `setInterval` is running twice as often as expected in development mode.
2.  How does `React.StrictMode` help identify this type of bug?
3.  Fix the `useEffect` to ensure the `setInterval` runs at the correct frequency (once per second), even when `React.StrictMode` is active.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


function Timer() {
  useEffect(() => {
    // THE BUG: This effect sets up a setInterval but lacks a cleanup function.
    // In Strict Mode, effects are double-invoked, leading to duplicate intervals.
    const intervalId = setInterval(() => {
      console.log('Timer tick!');
    }, 1000);

    // Missing cleanup function!
  }, []);

  return <div>Check the console for timer ticks.</div>;
}

export default function App() {
  return (
    <React.StrictMode>
      <Timer />
    </React.StrictMode>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. With the bug, "Timer tick!" will appear
//    approximately every 0.5 seconds (twice as fast as intended).
//
// After your fix:
// 1. "Timer tick!" should appear exactly once per second.

console.log("Goal: Ensure the timer ticks once per second, even in Strict Mode.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: Strict Mode's Double Invocation and Effect Cleanup

 {
  const intervalId = setInterval(() => {
    console.log('Timer tick!');
  }, 1000);
  // Missing cleanup function!
}, []);`,
    explanation: "Without a cleanup function, when Strict Mode double-invokes the effect, two intervals are created and both continue running. This causes the timer to tick twice as fast (every 0.5 seconds instead of 1 second)."
  }}
  rightAnswer={{
    title: "With Cleanup Function",
    code: `useEffect(() => {
  const intervalId = setInterval(() => {
    console.log('Timer tick!');
  }, 1000);

  return () => {
    clearInterval(intervalId);
    console.log('Timer cleanup!');
  };
}, []);`,
    explanation: "The cleanup function ensures that when the effect is re-invoked, the previous interval is cleared. In Strict Mode, the cleanup runs after the first setup, preventing duplicate intervals."
  }}
/>

### Why Strict Mode Does This

The problem you're observing is a direct consequence of **React Strict Mode**. In development mode, `React.StrictMode` intentionally **double-invokes** certain lifecycle methods and effects (including `useEffect`'s setup and cleanup functions).

**Why does Strict Mode do this?**
Strict Mode's goal is to help developers identify potential problems and ensure their components are resilient to future changes in React. By double-invoking effects, it helps uncover:
1.  **Missing cleanup functions**: If an effect sets up a subscription, timer, or event listener but doesn't clean it up, double-invocation will cause two instances to run simultaneously, as seen here.
2.  **Non-idempotent side effects**: If an effect performs an action that has observable side effects (like fetching data or modifying global state) and doesn't produce the same result when run twice, Strict Mode will highlight this.

In our `Timer` component, the `useEffect` sets up a `setInterval` but **lacks a cleanup function**. When Strict Mode runs the effect twice:
1.  First invocation: `setInterval` is called, `intervalId_1` is created.
2.  Second invocation (simulated unmount/remount): `setInterval` is called again, `intervalId_2` is created.
Since there's no cleanup, both `intervalId_1` and `intervalId_2` continue to run, leading to two "Timer tick!" logs per second.

### The Fix: Implementing a Cleanup Function

The solution is to provide a **cleanup function** for the `useEffect` hook. This function is returned by the effect and is executed:
*   Before the effect re-runs (if dependencies change).
*   When the component unmounts.
*   Crucially, in Strict Mode, it runs *after* the first invocation and *before* the second invocation, effectively "resetting" the effect.

```jsx


function Timer() {
  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log('Timer tick!');
    }, 1000);

    // FIX: Return a cleanup function to clear the interval.
    // Strict Mode will call this cleanup after the first setup,
    // then set up the effect again.
    return () => {
      clearInterval(intervalId);
      console.log('Timer cleanup!'); // You'll see this log in Strict Mode
    };
  }, []); // Empty dependency array to run once on mount/unmount cycle

  return <div>Check the console for timer ticks.</div>;
}

export default function App() {
  return (
    <React.StrictMode>
      <Timer />
    </React.StrictMode>
  );
}
```
With the cleanup function, when Strict Mode double-invokes the effect:
1.  Effect setup runs, `intervalId_1` is created.
2.  Cleanup runs, `intervalId_1` is cleared.
3.  Effect setup runs again, `intervalId_2` is created.
Now, only `intervalId_2` is active, and the timer ticks at the correct frequency.


---

### Quick Check

**What is the primary purpose of `React.StrictMode` in development mode?**

   A. To enforce type checking for all props and state.
   B. To prevent components from re-rendering unnecessarily.
-> C. **To highlight potential problems in an application by activating additional checks and warnings.**
   D. To automatically optimize the production build size.

<details>
<summary>See Answer</summary>

React.StrictMode is a development tool that activates additional checks and warnings to help identify potential problems in your application. It double-invokes effects and lifecycle methods to surface issues like missing cleanup functions.

</details>

---

### Question 18: Your team is evaluating React. How would you explain its core benefits?

**Type:** Conceptual | **Category:** React Fundamentals

<JuniorVsSenior
  juniorAnswer="React is popular and has lots of libraries. It's what everyone uses."
  seniorAnswer="React solves three fundamental problems that make building complex UIs difficult:">

**1. Declarative UI**

You describe *what* the UI should look like for a given state, not *how* to change it. React handles the DOM manipulation.

- **For developers:** Easier to reason about—focus on the end state, not the transitions
- **For users:** More stable UIs with fewer visual glitches

**2. Component-Based Architecture**

Build UIs from small, isolated, reusable pieces. Each component manages its own state and logic.

- **For developers:** Modularity, maintainability, parallel team development
- **For users:** Consistent experience, faster feature delivery

**3. Virtual DOM and Efficient Updates**

React maintains an in-memory representation of the DOM. When state changes, it calculates the minimal set of changes needed and applies only those.

- **For developers:** Performance optimization is abstracted away
- **For users:** Faster, smoother UIs—especially in complex apps with frequent updates

These aren't just buzzwords—they translate into faster development, fewer bugs, and better user experiences.
</JuniorVsSenior>

---

### Question 19: This component has accessibility ID issues. Fix it with `useId`.

**Type:** Practical | **Category:** Hooks & Patterns

A developer is building a custom `Checkbox` component. For accessibility, they need to associate the checkbox's `label` with the input using the `htmlFor` attribute on the label and the `id` attribute on the input.

They've tried to generate a unique ID using `Math.random()` to ensure each instance of the checkbox has a distinct ID. However, in a server-rendered application (or even with multiple instances of the component on the client), this approach leads to:
*   **Hydration mismatches**: `Math.random()` generates different numbers on the server and client, causing React to complain.
*   **Duplicate IDs**: If `Math.random()` generates the same number twice, or if multiple instances of the component are rendered, you can end up with non-unique IDs, which breaks accessibility.

You've been given the `CustomCheckbox` component.

1.  Explain why using `Math.random()` (or similar non-deterministic methods) is problematic for generating IDs in server-rendered React applications.
2.  Fix the component using the `useId` hook to generate stable, unique IDs that are consistent across server and client, ensuring proper accessibility.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Custom Checkbox component
function CustomCheckbox({ label, checked, onChange }) {
  // THE BUG: Using Math.random() for ID generation.
  // This can cause hydration mismatches in SSR and duplicate IDs.
  const uniqueId = `checkbox-${Math.random().toString(36).substr(2, 9)}`;

  return (
    <div>
      <input
        type="checkbox"
        id={uniqueId}
        checked={checked}
        onChange={onChange}
      />
      <label htmlFor={uniqueId}>{label}</label>
    </div>
  );
}

export default function App() {
  const [isChecked1, setIsChecked1] = useState(false);
  const [isChecked2, setIsChecked2] = useState(true);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Accessibility ID Issues</h1>
      <CustomCheckbox
        label="Enable Feature A"
        checked={isChecked1}
        onChange={() => setIsChecked1(!isChecked1)}
      />
      <CustomCheckbox
        label="Enable Feature B"
        checked={isChecked2}
        onChange={() => setIsChecked2(!isChecked2)}
      />
      <p>Check the console for potential hydration warnings or inspect elements for duplicate IDs.</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based and inspection test.
// 1. Run the code.
// 2. Observe the console. With the bug, in a real SSR environment,
//    you would likely see hydration mismatch warnings.
// 3. Inspect the DOM elements. You might find duplicate IDs if Math.random()
//    generates the same value (unlikely but possible), or if the component
//    is rendered multiple times in a complex scenario.
//
// After your fix:
// 1. There should be no hydration warnings in the console.
// 2. Each checkbox input and its corresponding label should have a unique,
//    correctly associated ID.

console.log("Goal: Generate stable, unique IDs for accessibility attributes without hydration issues.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer="I would use a counter or generate IDs based on the label text to make them unique."
  seniorAnswer="I would use the `useId` hook introduced in React 18. It generates unique IDs that are stable across server and client renders, preventing hydration mismatches. The IDs are guaranteed to be unique within the application, making it perfect for accessibility attributes like `id` and `htmlFor`.">

The issue with using `Math.random()` (or `Date.now()`) for generating IDs in React, especially in server-rendered (SSR) applications, is that these functions are **non-deterministic**.

*   **Hydration Mismatch**: In an SSR environment, the server renders the HTML, including the generated IDs. When the client-side React code then "hydrates" this HTML, it re-renders the components. If `Math.random()` is used, it will likely generate a *different* random number on the client than it did on the server. React sees this discrepancy in the generated IDs, leading to a **hydration mismatch warning** (e.g., "Text content did not match server-rendered HTML").
*   **Duplicate IDs**: While `Math.random()` is unlikely to produce duplicates in a single render, it's not guaranteed. More importantly, if you have multiple instances of the same component, or if the component re-renders, you might accidentally generate the same ID, which breaks the fundamental rule of HTML IDs (they must be unique within the document).

The `useId` hook, introduced in React 18, is specifically designed to solve this problem. It generates a unique ID that is:
*   **Stable**: The same ID is generated on both the server and the client for a given component instance, preventing hydration mismatches.
*   **Unique**: It guarantees uniqueness across the entire application, even for multiple instances of the same component.

Here is the corrected implementation:

```jsx


function CustomCheckbox({ label, checked, onChange }) {
  // FIX: Use useId to generate a stable and unique ID.
  const uniqueId = useId();

  return (
    <div>
      <input
        type="checkbox"
        id={uniqueId}
        checked={checked}
        onChange={onChange}
      />
      <label htmlFor={uniqueId}>{label}</label>
    </div>
  );
}

export default function App() {
  const [isChecked1, setIsChecked1] = useState(false);
  const [isChecked2, setIsChecked2] = useState(true);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Accessibility ID Issues</h1>
      <CustomCheckbox
        label="Enable Feature A"
        checked={isChecked1}
        onChange={() => setIsChecked1(!isChecked1)}
      />
      <CustomCheckbox
        label="Enable Feature B"
        checked={isChecked2}
        onChange={() => setIsChecked2(!isChecked2)}
      />
      <p>Check the console for potential hydration warnings or inspect elements for duplicate IDs.</p>
    </div>
  );
}
```
By replacing `Math.random()` with `useId()`, we ensure that each checkbox instance has a unique and stable ID, resolving both hydration mismatches in SSR and potential duplicate ID issues, thereby improving accessibility.

</JuniorVsSenior>


---

### Quick Check

**What is the primary purpose of the `useId` hook in React?**

   A. To generate random numbers for cryptographic purposes.
-> B. **To create unique, stable IDs for accessibility attributes that are consistent across server and client.**
   C. To provide a unique key for list items when rendering dynamic lists.
   D. To identify the current user's session ID.

<details>
<summary>See Answer</summary>

The `useId` hook is specifically designed to generate unique, stable IDs that remain consistent between server-side rendering and client-side hydration. This makes it perfect for accessibility attributes like `id` and `htmlFor`, preventing hydration mismatches and ensuring proper HTML semantics.

</details>

---

### Question 20: This modal is clipped by its parent. Fix it using a React Portal.

**Type:** Practical | **Category:** Component Patterns

A developer has implemented a `Modal` component. It's designed to pop up and display important information. However, when it's used inside a `Card` component that has `overflow: hidden` and a specific `z-index` applied, the modal gets clipped or appears behind other elements on the page. This makes the modal unusable.

The problem is that the modal is rendered within the DOM hierarchy of its parent (`Card`), inheriting its parent's styling properties that prevent it from displaying correctly as an overlay.

You've been given the code for the `App` and `Modal` components. Your task is to fix this issue by using a **React Portal** to render the modal outside the parent's DOM hierarchy. This will ensure it appears correctly as an overlay, fully visible and on top of all other content.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// The Modal component - currently renders within its parent's DOM hierarchy
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  // THE BUG: This modal renders directly inside its parent,
  // making it susceptible to parent's CSS properties like overflow and z-index.
  return (
    <div style={{
      position: 'fixed',
      top: '50%',
      left: '50%',
      transform: 'translate(-50%, -50%)',
      backgroundColor: 'white',
      padding: '20px',
      borderRadius: '8px',
      boxShadow: '0 4px 8px rgba(0,0,0,0.2)',
      zIndex: 1000, // This z-index might not be enough if parent has higher
      maxWidth: '80vw',
      maxHeight: '80vh',
      overflow: 'auto',
    }}>
      {children}
      <button onClick={onClose} style={{ position: 'absolute', top: '10px', right: '10px' }}>
        X
      </button>
    </div>
  );
}

// A Card component with styling that will clip the modal
function Card({ children }) {
  return (
    <div style={{
      border: '1px solid #ccc',
      padding: '20px',
      width: '300px',
      height: '200px',
      margin: '50px auto',
      overflow: 'hidden', // This is the culprit!
      position: 'relative',
      zIndex: 1,
    }}>
      {children}
    </div>
  );
}

export default function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <h1>Main Application Content</h1>
      <Card>
        <h2>Content inside Card</h2>
        <p>This card has overflow: hidden.</p>
        <button onClick={() => setIsModalOpen(true)}>Open Clipped Modal</button>
        <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)}>
          <h3>Important Message</h3>
          <p>This modal content should be fully visible!</p>
          <p>But it's being clipped by the card.</p>
        </Modal>
      </Card>
      {/* We need a target DOM node for the portal. Add this to your index.html:
          <div id="modal-root"></div>
          For this runner, we'll simulate it.
      */}
      <div id="modal-root" style={{ position: 'absolute', top: 0, left: 0, width: '100%', height: '100%', pointerEvents: 'none' }}></div>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code.
// 2. Click the "Open Clipped Modal" button.
// 3. Observe that the modal content is cut off or not fully visible.
//
// After your fix:
// 1. The modal should appear fully on top of all content,
//    without any clipping, when the button is clicked.

console.log("Goal: Make the modal appear fully and correctly as an overlay.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer="I would change the Card's CSS to remove `overflow: hidden` or adjust the z-index values to make the modal appear on top."
  seniorAnswer="I would use `ReactDOM.createPortal` to render the modal into a separate DOM node outside the Card's hierarchy, typically directly under the body. This breaks the modal free from its parent's CSS constraints (like overflow and z-index context) while maintaining React's component hierarchy and event bubbling, making it a proper overlay solution.">

The problem arises because the `Modal` component is rendered directly within the `Card` component's DOM hierarchy. When a parent element has CSS properties like `overflow: hidden` or a specific `z-index` context, its children are constrained by those properties. This is why the modal gets clipped or appears behind other elements.

**React Portals** provide a way to render children into a DOM node that exists outside the DOM hierarchy of the parent component. This is perfect for modals, tooltips, and other overlays that need to "break out" of their parent's styling constraints.

To use a Portal:
1.  You need a target DOM node outside your main React application's root. This is typically an empty `div` element added directly to your `index.html` (e.g., `<div id="modal-root"></div>`).
2.  Inside your component (e.g., `Modal`), you use `ReactDOM.createPortal(child, domNode)` to render the modal's content into that external DOM node.

Here is the corrected implementation:

```jsx


// The Modal component - now uses a Portal
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  // Ensure the modal root element exists
  const modalRoot = document.getElementById('modal-root');
  if (!modalRoot) {
    // In a real app, you'd ensure this exists or create it dynamically
    console.error("Modal root element not found! Please add <div id='modal-root'></div> to your index.html");
    return null;
  }

  // FIX: Use ReactDOM.createPortal to render the modal content
  // into the 'modal-root' DOM node.
  return ReactDOM.createPortal(
    <div style={{
      position: 'fixed',
      top: '50%',
      left: '50%',
      transform: 'translate(-50%, -50%)',
      backgroundColor: 'white',
      padding: '20px',
      borderRadius: '8px',
      boxShadow: '0 4px 8px rgba(0,0,0,0.2)',
      zIndex: 1000,
      maxWidth: '80vw',
      maxHeight: '80vh',
      overflow: 'auto',
    }}>
      {children}
      <button onClick={onClose} style={{ position: 'absolute', top: '10px', right: '10px' }}>
        X
      </button>
    </div>,
    modalRoot // Render into the external DOM node
  );
}

// A Card component with styling that will clip the modal
function Card({ children }) {
  return (
    <div style={{
      border: '1px solid #ccc',
      padding: '20px',
      width: '300px',
      height: '200px',
      margin: '50px auto',
      overflow: 'hidden',
      position: 'relative',
      zIndex: 1,
    }}>
      {children}
    </div>
  );
}

export default function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  // Ensure the modal-root div exists in the DOM for the runner
  useEffect(() => {
    let modalRoot = document.getElementById('modal-root');
    if (!modalRoot) {
      modalRoot = document.createElement('div');
      modalRoot.id = 'modal-root';
      document.body.appendChild(modalRoot);
    }
    return () => {
      if (modalRoot && modalRoot.parentNode === document.body) {
        document.body.removeChild(modalRoot);
      }
    };
  }, []);


  return (
    <div>
      <h1>Main Application Content</h1>
      <Card>
        <h2>Content inside Card</h2>
        <p>This card has overflow: hidden.</p>
        <button onClick={() => setIsModalOpen(true)}>Open Modal</button>
        <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)}>
          <h3>Important Message</h3>
          <p>This modal content should be fully visible!</p>
        </Modal>
      </Card>
    </div>
  );
}
```
With the `Modal` component now using `ReactDOM.createPortal`, its content is rendered directly under the `body` element (or whatever `modalRoot` is), effectively "breaking out" of the `Card`'s `overflow: hidden` and `z-index` constraints. This ensures the modal always appears as a top-level overlay.

</JuniorVsSenior>

---

### Question 21: This component re-renders unnecessarily due to context object reference changes. How do you fix it?

**Type:** Debugging | **Category:** Performance

A developer has a `UserContext` that provides a `user` object containing `id` and `name`. They also have a `UserProfile` component that consumes this context to display the user's name.

They've noticed a performance issue: `UserProfile` re-renders every time its parent component re-renders, even if the `user` object's *content* (id and name) hasn't changed. This is causing unnecessary work on a complex profile page.

You've been given the code that demonstrates this issue.

1.  Explain why the `UserProfile` component re-renders when the parent component re-renders, even if the `user` data is logically the same.
2.  Optimize the `UserProvider` to prevent these unnecessary re-renders in `UserProfile`.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// User Context
const UserContext = createContext();

// Component consuming the UserContext
function UserProfile() {
  const { user } = useContext(UserContext);
  console.log('Re-rendering UserProfile...'); // This should not happen on parent re-render if user data is same
  return (
    <div>
      <h2>User Profile</h2>
      <p>ID: {user.id}</p>
      <p>Name: {user.name}</p>
    </div>
  );
}

// User Provider
function UserProvider({ children }) {
  const [userId, setUserId] = useState(1);
  const [userName, setUserName] = useState('Alice');

  // THE BUG: The user object is created inline on every render.
  // Its reference changes even if userId and userName are the same.
  const user = { id: userId, name: userName };

  // This context value object's reference changes on every render of UserProvider
  const contextValue = { user };

  return (
    <UserContext.Provider value={contextValue}>
      {children}
    </UserContext.Provider>
  );
}

export default function App() {
  const [count, setCount] = useState(0); // State to force App re-renders

  return (
    <UserProvider>
      <div style={{ padding: '20px' }}>
        <h1>Application</h1>
        <button onClick={() => setCount(c => c + 1)}>
          Force App Re-render (Count: {count})
        </button>
        <hr />
        <UserProfile />
      </div>
    </UserProvider>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. You will see "Re-rendering UserProfile..."
//    on initial render.
// 3. Click the "Force App Re-render" button.
// 4. With the bug, you will see "Re-rendering UserProfile..." logged again,
//    even though the user's ID and Name haven't changed.
//
// After your fix, clicking "Force App Re-render" should NOT cause
// "Re-rendering UserProfile..." to be logged.

console.log("Goal: Prevent UserProfile from re-rendering when user data is logically unchanged.");
```
  </Fragment>
</SmartCodeRunner>


The problem stems from how JavaScript handles objects and how React's `useContext` hook detects changes.

In the `UserProvider` component, the `user` object (`{ id: userId, name: userName }`) is created directly within the component's render function. Even if `userId` and `userName` remain the same, every time `UserProvider` re-renders (e.g., when its parent `App` re-renders due to `setCount`), a **new object reference** for `user` is created.

When this new `user` object is then placed into `contextValue` (which also becomes a new object reference), React sees that the `value` prop passed to `UserContext.Provider` has changed (because its reference is different). Consequently, `useContext` in `UserProfile` detects a change in the context value and triggers a re-render of `UserProfile`, even though the actual `id` and `name` properties within the `user` object are identical.

To prevent this unnecessary re-render, we need to ensure that the `contextValue` object's reference only changes when its actual dependencies (`userId` or `userName`) change. This can be achieved using the `useMemo` hook.

`useMemo` will memoize the `contextValue` object. It will only re-create the object (and thus change its reference) if any of its dependencies (specified in the dependency array) have changed.

Here is the corrected implementation:

```jsx


const UserContext = createContext();

function UserProfile() {
  const { user } = useContext(UserContext);
  console.log('Re-rendering UserProfile...');
  return (
    <div>
      <h2>User Profile</h2>
      <p>ID: {user.id}</p>
      <p>Name: {user.name}</p>
    </div>
  );
}

function UserProvider({ children }) {
  const [userId, setUserId] = useState(1);
  const [userName, setUserName] = useState('Alice');

  // FIX: Memoize the user object and the context value.
  // The user object will only be re-created if userId or userName change.
  const user = useMemo(() => ({ id: userId, name: userName }), [userId, userName]);

  // The contextValue object will only be re-created if the 'user' object's
  // reference changes (which is now controlled by the useMemo above).
  const contextValue = useMemo(() => ({ user }), [user]);

  return (
    <UserContext.Provider value={contextValue}>
      {children}
    </UserContext.Provider>
  );
}

export default function App() {
  const [count, setCount] = useState(0);

  return (
    <UserProvider>
      <div style={{ padding: '20px' }}>
        <h1>Application</h1>
        <button onClick={() => setCount(c => c + 1)}>
          Force App Re-render (Count: {count})
        </button>
        <hr />
        <UserProfile />
      </div>
    </UserProvider>
  );
}
```
With `useMemo`, the `user` object and `contextValue` object references will remain stable across `UserProvider` re-renders as long as `userId` and `userName` don't change. This prevents `UserProfile` from re-rendering unnecessarily.

---

### Question 22: This button should increment twice, but only increments once. How do you fix it?

**Type:** Debugging | **Category:** State Management

A developer has a `Counter` component with a "Double Increment" button. The intention is that when this button is clicked, the counter's value should increase by 2.

They've implemented this by calling `setCount(count + 1)` twice in the same event handler. However, they notice a bug: clicking the "Double Increment" button only increments the count by 1, not by 2.

You've been given the `Counter` component.

1.  Explain why calling `setCount(count + 1)` twice in a row only increments the count by 1.
2.  Fix the component to correctly increment the counter by 2 with a single button click.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


export default function Counter() {
  const [count, setCount] = useState(0);

  const handleDoubleIncrement = () => {
    // THE BUG: Calling setCount with a direct value multiple times
    // in the same event handler.
    setCount(count + 1); // React sees count as 0, schedules update to 1
    setCount(count + 1); // React still sees count as 0, schedules update to 1 again
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Count: {count}</h1>
      <button onClick={handleDoubleIncrement}>Double Increment</button>
      <p>Click the button and observe the count.</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. The count is 0.
// 2. Click the "Double Increment" button.
// 3. With the bug, the count will only increase to 1.
//
// After your fix:
// 1. Clicking the "Double Increment" button should correctly increase the count by 2.

console.log("Goal: Make the 'Double Increment' button increase the count by 2.");
```
  </Fragment>
</SmartCodeRunner>


> **Common Mistake:** Calling `setCount(count + 1)` twice uses the same stale count value from the current render closure. Both calls schedule updates to the same value, and React batches them, resulting in only one increment.

> **Senior Engineer Approach:** Use the functional update form `setCount(prevCount => prevCount + 1)`. Each call receives the most recent state value as an argument, ensuring sequential updates are applied correctly even when batched.

The bug occurs because React's state updates (triggered by `setCount`) are **asynchronous** and often **batched** for performance reasons.

When `handleDoubleIncrement` is called:
1.  `setCount(count + 1)`: At this point, `count` is `0`. React schedules an update to set `count` to `1`.
2.  `setCount(count + 1)`: Immediately after, this line is executed. Crucially, `count` in this line *still refers to the value from the current render scope*, which is `0`. So, React schedules *another* update to set `count` to `1`.

Because React batches these updates, it sees two requests to set `count` to `1`. The second request effectively overwrites the first, resulting in the count only increasing by 1.

To ensure that state updates are based on the *most recent* state, you should use the **functional update form** of `setCount` (or `setState` in class components). This form passes a function to `setCount`, and that function receives the `prevState` as its argument. React guarantees that `prevState` will always be the latest available state.

Here is the corrected implementation:

```jsx


export default function Counter() {
  const [count, setCount] = useState(0);

  const handleDoubleIncrement = () => {
    // FIX: Use the functional update form to ensure each update
    // is based on the previous state.
    setCount(prevCount => prevCount + 1); // Schedules update based on prevCount
    setCount(prevCount => prevCount + 1); // Schedules another update based on the result of the first
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Count: {count}</h1>
      <button onClick={handleDoubleIncrement}>Double Increment</button>
      <p>Click the button and observe the count.</p>
    </div>
  );
}
```
By using `setCount(prevCount => prevCount + 1)`, each call to `setCount` now correctly receives the result of the *previous* state update, ensuring that the counter increments by 2.

---

### Question 23: An expensive component is re-rendering unnecessarily. How would you fix this state management issue?

**Type:** Debugging | **Category:** Performance

A junior developer built a new feature using React Context to manage the application's global state. The state object contains everything: the current user's info, and the UI theme (light/dark mode).

They've noticed a performance problem. The `UserWelcome` component, which is expensive to render, re-renders every single time the theme is changed. This is making the UI feel sluggish.

You've been given the code that demonstrates this issue.

1.  Explain why the `UserWelcome` component re-renders every time the "Toggle Theme" button is clicked.
2.  Describe two different strategies to fix this performance issue so that `UserWelcome` only re-renders when the `user` data it cares about actually changes.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// The monolithic context holding all global state
const AppContext = createContext();

// A component that is "expensive" to render
function UserWelcome() {
  const { user } = useContext(AppContext);
  console.log('Re-rendering UserWelcome (expensive)...');
  return <h1>Welcome, {user.name}!</h1>;
}

// A component to change the theme
function ThemeToggler() {
  const { theme, setTheme } = useContext(AppContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </button>
  );
}

// The main App provider
export default function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  const contextValue = { user, setUser, theme, setTheme };

  return (
    <AppContext.Provider value={contextValue}>
      <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff', padding: '20px' }}>
        <ThemeToggler />
        <hr />
        <UserWelcome />
      </div>
    </AppContext.Provider>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Click the "Toggle Theme" button.
// 3. Observe the console. You will see "Re-rendering UserWelcome (expensive)..."
//    logged every time you click, even though the user data hasn't changed.
//
// After your fix, this log should NOT appear when you only toggle the theme.

console.log("Goal: Prevent the expensive component from re-rendering on unrelated state changes.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState, useContext, createContext } from 'react';

// The monolithic context holding all global state
const AppContext = createContext();

// A component that is "expensive" to render
function UserWelcome() {
  const { user } = useContext(AppContext);
  console.log('Re-rendering UserWelcome (expensive)...');
  return <h1>Welcome, {user.name}!</h1>;
}

// A component to change the theme
function ThemeToggler() {
  const { theme, setTheme } = useContext(AppContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </button>
  );
}

// The main App provider
export default function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  const contextValue = { user, setUser, theme, setTheme };

  return (
    <AppContext.Provider value={contextValue}>
      <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff', padding: '20px' }}>
        <ThemeToggler />
        <hr />
        <UserWelcome />
      </div>
    </AppContext.Provider>
  );
}`}
  seniorAnswer={`// Solution 1: Split the Contexts (Best Practice)

// Create two separate contexts
const ThemeContext = createContext();
const UserContext = createContext();

function UserWelcome() {
  // Subscribe ONLY to the UserContext
  const { user } = useContext(UserContext);
  console.log('Re-rendering UserWelcome (expensive)...');
  return <h1>Welcome, {user.name}!</h1>;
}

function ThemeToggler() {
  // Subscribe ONLY to the ThemeContext
  const { theme, setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </button>
  );
}

export default function App() {
  const [user, setUser] = useState({ name: 'Alice' });
  const [theme, setTheme] = useState('light');

  // Nest the providers
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff', padding: '20px' }}>
          <ThemeToggler />
          <hr />
          <UserWelcome />
        </div>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}`}
  juniorExplanation="The performance issue is a classic pitfall of using React Context. When a component consumes a context (with useContext), it subscribes to all changes in that context's value. In this code, we have a single, monolithic AppContext that holds both user and theme. The UserWelcome component only needs user, but because it's subscribed to the entire AppContext, React will re-render it whenever any value in the context provider's value object changes. Clicking 'Toggle Theme' creates a new contextValue object with a new theme, triggering a re-render in every component that consumes AppContext, including the expensive UserWelcome."
  seniorExplanation="The most robust solution is to separate unrelated state into different contexts. Components can then subscribe only to the state they care about. Now, when the theme changes, only the ThemeContext.Provider gets a new value, and only its consumers (ThemeToggler and the div) will re-render. UserWelcome is unaffected. A quicker, but often more brittle, solution is to use React.memo to wrap the expensive component. React.memo is a higher-order component that prevents a component from re-rendering if its props haven't changed. However, this doesn't work by itself because the component has no props. You'd need to combine memo with extracting the component and passing state down as props, which shows the complexity and why splitting contexts is often better. Splitting contexts is almost always the cleaner, more scalable solution."
  client:load
/>

---

### Question 24: This component flickers when resizing. Fix it using the correct effect hook.

**Type:** Practical | **Category:** Hooks & Patterns

## The Scenario

A developer is building a component that needs to display a `div` that is always a perfect square (its height should always equal its width). They've implemented logic to measure the `div`'s width and then set its height using state, all within a `useEffect` hook.

However, they notice a brief visual flicker: the `div` first appears with its default height (or an incorrect height), then quickly resizes to become a square. This flicker is undesirable and creates a jarring user experience.

## The Challenge

You've been given the `SquareBox` component.

1.  Explain the difference between `useEffect` and `useLayoutEffect` in terms of their execution timing.
2.  Fix the component to eliminate the visual flicker, ensuring the `div` always renders as a perfect square from the very first paint.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


export default function SquareBox() {
  const boxRef = useRef(null);
  const [height, setHeight] = useState(0);

  useEffect(() => {
    // THE BUG: useEffect runs AFTER the browser has painted.
    // This causes a flicker as the div is first painted with default height,
    // then measured, then re-rendered with the correct height.
    if (boxRef.current) {
      const width = boxRef.current.offsetWidth;
      console.log(`useEffect: Measured width ${width}, setting height to ${width}`);
      setHeight(width);
    }
  }, [height]); // Re-run when height changes (to re-measure if width changes)

  return (
    <div style={{ padding: '20px' }}>
      <h1>Square Box</h1>
      <div
        ref={boxRef}
        style={{
          width: '200px', // Fixed width for demonstration
          height: height || 'auto', // Initial height might be 'auto' or 0
          backgroundColor: 'lightblue',
          border: '2px solid blue',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: '20px',
          fontWeight: 'bold',
        }}
      >
        I am a square!
      </div>
      <p>Resize your browser window to see the effect (if width changes).</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code.
// 2. Observe the "Square Box" div. With the bug, you might see a brief flicker
//    where it's not square, then it snaps into shape.
//    (This flicker might be subtle depending on browser speed).
//
// After your fix:
// 1. The "Square Box" div should always appear as a perfect square from the
//    very first render, with no visible flicker.

console.log("Goal: Eliminate the visual flicker when the component renders and resizes.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: `useEffect` vs. `useLayoutEffect` Timing

 {
  if (boxRef.current) {
    const width = boxRef.current.offsetWidth;
    setHeight(width);
  }
}, [height]);`,
    explanation: "useEffect runs asynchronously AFTER the browser has painted. The sequence is: React renders → Browser paints (shows incorrect height) → useEffect runs → setHeight triggers re-render → Browser paints again. This causes visible flicker."
  }}
  rightAnswer={{
    title: "Using useLayoutEffect (No Flicker)",
    code: `useLayoutEffect(() => {
  if (boxRef.current) {
    const width = boxRef.current.offsetWidth;
    if (height !== width) {
      setHeight(width);
    }
  }
}, [height]);`,
    explanation: "useLayoutEffect runs synchronously BEFORE the browser paints. The sequence is: React renders → useLayoutEffect runs → setHeight triggers re-render → Browser paints (shows correct height). No flicker because the browser only paints once with the correct dimensions."
  }}
/>

### Understanding the Timing Difference

The visual flicker occurs because of the execution timing of `useEffect`.

*   **`useEffect`**: Runs **asynchronously** after React has performed all DOM mutations *and* after the browser has had a chance to paint the screen. This means:
    1.  React renders the `SquareBox` with its initial `height` (which is `0` or `'auto'`).
    2.  The browser paints this initial, potentially incorrect, state.
    3.  `useEffect` runs, measures the `width`, and calls `setHeight(width)`.
    4.  `setHeight` triggers another render.
    5.  React renders `SquareBox` with the correct `height`.
    6.  The browser paints the corrected state.
    This sequence causes the flicker.

*   **`useLayoutEffect`**: Runs **synchronously** after React has performed all DOM mutations *but before* the browser has painted the screen.

### The Fix: Using `useLayoutEffect`

For tasks that involve reading from the DOM (like measuring layout) and then immediately making synchronous DOM updates (like setting a style based on that measurement) that need to be visible *before* the user sees the next paint, `useLayoutEffect` is the correct hook to use. It ensures that these operations happen before the browser has a chance to paint the intermediate, incorrect state.

Here is the corrected implementation:

```jsx


export default function SquareBox() {
  const boxRef = useRef(null);
  const [height, setHeight] = useState(0);

  // FIX: Use useLayoutEffect for DOM measurements that need to happen
  // before the browser paints, to prevent flicker.
  useLayoutEffect(() => {
    if (boxRef.current) {
      const width = boxRef.current.offsetWidth;
      console.log(`useLayoutEffect: Measured width ${width}, setting height to ${width}`);
      // Only update if necessary to avoid infinite loops
      if (height !== width) {
        setHeight(width);
      }
    }
  }, [height]); // Re-run when height changes (to re-measure if width changes)

  return (
    <div style={{ padding: '20px' }}>
      <h1>Square Box</h1>
      <div
        ref={boxRef}
        style={{
          width: '200px',
          height: height || 'auto',
          backgroundColor: 'lightblue',
          border: '2px solid blue',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: '20px',
          fontWeight: 'bold',
        }}
      >
        I am a square!
      </div>
      <p>Resize your browser window to see the effect (if width changes).</p>
    </div>
  );
}
```
By using `useLayoutEffect`, the `div`'s height is measured and updated *before* the browser paints the screen, eliminating the visual flicker.


---

### Quick Check

**When does `useLayoutEffect` execute in the React component lifecycle?**

   A. Asynchronously, after the browser has painted the screen.
-> B. **Synchronously, after all DOM mutations but before the browser has painted the screen.**
   C. Only once, when the component first mounts.
   D. Before any DOM mutations are made by React.

<details>
<summary>See Answer</summary>

useLayoutEffect runs synchronously after all DOM mutations but before the browser paints. This timing is crucial for DOM measurements and synchronous updates that need to be visible before the user sees the first paint, preventing visual flickering.

</details>

---

### Question 25: This data fetching component is causing an infinite loop. How do you fix it?

**Type:** Debugging | **Category:** Data Fetching

**The Loop:**
1. Component renders
2. `useEffect` runs (no dependency array = runs every render)
3. `setItems()` updates state → triggers re-render
4. Back to step 1... forever

**The Fix:** Add an empty dependency array `[]` to run the effect only once on mount.


<SmartCodeRunner mode="run" title="ItemList - Fix the Infinite Loop">
  <Fragment slot="code">
```jsx


const fetchItems = () => {
  console.log('Fetching data...');
  return new Promise(resolve => {
    setTimeout(() => resolve(['Item 1', 'Item 2', 'Item 3']), 500);
  });
};

export default function ItemList() {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // THE BUG: Missing dependency array!
    fetchItems().then(data => {
      setItems(data);
      setLoading(false);
    });
  }); // ← Add [] here

  if (loading) return <div>Loading items...</div>;

  return (
    <ul>
      {items.map((item, index) => <li key={index}>{item}</li>)}
    </ul>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```
Goal: Stop the infinite loop - "Fetching data..." should appear only once.
```
  </Fragment>
</SmartCodeRunner>

## The Solution

```jsx
useEffect(() => {
  fetchItems().then(data => {
    setItems(data);
    setLoading(false);
  });
}, []); // Empty array = run once on mount
```

**Dependency Array Rules:**
- `[]` → Run once on mount (like `componentDidMount`)
- `[dep1, dep2]` → Re-run when dependencies change
- No array → Run every render (almost never what you want)


---

### Quick Check

**What is the purpose of the dependency array in a useEffect hook?**

   A. It specifies which props the component should receive
-> B. **It tells React when to re-run the effect function**
   C. It defines the initial state values for the component
   D. It lists the external libraries that the effect depends on

<details>
<summary>See Answer</summary>

The dependency array tells React when to re-run the effect. An empty array means run once on mount. Including values means re-run when those values change. Omitting it entirely means run after every render.

</details>

---

### Question 26: This component is crashing with a JSX syntax error. How do you fix it?

**Type:** Debugging | **Category:** React Fundamentals

A developer is trying to create a simple `ItemList` component that renders a list of items. They've written JSX that looks like this:

```jsx
function ItemList() {
  return (
    <li>Item 1</li>
    <li>Item 2</li>
  );
}
```

However, their code is crashing with a syntax error, typically something like "Adjacent JSX elements must be wrapped in an enclosing tag."

## The Challenge

You've been given the `ItemList` component.

1.  Identify the JSX syntax error.
2.  Fix the component to correctly render a list of items without introducing unnecessary DOM nodes.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


export default function ItemList() {
  // THE BUG: JSX expressions must have exactly one outermost element.
  // Returning multiple sibling elements directly is a syntax error.
  return (
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  );
}

function App() {
  return (
    <div>
      <h1>My Shopping List</h1>
      <ul>
        <ItemList />
      </ul>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a syntax error that will prevent the code from running.
// 1. Run the code. You will see a compilation error in the console
//    (e.g., "Adjacent JSX elements must be wrapped in an enclosing tag").
//
// After your fix:
// 1. The application should render correctly, displaying "My Shopping List"
//    and a bulleted list of "Item 1", "Item 2", and "Item 3".

console.log("Goal: Fix the JSX syntax error and render the list correctly.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  );
}`}
  rightAnswer={`export default function ItemList() {
  // FIX: Wrap the multiple <li> elements in a React Fragment.
  return (
    <>
      <li>Item 1</li>
      <li>Item 2</li>
      <li>Item 3</li>
    </>
  );
}`}
/>

### Why This Works

The error "Adjacent JSX elements must be wrapped in an enclosing tag" is a fundamental rule of JSX. A JSX expression must always have **exactly one outermost element**. You cannot return multiple sibling elements directly from a component's `render` method or a JSX expression.

This rule exists because JSX is ultimately compiled into `React.createElement()` calls. `React.createElement()` can only create one element at a time. If you try to return multiple elements, it's ambiguous which one should be the "root" of that particular `createElement` call.

**The Fix: Wrapping with a Fragment**

To fix this, you need to wrap the multiple sibling elements in a single parent element. The most common and semantically correct way to do this without adding an extra, unnecessary DOM node is to use a **React Fragment**.

A React Fragment allows you to group a list of children without adding extra nodes to the DOM. You can use `React.Fragment` explicitly or, more commonly, the shorthand syntax `<>...</>`.

By wrapping the `<li>` elements in a Fragment, we satisfy the JSX rule of having a single root element without introducing an extra `div` that might interfere with styling or semantic HTML (especially important when rendering lists or table rows).

---

### Question 27: This component is breaking layout. Fix it using a React Fragment.

**Type:** Practical | **Category:** Component Patterns

A developer is building a table component. They have a `TableRows` component that is responsible for rendering a list of `<tr>` elements.

However, because a React component must return a single root element, they've wrapped their `<tr>` elements in a `div`. This `div` is breaking the table's layout, causing the rows to not align correctly, and also creating invalid semantic HTML, which can impact accessibility.

## The Challenge

You've been given the `TableRows` component. Your task is to fix the layout and semantic HTML issue by using a **React Fragment** to group the `<tr>` elements without adding an extra node to the DOM.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// This component is intended to be used inside a <table>
function TableRows() {
  // THE BUG: Wrapping <tr> elements directly in a <div> breaks table semantics.
  return (
    <div>
      <tr>
        <td>Row 1, Cell 1</td>
        <td>Row 1, Cell 2</td>
      </tr>
      <tr>
        <td>Row 2, Cell 1</td>
        <td>Row 2, Cell 2</td>
      </tr>
    </div>
  );
}

export default function App() {
  return (
    <table border="1" style={{ width: '100%', borderCollapse: 'collapse' }}>
      <thead>
        <tr>
          <th>Column A</th>
          <th>Column B</th>
        </tr>
      </thead>
      <tbody>
        <TableRows /> {/* This is where the issue manifests */}
      </tbody>
    </table>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. You will observe that the table rows are not
//    aligned correctly under the headers, and the table borders
//    might look broken. This is because the <div> is an invalid
//    child of <tbody>.
//
// After your fix:
// 1. The table rows should align perfectly under their headers.
// 2. The table borders should render correctly, indicating valid
//    semantic HTML structure.

console.log("Goal: Fix the table layout and semantic HTML using a React Fragment.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

 elements directly in a <div> breaks table semantics.
  return (
    <div>
      <tr>
        <td>Row 1, Cell 1</td>
        <td>Row 1, Cell 2</td>
      </tr>
      <tr>
        <td>Row 2, Cell 1</td>
        <td>Row 2, Cell 2</td>
      </tr>
    </div>
  );
}`}
  rightAnswer={`function TableRows() {
  // FIX: Use a React Fragment (shorthand syntax) to group the <tr> elements.
  // This allows them to be direct children of <tbody> without an extra DOM node.
  return (
    <>
      <tr>
        <td>Row 1, Cell 1</td>
        <td>Row 1, Cell 2</td>
      </tr>
      <tr>
        <td>Row 2, Cell 1</td>
        <td>Row 2, Cell 2</td>
      </tr>
    </>
  );
}`}
/>

### Why This Works

React components must return a single root element. This is a fundamental rule of JSX. However, sometimes you need to group multiple elements without introducing an unnecessary `div` or other wrapper element into the DOM. This is especially true when dealing with semantic HTML structures like tables, lists, or flexbox/grid layouts, where an extra `div` can break the styling or accessibility.

**The Fix: Using React Fragments**

**React Fragments** allow you to group a list of children without adding extra nodes to the DOM. They solve the "single root element" requirement without interfering with your HTML structure.

You can use `React.Fragment` explicitly or, more commonly, the shorthand syntax `<>...</>`.

By replacing the `div` with `<>`, the `<tr>` elements are now direct children of `<tbody>`, which is semantically correct and resolves the layout issues.


---

### Quick Check

**What is the primary benefit of using React Fragments?**

   A. They improve component performance by memoizing children.
-> B. **They allow you to group multiple elements without adding an extra node to the DOM.**
   C. They provide a way to pass data between sibling components.
   D. They enable server-side rendering for React applications.

<details>
<summary>See Answer</summary>

They allow you to group multiple elements without adding an extra node to the DOM. This is particularly important when working with semantic HTML structures like tables, lists, or CSS layouts where an extra wrapper element could break the intended structure or styling.

</details>

---

### Question 28: This component is trying to modify a prop directly. How do you fix it?

**Type:** Debugging | **Category:** Props & State

A developer has a `Greeting` component that receives a `name` prop (e.g., "Alice"). They want to allow the user to edit this name directly within the `Greeting` component using an input field.

They tried to implement this by adding an input field and updating the `name` prop using `setName(event.target.value)`. However, the application is crashing or showing a warning about trying to modify a read-only property.

## The Challenge

You've been given the `App` and `Greeting` components.

1.  Explain the fundamental difference between **state** and **props** in React.
2.  Fix the component to allow the user to edit the name correctly, adhering to React's principles.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// The Greeting component receives a name prop
function Greeting({ name }) {
  // THE BUG: Trying to modify a prop directly.
  // Props are read-only! This will cause an error or warning.
  const handleNameChange = (event) => {
    // This line is problematic:
    // name = event.target.value; // Cannot assign to read only property 'name' of object
    console.log("Attempting to change prop directly:", event.target.value);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', margin: '10px' }}>
      <h1>Hello, {name}!</h1>
      <label>
        Edit Name:
        <input type="text" value={name} onChange={handleNameChange} />
      </label>
      <p>Try typing in the input field.</p>
    </div>
  );
}

export default function App() {
  // The App component currently just passes a static name
  return (
    <div style={{ padding: '20px' }}>
      <Greeting name="Alice" />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual and console-based test.
// 1. Run the code.
// 2. Try typing in the input field.
// 3. With the bug, the input field will be read-only (you can't type),
//    and you might see an error in the console about trying to modify a prop.
//
// After your fix:
// 1. You should be able to type in the input field.
// 2. The name displayed in the <h1> tag should update in real-time as you type.

console.log("Goal: Make the input editable and update the displayed name.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

 {
    // THE BUG: Trying to modify a prop directly
    name = event.target.value; // Cannot assign to read only property 'name'
    console.log("Attempting to change prop directly:", event.target.value);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', margin: '10px' }}>
      <h1>Hello, {name}!</h1>
      <label>
        Edit Name:
        <input type="text" value={name} onChange={handleNameChange} />
      </label>
    </div>
  );
}

export default function App() {
  return (
    <div style={{ padding: '20px' }}>
      <Greeting name="Alice" />
    </div>
  );
}`}
  rightAnswer={`// The Greeting component now receives the name and a function to change it
function Greeting({ name, onNameChange }) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', margin: '10px' }}>
      <h1>Hello, {name}!</h1>
      <label>
        Edit Name:
        {/* FIX: Input value is controlled by the 'name' prop,
            and changes are requested via 'onNameChange' prop. */}
        <input type="text" value={name} onChange={onNameChange} />
      </label>
    </div>
  );
}

export default function App() {
  // FIX: The 'name' is now managed as state in the parent component (App).
  const [userName, setUserName] = useState('Alice');

  const handleUserNameChange = (event) => {
    setUserName(event.target.value);
  };

  return (
    <div style={{ padding: '20px' }}>
      {/* FIX: Pass the state and the setter function (or a wrapper) as props. */}
      <Greeting name={userName} onNameChange={handleUserNameChange} />
    </div>
  );
}`}
/>

### Why This Works

The core issue here is a misunderstanding of the fundamental difference between **state** and **props** in React:

*   **Props (Properties)**:
    *   Are passed from a parent component to a child component.
    *   Are **read-only** within the child component. A child component should never attempt to modify its props directly.
    *   Are used to configure a component and pass data down the component tree.
*   **State**:
    *   Is managed internally within a component.
    *   Is **mutable** and can change over time.
    *   Is used for data that a component needs to keep track of and that might change (e.g., user input, toggled visibility, fetched data).

In the buggy code, the `Greeting` component receives `name` as a prop. Trying to assign a new value to `name` within `handleNameChange` directly violates the immutability of props, leading to an error.

**The Fix: Lifting State Up and Passing a Callback**

To allow the `name` to be editable, the `name` data must be managed as **state** by the component that "owns" it (typically the closest common ancestor that needs to manage or react to its changes). The child component then receives the current value as a prop and a **callback function** (also as a prop) to request changes from the parent.

Now, the `App` component owns the `userName` state. It passes the current `userName` as a prop to `Greeting` and also passes a `handleUserNameChange` function. When the user types in the input, `Greeting` calls `handleUserNameChange`, which updates the `userName` state in `App`. This triggers a re-render of `App` and `Greeting` with the new `userName`, correctly updating the UI.

---

### Question 29: This application suffers from 'prop drilling.' How do you fix it?

**Type:** Practical | **Category:** State Management

You're working on a React application with a `UserProfile` component that needs to display the user's preferred `theme`. The `theme` state (e.g., 'light' or 'dark') is managed at the top-level `App` component.

To get the `theme` down to `UserProfile`, it's currently being passed as a prop through several intermediate components: `App -> Dashboard -> Settings -> UserProfile`. The `Dashboard` and `Settings` components don't actually use the `theme` prop themselves; they just pass it along. This is a classic case of **prop drilling**.

This makes the code verbose, harder to read, and more difficult to refactor, as changes to the `theme` prop would require modifying multiple intermediate components.

## The Challenge

Your task is to refactor the application to eliminate prop drilling. Use **React Context** to provide the `theme` and a function to `setTheme` directly to the `UserProfile` component (and any other component that needs it), bypassing the intermediate components.

The application's functionality (toggling the theme and displaying it) should remain identical.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Component that needs the theme
function UserProfile({ theme }) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', marginTop: '10px' }}>
      <h3>User Profile</h3>
      <p>Current Theme: {theme}</p>
    </div>
  );
}

// Intermediate component 1 (doesn't use theme, just passes it)
function Settings({ theme }) {
  return (
    <div style={{ border: '1px solid #eee', padding: '10px', margin: '10px' }}>
      <h4>Settings Component</h4>
      <UserProfile theme={theme} />
    </div>
  );
}

// Intermediate component 2 (doesn't use theme, just passes it)
function Dashboard({ theme }) {
  return (
    <div style={{ border: '1px solid #ddd', padding: '10px', margin: '10px' }}>
      <h2>Dashboard Component</h2>
      <Settings theme={theme} />
    </div>
  );
}

export default function App() {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  return (
    <div style={{ padding: '20px', background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff' }}>
      <h1>App Component</h1>
      <button onClick={toggleTheme}>Toggle Theme</button>
      <Dashboard theme={theme} /> {/* Prop drilling starts here */}
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. You will see the "User Profile" displaying the current theme.
// 2. Click the "Toggle Theme" button.
// 3. The theme displayed in the User Profile and the background of the App
//    should change correctly.
//
// After your fix:
// 1. The visual output and functionality should remain identical.
// 2. The `Dashboard` and `Settings` components should no longer receive
//    the `theme` prop.

console.log("Goal: Eliminate prop drilling for the 'theme' prop using React Context.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution


      <h3>User Profile</h3>
      <p>Current Theme: {theme}</p>
    </div>
  );
}

// Intermediate component 1 (doesn't use theme, just passes it)
function Settings({ theme }) {
  return (
    <div style={{ border: '1px solid #eee', padding: '10px', margin: '10px' }}>
      <h4>Settings Component</h4>
      <UserProfile theme={theme} />
    </div>
  );
}

// Intermediate component 2 (doesn't use theme, just passes it)
function Dashboard({ theme }) {
  return (
    <div style={{ border: '1px solid #ddd', padding: '10px', margin: '10px' }}>
      <h2>Dashboard Component</h2>
      <Settings theme={theme} />
    </div>
  );
}

export default function App() {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  return (
    <div style={{ padding: '20px', background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff' }}>
      <h1>App Component</h1>
      <button onClick={toggleTheme}>Toggle Theme</button>
      <Dashboard theme={theme} /> {/* Prop drilling starts here */}
    </div>
  );
}`}
  rightAnswer={`import React, { useState, useContext, createContext } from 'react';

// 1. Create a ThemeContext
const ThemeContext = createContext();

// Component that needs the theme - now consumes context directly
function UserProfile() {
  const { theme } = useContext(ThemeContext); // Consume theme from context
  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', marginTop: '10px' }}>
      <h3>User Profile</h3>
      <p>Current Theme: {theme}</p>
    </div>
  );
}

// Intermediate component 1 - no longer needs to pass theme prop
function Settings() {
  return (
    <div style={{ border: '1px solid #eee', padding: '10px', margin: '10px' }}>
      <h4>Settings Component</h4>
      <UserProfile /> {/* No theme prop needed */}
    </div>
  );
}

// Intermediate component 2 - no longer needs to pass theme prop
function Dashboard() {
  return (
    <div style={{ border: '1px solid #ddd', padding: '10px', margin: '10px' }}>
      <h2>Dashboard Component</h2>
      <Settings /> {/* No theme prop needed */}
    </div>
  );
}

export default function App() {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  return (
    // 2. Provide the theme and toggle function via ThemeContext.Provider
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <div style={{ padding: '20px', background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff' }}>
        <h1>App Component</h1>
        <button onClick={toggleTheme}>Toggle Theme</button>
        <Dashboard /> {/* No theme prop needed */}
      </div>
    </ThemeContext.Provider>
  );
}`}
/>

### Why This Works

**Prop drilling** is an anti-pattern where props are passed down through multiple levels of components in a component tree, even if those intermediate components don't actually need the data. This makes the code verbose, harder to read, and more difficult to maintain because any change to the prop's name or type requires updating all intermediate components.

**The Fix: Using React Context**

**React Context** provides a way to share values like `theme` or user authentication status between components without having to explicitly pass a prop through every level of the tree. It's designed for "global" data that many components might need.

Here's how to refactor using Context:
1.  **Create a Context**: Define a `ThemeContext` using `React.createContext()`.
2.  **Provide the Context**: Wrap the part of your component tree that needs access to the context with `ThemeContext.Provider`. Pass the `theme` state and the `setTheme` function (or `toggleTheme` function) as the `value` prop to the provider.
3.  **Consume the Context**: In any component that needs the `theme` (like `UserProfile`), use the `useContext` hook to directly access the values from the context.

By using `ThemeContext`, we've eliminated the need to pass the `theme` prop through `Dashboard` and `Settings`. `UserProfile` now directly accesses the `theme` from the context, making the component tree cleaner and more maintainable.

---

### Question 30: This component is crashing due to a 'Rules of Hooks' violation. How do you fix it?

**Type:** Debugging | **Category:** Hooks & Patterns

> **Common Mistake:** Maybe wrap useState in a try-catch to handle the error?

> **Senior Engineer Approach:** The useState is inside an if statement, which violates the Rules of Hooks. React relies on the call order of hooks being the same on every render. Conditional hooks break this:

**The Rule:** Don't call Hooks inside loops, conditions, or nested functions.

**Why It Matters:** React tracks hooks by their call order. If a hook is called conditionally, the order changes between renders, corrupting React's internal state tracking.

**The Fix:** Always call hooks at the top level. Use conditional logic *after* the hook call.


<SmartCodeRunner mode="run" title="ConditionalCounter - Fix the Rules of Hooks Violation">
  <Fragment slot="code">
```jsx


export default function ConditionalCounter({ showCounter }) {
  // THE BUG: useState inside a condition!
  if (showCounter) {
    const [count, setCount] = useState(0); // This crashes!

    return (
      <div>
        <p>Count: {count}</p>
        <button onClick={() => setCount(count + 1)}>Increment</button>
      </div>
    );
  }

  return <p>Counter is hidden.</p>;
}

function App() {
  const [show, setShow] = useState(true);
  return (
    <div>
      <button onClick={() => setShow(!show)}>Toggle Counter</button>
      <ConditionalCounter showCounter={show} />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```
Goal: Fix the "Rules of Hooks" violation - hooks must be called at top level.
```
  </Fragment>
</SmartCodeRunner>

## The Solution

**Option 1: Always call the hook, conditionally render the output**

```jsx
export default function ConditionalCounter({ showCounter }) {
  // Always call hooks at the top level
  const [count, setCount] = useState(0);

  if (!showCounter) {
    return <p>Counter is hidden.</p>;
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Option 2: Conditionally render the entire component**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}

function App() {
  const [show, setShow] = useState(true);
  return show ? <Counter /> : <p>Hidden</p>;
}
```


---

### Quick Check

**Which of the following is a valid rule for using React Hooks?**

   A. Hooks can be called inside any JavaScript function
-> B. **Hooks must be called at the top level of a React function component or a custom Hook**
   C. Hooks can be called inside loops and conditional statements
   D. Hooks can only be used in class components

<details>
<summary>See Answer</summary>

Hooks must be called at the top level of React function components or custom hooks. They cannot be called inside conditions, loops, or nested functions because React relies on the consistent call order of hooks to maintain state correctly across renders.

</details>

---

### Question 31: This custom component is not forwarding its ref. How do you fix it?

**Type:** Practical | **Category:** Component Patterns

A developer has created a reusable `CustomInput` component. They want to use a `ref` from a parent component to directly focus this `CustomInput` when a button is clicked.

However, when they click the "Focus Input" button, nothing happens. If they check the console, they might see a warning like "Function components cannot be given refs. Attempts to access this ref will fail." or simply find that `inputRef.current` is `null`.

## The Challenge

You've been given the `App` and `CustomInput` components.

1.  Explain why the `ref` is not being forwarded correctly to the internal `<input>` element.
2.  Fix the `CustomInput` component using `React.forwardRef` so that the parent component can successfully focus the input.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// The CustomInput component - a functional component
function CustomInput(props) {
  // THE BUG: Refs are not automatically passed through functional components.
  // The 'ref' prop here is just a regular prop and won't attach to the <input>.
  return <input type="text" {...props} />;
}

export default function App() {
  const inputRef = useRef(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
      inputRef.current.style.borderColor = 'blue';
    } else {
      console.log("Ref is not attached to the input element!");
    }
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Ref Forwarding Example</h1>
      {/* Attempting to pass a ref to a functional component */}
      <CustomInput ref={inputRef} placeholder="Type something..." />
      <button onClick={focusInput} style={{ marginLeft: '10px' }}>
        Focus Input
      </button>
      <p>Click the button to try and focus the input.</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual and console-based test.
// 1. Run the code.
// 2. Click the "Focus Input" button.
// 3. With the bug, the input will not gain focus, and you will see
//    "Ref is not attached to the input element!" in the console.
//
// After your fix:
// 1. Clicking the "Focus Input" button should successfully focus the input field,
//    and its border should turn blue.
// 2. The console message should no longer appear.

console.log("Goal: Make the 'Focus Input' button successfully focus the CustomInput.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

.
  return <input type="text" {...props} />;
}

export default function App() {
  const inputRef = useRef(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
      inputRef.current.style.borderColor = 'blue';
    } else {
      console.log("Ref is not attached to the input element!");
    }
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Ref Forwarding Example</h1>
      {/* Attempting to pass a ref to a functional component */}
      <CustomInput ref={inputRef} placeholder="Type something..." />
      <button onClick={focusInput} style={{ marginLeft: '10px' }}>
        Focus Input
      </button>
    </div>
  );
}`}
  rightAnswer={`// FIX: Wrap CustomInput with React.forwardRef
const CustomInput = forwardRef((props, ref) => {
  // The 'ref' argument is now the ref passed from the parent.
  // Attach it to the internal <input> element.
  return <input type="text" ref={ref} {...props} />;
});

export default function App() {
  const inputRef = useRef(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
      inputRef.current.style.borderColor = 'blue';
    } else {
      console.log("Ref is not attached to the input element!");
    }
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Ref Forwarding Example</h1>
      {/* Now, passing a ref to CustomInput works correctly */}
      <CustomInput ref={inputRef} placeholder="Type something..." />
      <button onClick={focusInput} style={{ marginLeft: '10px' }}>
        Focus Input
      </button>
    </div>
  );
}`}
/>

### Why This Works

The problem arises because, by default, React functional components do not automatically "forward" refs to their internal DOM elements. When you pass a `ref` prop to a custom functional component (like `CustomInput`), React treats it as a regular prop. It doesn't know that you intend for that `ref` to attach to an internal DOM element.

**The Fix: Using `React.forwardRef`**

`React.forwardRef` is a higher-order function that lets you pass a ref through a component to one of its children. It takes a functional component as an argument and returns a new React component that can receive a `ref` prop and pass it down.

The component wrapped with `forwardRef` receives `props` as its first argument and the `ref` as its second argument. You then attach this `ref` to the specific DOM element you want to expose.

By using `React.forwardRef`, the `CustomInput` component now correctly receives the `ref` from `App` and attaches it to its internal `<input>` element. This allows the `App` component to directly access and manipulate the `<input>` element, such as calling its `focus()` method.


---

### Quick Check

**What is the primary purpose of React.forwardRef?**

   A. To create a new component that automatically focuses an input on mount.
-> B. **To allow a parent component to pass a ref to a DOM element or class component exposed by a child functional component.**
   C. To prevent unnecessary re-renders of child components.
   D. To enable server-side rendering for functional components.

<details>
<summary>See Answer</summary>

To allow a parent component to pass a ref to a DOM element or class component exposed by a child functional component. React.forwardRef enables ref forwarding, which is essential when building reusable component libraries where parent components need direct access to DOM elements within custom components.

</details>

---

### Question 32: This `useEffect` is logging a stale value. How do you fix its dependency array?

**Type:** Debugging | **Category:** Hooks & Patterns

> **Common Mistake:** Just remove the setInterval and use a different approach.

> **Senior Engineer Approach:** This is a classic stale closure bug. The empty dependency array means the effect captures the initial count (0) forever. The fix is to include count in the dependency array:

**Why It's Stale:**
- Empty `[]` = effect runs once on mount
- The `setInterval` callback "closes over" `count = 0`
- Even when state updates, the callback still sees `count = 0`

**The Fix:** Add `count` to the dependency array. The effect will re-run when count changes, recreating the interval with the fresh value.


<SmartCodeRunner mode="run" title="Counter - Fix the Stale Closure">
  <Fragment slot="code">
```jsx


export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // THE BUG: Captures initial count (0) due to empty []
    const intervalId = setInterval(() => {
      console.log(`Count from useEffect: ${count}`);
    }, 2000);

    return () => clearInterval(intervalId);
  }, []); // ← Should include count

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <p>Check console - it always logs 0!</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```
Goal: Console should log the current count, not always 0.
```
  </Fragment>
</SmartCodeRunner>

## The Solution

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log(`Count from useEffect: ${count}`);
  }, 2000);

  return () => clearInterval(intervalId); // Cleanup is crucial!
}, [count]); // Re-run when count changes
```

The cleanup function clears the old interval before creating a new one, preventing multiple intervals from stacking up.

---

### Question 33: This component's UI is not updating. Why is direct state mutation problematic?

**Type:** Debugging | **Category:** State Management

A developer has built a simple `TodoList` component. It displays a list of tasks, and each task has a "Mark Complete" button. When the button is clicked, the task's `completed` status should toggle, and the UI should update to reflect this change.

However, they've noticed a bug: clicking "Mark Complete" does not update the UI. The task's status remains unchanged visually, even though the underlying data *seems* to be modified.

## The Challenge

You've been given the `TodoList` component.

1.  Explain why directly mutating state (or objects within state) is problematic in React and why the UI is not updating.
2.  Fix the component to update the todo's status **immutably**, ensuring the UI correctly reflects the changes.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


export default function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build a project', completed: false },
  ]);

  const toggleComplete = (id) => {
    const todo = todos.find(t => t.id === id);
    if (todo) {
      // THE BUG: Directly mutating the 'completed' property of the todo object.
      // React's shallow comparison won't detect this change.
      todo.completed = !todo.completed;
      setTodos(todos); // Passing the same array reference
      console.log(`Todo ${id} completed status: ${todo.completed}`);
    }
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Todo List</h1>
      <ul>
        {todos.map(todo => (
          <li key={todo.id} style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            {todo.text}
            <button onClick={() => toggleComplete(todo.id)} style={{ marginLeft: '10px' }}>
              {todo.completed ? 'Undo' : 'Mark Complete'}
            </button>
          </li>
        ))}
      </ul>
      <p>Click "Mark Complete" and observe the UI.</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual and console-based test.
// 1. Run the code.
// 2. Click "Mark Complete" for any todo item.
// 3. With the bug, the text decoration will NOT change,
//    even though the console log might show the 'completed' status changing.
//
// After your fix:
// 1. Clicking "Mark Complete" should visually toggle the line-through
//    decoration for the respective todo item.

console.log("Goal: Ensure the UI updates correctly when a todo's completed status changes.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

 {
  const todo = todos.find(t => t.id === id);
  if (todo) {
    // THE BUG: Directly mutating the 'completed' property of the todo object.
    // React's shallow comparison won't detect this change.
    todo.completed = !todo.completed;
    setTodos(todos); // Passing the same array reference
    console.log(\`Todo \${id} completed status: \${todo.completed}\`);
  }
};`}
  rightAnswer={`const toggleComplete = (id) => {
  // FIX: Update state immutably by creating new objects/arrays.
  setTodos(prevTodos =>
    prevTodos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed } // Create a NEW todo object
        : todo
    )
  );
};`}
/>

### Why This Works

The problem arises because React relies on **immutability** for efficient change detection during its **reconciliation** process.

When you update state in React (e.g., by calling `setTodos`), React performs a shallow comparison between the new state value and the previous state value.
*   If the new state value is a primitive (number, string, boolean) and is different, React knows to re-render.
*   If the new state value is an object or an array, React only compares their **references**. If the reference is the same, React assumes the content hasn't changed and *does not trigger a re-render*.

In the buggy code:
1.  `todo.completed = !todo.completed;` directly mutates the `todo` object within the `todos` array.
2.  `setTodos(todos);` is then called. However, the `todos` array itself is the *same array instance* as before. Its reference hasn't changed.

Because the `todos` array's reference remains the same, React's shallow comparison doesn't detect a change in the `todos` state, and therefore, the `TodoList` component does not re-render, leading to a stale UI.

**The Fix: Updating State Immutably**

To correctly update state that contains objects or arrays, you must always create a **new object or array** with the desired changes. This ensures that React detects a new reference and triggers a re-render.

Here's how to update the `todos` array immutably:
1.  Map over the `todos` array.
2.  When you find the `todo` to update, create a *new* `todo` object with the updated properties (using spread syntax `...`).
3.  Return a *new* array containing the updated `todo` and the other unchanged `todos`.

By creating a new `todo` object and a new `todos` array, React now detects a change in the `todos` state's reference, triggering a re-render and correctly updating the UI.

---

### Question 34: This component has issues managing a timer ID. Fix it with `useRef`.

**Type:** Practical | **Category:** Hooks & Patterns

A developer is building a `Timer` component that needs to set up a `setInterval` when it mounts and clear it when it unmounts. They are trying to store the `intervalId` (returned by `setInterval`) in a `useState` variable so they can access it in the cleanup function.

However, they've noticed that every time the component re-renders (e.g., due to a parent component's state change), the `intervalId` state update causes another re-render, leading to unexpected behavior or even an infinite loop if not handled carefully.

## The Challenge

You've been given the `Timer` component.

1.  Explain why storing the `intervalId` in `useState` is problematic in this scenario.
2.  Fix the component to correctly store the `intervalId` using the `useRef` hook, ensuring it persists across renders without triggering unnecessary re-renders.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


export default function Timer() {
  const [seconds, setSeconds] = useState(0);
  // THE BUG: Storing intervalId in useState.
  // Updating this state will cause re-renders, which is not needed for a mutable ID.
  const [intervalId, setIntervalId] = useState(null);

  useEffect(() => {
    console.log('Setting up interval...');
    const id = setInterval(() => {
      setSeconds(prevSeconds => prevSeconds + 1);
    }, 1000);

    // Storing the ID in state causes a re-render, which can be problematic.
    setIntervalId(id);

    return () => {
      console.log('Clearing interval...');
      clearInterval(intervalId); // This intervalId might be stale if not careful
    };
  }, []); // Empty dependency array to run once on mount/unmount

  return (
    <div style={{ padding: '20px' }}>
      <h1>Timer: {seconds}s</h1>
      <p>Check the console for setup/cleanup logs.</p>
    </div>
  );
}

function App() {
  const [showTimer, setShowTimer] = useState(true);
  return (
    <div>
      <button onClick={() => setShowTimer(!showTimer)}>
        {showTimer ? 'Hide Timer' : 'Show Timer'}
      </button>
      <hr />
      {showTimer && <Timer />}
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. You will see "Setting up interval..." and "Clearing interval..."
//    logs, potentially more than expected, or the timer might behave oddly.
// 3. Click "Hide Timer" and "Show Timer" multiple times.
//
// After your fix:
// 1. The timer should increment smoothly.
// 2. "Setting up interval..." should only log once when the Timer component mounts.
// 3. "Clearing interval..." should only log once when the Timer component unmounts.
// 4. No unnecessary re-renders should be triggered by storing the interval ID.

console.log("Goal: Correctly manage the setInterval ID without causing unnecessary re-renders.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

 {
    console.log('Setting up interval...');
    const id = setInterval(() => {
      setSeconds(prevSeconds => prevSeconds + 1);
    }, 1000);

    // Storing the ID in state causes a re-render, which can be problematic.
    setIntervalId(id);

    return () => {
      console.log('Clearing interval...');
      clearInterval(intervalId); // This intervalId might be stale if not careful
    };
  }, []); // Empty dependency array to run once on mount/unmount

  return (
    <div style={{ padding: '20px' }}>
      <h1>Timer: {seconds}s</h1>
      <p>Check the console for setup/cleanup logs.</p>
    </div>
  );
}`}
  rightAnswer={`export default function Timer() {
  const [seconds, setSeconds] = useState(0);
  // FIX: Use useRef to store the intervalId.
  // Updating intervalRef.current does NOT trigger a re-render.
  const intervalRef = useRef(null);

  useEffect(() => {
    console.log('Setting up interval...');
    const id = setInterval(() => {
      setSeconds(prevSeconds => prevSeconds + 1);
    }, 1000);

    // Store the interval ID in the ref's .current property.
    intervalRef.current = id;

    return () => {
      console.log('Clearing interval...');
      // Access the stored ID from the ref's .current property for cleanup.
      clearInterval(intervalRef.current);
    };
  }, []); // Empty dependency array to run once on mount/unmount

  return (
    <div style={{ padding: '20px' }}>
      <h1>Timer: {seconds}s</h1>
      <p>Check the console for setup/cleanup logs.</p>
    </div>
  );
}`}
/>

### Why This Works

The problem arises from using `useState` to store the `intervalId`. While `useState` is perfect for managing state that should trigger re-renders, it's not ideal for values that are mutable but whose changes *should not* cause a re-render.

When `setIntervalId(id)` is called inside `useEffect`, it updates the component's state. This state update triggers a re-render of the `Timer` component. If the `useEffect`'s dependency array is not carefully managed, this can lead to an infinite loop or multiple intervals running simultaneously (as seen in the Strict Mode example). Even with an empty dependency array, the `setIntervalId` call itself causes a re-render, which is unnecessary for a value that's only used internally for cleanup.

**The Fix: Using `useRef` for Mutable, Non-Rendering Values**

The `useRef` hook is designed for exactly this kind of scenario. It returns a mutable `ref` object whose `.current` property can hold any value. Crucially, updating the `.current` property of a ref **does not trigger a re-render** of the component. The ref object itself persists across renders.

This makes `useRef` ideal for:
*   Accessing DOM elements directly.
*   Storing mutable values (like timer IDs, previous state values, or any instance variable) that need to persist across renders but whose changes shouldn't cause the component to update its UI.

By using `useRef`, the `intervalId` is stored in a mutable container that persists across renders without causing any additional re-renders when it's updated. This ensures the `setInterval` is managed correctly and efficiently.

---

### Question 35: This optimized component re-renders unnecessarily. How do you fix it with `useCallback`?

**Type:** Practical | **Category:** Performance

A developer has an `App` component that manages a `count` state. It renders a child component, `OptimizedChild`, which is wrapped in `React.memo` to prevent unnecessary re-renders. The `App` component passes a `handleClick` function as a prop to `OptimizedChild`.

However, they've noticed a performance issue: `OptimizedChild` still re-renders every time the `App` component re-renders (e.g., when `count` changes), even though `OptimizedChild`'s props (specifically the `handleClick` function) *seem* to be the same. This defeats the purpose of `React.memo`.

## The Challenge

You've been given the `App` and `OptimizedChild` components.

1.  Explain why `OptimizedChild` re-renders unnecessarily despite being wrapped in `React.memo`.
2.  Fix the `App` component using the `useCallback` hook to prevent `OptimizedChild` from re-rendering when `count` changes.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// OptimizedChild is wrapped in React.memo
const OptimizedChild = memo(function OptimizedChild({ onClick }) {
  console.log('Re-rendering OptimizedChild...');
  return (
    <button onClick={onClick} style={{ padding: '10px', margin: '10px' }}>
      Click me (Child)
    </button>
  );
});

export default function App() {
  const [count, setCount] = useState(0);

  // THE BUG: This function is re-created on every render of App.
  // Its reference changes, causing OptimizedChild to re-render.
  const handleClick = () => {
    console.log('Child button clicked!');
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Parent Count: {count}</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Increment Parent Count
      </button>
      <hr />
      <OptimizedChild onClick={handleClick} />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. You will see "Re-rendering OptimizedChild..."
//    on initial render.
// 3. Click the "Increment Parent Count" button.
// 4. With the bug, you will see "Re-rendering OptimizedChild..." logged again,
//    even though the `onClick` function's logic hasn't changed.
//
// After your fix:
// 1. Clicking "Increment Parent Count" should NOT cause
//    "Re-rendering OptimizedChild..." to be logged.
// 2. Clicking "Click me (Child)" should still log "Child button clicked!".

console.log("Goal: Prevent OptimizedChild from re-rendering when its function prop's reference is stable.");
```
  </Fragment>
</SmartCodeRunner>

## The Solution

 {
    console.log('Child button clicked!');
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Parent Count: {count}</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Increment Parent Count
      </button>
      <hr />
      <OptimizedChild onClick={handleClick} />
    </div>
  );
}`}
  rightAnswer={`export default function App() {
  const [count, setCount] = useState(0);

  // FIX: Memoize the handleClick function using useCallback.
  // The function's reference will now only change if its dependencies change.
  // In this case, handleClick doesn't depend on 'count' or any other state/props,
  // so an empty dependency array ensures it's created only once.
  const handleClick = useCallback(() => {
    console.log('Child button clicked!');
  }, []); // Empty dependency array means this function is stable across renders

  return (
    <div style={{ padding: '20px' }}>
      <h1>Parent Count: {count}</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Increment Parent Count
      </button>
      <hr />
      <OptimizedChild onClick={handleClick} />
    </div>
  );
}`}
/>

### Why This Works

The problem lies in how JavaScript handles functions and how `React.memo` performs its shallow comparison of props.

1.  **Functions are Objects**: In JavaScript, functions are first-class citizens, meaning they are objects. When you define a function directly inside a component's render function (like `handleClick` in `App`), a **new function object** is created on *every single render* of the `App` component.
2.  **`React.memo` and Shallow Comparison**: `React.memo` is a Higher-Order Component that prevents a functional component from re-rendering if its props have not changed. However, `React.memo` performs a **shallow comparison** of props. When `App` re-renders, it creates a *new* `handleClick` function. Even though the *code* inside `handleClick` is the same, its *reference* is different from the previous render.
3.  **Defeating Memoization**: `React.memo` sees that the `onClick` prop (which is the `handleClick` function) has a new reference, concludes that the prop has changed, and therefore re-renders `OptimizedChild`, defeating the optimization.

**The Fix: Memoizing Functions with `useCallback`**

To prevent this unnecessary re-render, we need to ensure that the `handleClick` function's reference remains stable across renders, as long as its dependencies don't change. This is precisely what the `useCallback` hook is for.

`useCallback` returns a memoized version of the callback function. It will only re-create the function (and thus change its reference) if any of its dependencies (specified in the dependency array) have changed.

With `useCallback`, the `handleClick` function's reference remains stable across `App` re-renders (because its dependencies `[]` never change). `React.memo` on `OptimizedChild` now correctly sees that the `onClick` prop's reference hasn't changed, and thus prevents `OptimizedChild` from re-rendering unnecessarily.


---

### Quick Check

**What is the primary purpose of the useCallback hook?**

   A. To fetch data from an API and cache the results.
-> B. **To memoize a function, preventing it from being re-created on every render unless its dependencies change.**
   C. To create a new state variable that can be shared across components.
   D. To perform side effects after every render of a component.

<details>
<summary>See Answer</summary>

To memoize a function, preventing it from being re-created on every render unless its dependencies change. This is particularly useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (like components wrapped in React.memo).

</details>

---

### Question 36: This large component is slowing down initial load. Implement code splitting.

**Type:** Practical | **Category:** Performance

## The Scenario

You're working on a React application that includes a large `AdminDashboard` component. This dashboard is only accessible to users with administrator privileges.

Currently, the `AdminDashboard` component is directly imported and bundled with the main application code. This means that all users, including regular users who will never access the dashboard, download its code on initial page load. This is making the initial load time of the application unnecessarily slow.

## The Challenge

Your task is to refactor the application to implement **code splitting**. Load the `AdminDashboard` component only when it's actually needed (i.e., when the `showAdmin` flag is true).

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Simulate a large component that logs when it's loaded
function AdminDashboard() {
  console.log('AdminDashboard loaded!');
  return (
    <div style={{ border: '1px solid green', padding: '20px', margin: '10px' }}>
      <h2>Admin Dashboard</h2>
      <p>Welcome, Administrator!</p>
      <p>Here are your sensitive admin controls.</p>
    </div>
  );
}

export default function App() {
  const [showAdmin, setShowAdmin] = useState(false);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Main Application</h1>
      <button onClick={() => setShowAdmin(!showAdmin)}>
        {showAdmin ? 'Hide Admin Dashboard' : 'Show Admin Dashboard'}
      </button>
      <hr />
      {/* THE BUG: AdminDashboard is statically imported and bundled,
          even if showAdmin is false. */}
      {showAdmin && <AdminDashboard />}
      {!showAdmin && <p>Admin Dashboard is currently hidden.</p>}
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. With the bug, you will see "AdminDashboard loaded!"
//    logged immediately on initial page load, even before you click the button.
//
// After your fix:
// 1. "AdminDashboard loaded!" should NOT be logged on initial page load.
// 2. It should only be logged *after* you click the "Show Admin Dashboard" button.

console.log("Goal: Load AdminDashboard code only when it's needed.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: Improving Performance with Code Splitting

<JuniorVsSenior
  juniorAnswer={{
    title: "Static Import (Loads Everything Upfront)",
    code: `import React, { useState } from 'react';


export default function App() {
  const [showAdmin, setShowAdmin] = useState(false);

  return (
    <div>
      {showAdmin && <AdminDashboard />}
    </div>
  );
}`,
    explanation: "Static imports bundle the component with the main application code. Even if the component is conditionally rendered, its code is downloaded on initial load, increasing bundle size unnecessarily."
  }}
  seniorAnswer={{
    title: "Dynamic Import with Code Splitting",
    code: `import React, { useState, Suspense } from 'react';

const AdminDashboard = React.lazy(() =>
  import('./AdminDashboard')
);

export default function App() {
  const [showAdmin, setShowAdmin] = useState(false);

  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        {showAdmin && <AdminDashboard />}
      </Suspense>
    </div>
  );
}`,
    explanation: "Using React.lazy() and Suspense creates a separate chunk for AdminDashboard. The code is only downloaded when showAdmin becomes true, significantly reducing initial bundle size and improving load time."
  }}
/>

### The Problem with Static Imports

The `AdminDashboard` component is being **statically imported**. This means that even if `showAdmin` is `false` and the component is not rendered, its JavaScript code is still included in the main application bundle and downloaded by the browser on initial load. For large components, this significantly increases the initial bundle size and slows down the application's startup time.

### The Fix: Dynamic Imports with `React.lazy` and `React.Suspense`

**Code splitting** is a technique that allows you to split your code into smaller chunks, which can then be loaded on demand. React provides built-in support for code splitting using `React.lazy` and `React.Suspense`.

*   **`React.lazy()`**: This function lets you render a dynamic import as a regular component. It takes a function that returns a Promise, which resolves to a module with a default export (your component).
*   **`React.Suspense`**: This component lets you display a fallback UI (like a loading spinner) while the lazy-loaded component is being downloaded.

Here is the corrected implementation:

```jsx


// FIX: Use React.lazy to dynamically import the AdminDashboard component.
// This tells Webpack (or your bundler) to create a separate chunk for it.
const AdminDashboard = React.lazy(() => {
  console.log('AdminDashboard module is being downloaded...');
  return new Promise(resolve => setTimeout(() => {
    // Simulate network delay for loading the chunk
    resolve(import('./AdminDashboard'));
  }, 1000));
});

// In a real project, AdminDashboard would be in its own file:
// AdminDashboard.jsx
// function AdminDashboard() {
//   console.log('AdminDashboard loaded!');
//   return (
//     <div style={{ border: '1px solid green', padding: '20px', margin: '10px' }}>
//       <h2>Admin Dashboard</h2>
//       <p>Welcome, Administrator!</p>
//       <p>Here are your sensitive admin controls.</p>
//     </div>
//   );
// }
// export default AdminDashboard;


export default function App() {
  const [showAdmin, setShowAdmin] = useState(false);

  return (
    <div style={{ padding: '20px' }}>
      <h1>Main Application</h1>
      <button onClick={() => setShowAdmin(!showAdmin)}>
        {showAdmin ? 'Hide Admin Dashboard' : 'Show Admin Dashboard'}
      </button>
      <hr />
      {/* FIX: Wrap the lazy-loaded component with Suspense */}
      <Suspense fallback={<div>Loading Admin Dashboard...</div>}>
        {showAdmin && <AdminDashboard />}
        {!showAdmin && <p>Admin Dashboard is currently hidden.</p>}
      </Suspense>
    </div>
  );
}
```
With this change, the code for `AdminDashboard` will only be downloaded by the browser when `showAdmin` becomes `true` and the component is actually rendered. This significantly improves the initial load time for users who don't need the dashboard.

---

### Question 37: This component is crashing the entire app. Implement an Error Boundary to gracefully handle the error.

**Type:** Practical | **Category:** Debugging & Tools

## The Scenario

You're working on a React application, and a specific component, `BuggyComponent`, is causing problems. Due to some unexpected data or a logical flaw, this component throws a JavaScript error during its render phase.

When this error occurs, it propagates up the component tree, causing the entire React application to unmount and display a blank screen. This is a terrible user experience and makes the application fragile.

## The Challenge

Your task is to implement an **Error Boundary**.

1.  Wrap the `BuggyComponent` with your Error Boundary.
2.  The Error Boundary should catch the error thrown by `BuggyComponent`.
3.  Instead of crashing the whole application, it should display a user-friendly fallback message (e.g., "Something went wrong.").
4.  It should also log the error to the console for debugging purposes.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// This component will unconditionally throw an error during render.
function BuggyComponent() {
  // Simulate an error, e.g., trying to access a property of undefined
  throw new Error('I crashed!');
  // return <h1>I am a buggy component!</h1>; // This line will never be reached
}

// The main application component
export default function App() {
  return (
    <div>
      <h1>Welcome to the App!</h1>
      <p>Below is a component that might crash.</p>
      <BuggyComponent /> {/* This component will cause the app to crash */}
      <p>This text should still be visible if the error is handled.</p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual and console-based test.
// 1. Run the code. You will see a blank screen (or a React error overlay)
//    and an error in the console. The text "Welcome to the App!" and
//    "This text should still be visible..." will not appear.
//
// After your fix:
// 1. The text "Welcome to the App!" and "This text should still be visible..."
//    should be visible.
// 2. In place of the BuggyComponent, you should see your fallback UI
//    (e.g., "Something went wrong.").
// 3. The error should be logged to the console by your Error Boundary,
//    but it should not crash the entire application.

console.log("Goal: Prevent the app from crashing and display a fallback UI.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: Containing Errors with Error Boundaries

<JuniorVsSenior
  juniorAnswer={{
    title: "No Error Handling (App Crashes)",
    code: `export default function App() {
  return (
    <div>
      <h1>Welcome to the App!</h1>
      <BuggyComponent /> {/* Crashes entire app */}
      <p>This text won't be visible</p>
    </div>
  );
}`,
    explanation: "Without error boundaries, any error in a component's render method bubbles up and unmounts the entire component tree, resulting in a blank screen and poor user experience."
  }}
  seniorAnswer={{
    title: "With Error Boundary Protection",
    code: `class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error("Error:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

export default function App() {
  return (
    <div>
      <h1>Welcome to the App!</h1>
      <ErrorBoundary>
        <BuggyComponent />
      </ErrorBoundary>
      <p>This text remains visible</p>
    </div>
  );
}`,
    explanation: "Error boundaries catch errors in their child component tree, log the error, and display a fallback UI instead of crashing the entire app. This provides graceful degradation and better user experience."
  }}
/>

### Understanding Error Boundaries

In React, errors in a component's render method, lifecycle methods, or constructors will "bubble up" and unmount the entire component tree, leading to a blank screen. This is where **Error Boundaries** come in.

An Error Boundary is a special type of React component that catches JavaScript errors anywhere in its child component tree, logs those errors, and displays a fallback UI. They are implemented as class components and use two specific lifecycle methods:

*   `static getDerivedStateFromError(error)`: This method is called after an error has been thrown by a descendant component. It should return a value to update state, allowing the next render to display a fallback UI.
*   `componentDidCatch(error, info)`: This method is called after an error has been thrown. It's used for side effects like logging the error information to an error reporting service.

### The Fix: Implementing a Class-Based Error Boundary

Here is the corrected implementation:

```jsx


// The component that will unconditionally throw an error
function BuggyComponent() {
  throw new Error('I crashed!');
}

// The Error Boundary component
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  // This method is called after an error has been thrown by a descendant component.
  // It updates state so the next render will show the fallback UI.
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  // This method is called after an error has been thrown.
  // It's used for side effects like logging the error.
  componentDidCatch(error, errorInfo) {
    console.error("Error caught by ErrorBoundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Fallback UI when an error occurs
      return <h1>Something went wrong.</h1>;
    }

    // Render children normally if no error
    return this.props.children;
  }
}

// The main application component, now using the Error Boundary
export default function App() {
  return (
    <div>
      <h1>Welcome to the App!</h1>
      <p>Below is a component that might crash.</p>
      <ErrorBoundary> {/* Wrap the potentially buggy component */}
        <BuggyComponent />
      </ErrorBoundary>
      <p>This text should still be visible if the error is handled.</p>
    </div>
  );
}
```
By wrapping `BuggyComponent` with `ErrorBoundary`, any error thrown within `BuggyComponent` (or its children) will be caught by `ErrorBoundary`. The `ErrorBoundary` will then render its fallback UI, preventing the entire application from crashing and providing a better user experience.


---

### Quick Check

**What types of errors are React Error Boundaries designed to catch?**

   A. Errors inside event handlers.
   B. Errors in asynchronous code (e.g., `setTimeout` callbacks).
-> C. **JavaScript errors in a component's render method, lifecycle methods, or constructors of its children.**
   D. Errors during server-side rendering.

<details>
<summary>See Answer</summary>

Error Boundaries are specifically designed to catch JavaScript errors during rendering, in lifecycle methods, and in constructors of the component tree below them. They do NOT catch errors in event handlers, asynchronous code, server-side rendering, or errors thrown in the Error Boundary itself.

</details>

---

### Question 38: This component performs expensive calculations unnecessarily. Optimize it with `useMemo`.

**Type:** Practical | **Category:** Performance

## The Scenario

A developer has a `ProductList` component that displays a list of products. It also calculates a `totalPrice` from the list.

They've noticed a performance issue: the `totalPrice` is re-calculated on every render of the `ProductList`, even when the `products` array itself hasn't changed (e.g., when a parent component re-renders due to an unrelated state change). This expensive calculation is causing the UI to feel sluggish.

## The Challenge

You've been given the `App` and `ProductList` components.

1.  Explain why the `totalPrice` calculation is happening unnecessarily on every render.
2.  Fix the `ProductList` component using the `useMemo` hook to prevent redundant calculations, ensuring `totalPrice` is only re-calculated when the `products` array actually changes.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Simulate an expensive calculation
const calculateTotalPrice = (products) => {
  console.log('Calculating total price...'); // This log indicates an expensive operation
  let total = 0;
  for (let i = 0; i < 10000000; i++) { // Simulate heavy computation
    total += Math.random();
  }
  return products.reduce((sum, product) => sum + product.price, 0);
};

function ProductList({ products }) {
  // THE BUG: This expensive calculation runs on every render,
  // even if 'products' hasn't changed.
  const totalPrice = calculateTotalPrice(products);

  return (
    <div style={{ border: '1px solid blue', padding: '10px', margin: '10px' }}>
      <h2>Product List</h2>
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name} - ${product.price.toFixed(2)}</li>
        ))}
      </ul>
      <h3>Total Price: ${totalPrice.toFixed(2)}</h3>
    </div>
  );
}

export default function App() {
  const [count, setCount] = useState(0); // State to force App re-renders

  const products = [
    { id: 1, name: 'Laptop', price: 1200.00 },
    { id: 2, name: 'Mouse', price: 25.00 },
    { id: 3, name: 'Keyboard', price: 75.00 },
  ];

  return (
    <div style={{ padding: '20px' }}>
      <h1>App Component</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Force App Re-render (Count: {count})
      </button>
      <hr />
      <ProductList products={products} />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. You will see "Calculating total price..."
//    on initial render.
// 3. Click the "Force App Re-render" button.
// 4. With the bug, you will see "Calculating total price..." logged again,
//    even though the 'products' array passed to ProductList hasn't changed.
//
// After your fix:
// 1. Clicking "Force App Re-render" should NOT cause
//    "Calculating total price..." to be logged.
// 2. The calculation should only happen on initial render.

console.log("Goal: Prevent unnecessary re-calculation of total price.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: Expensive Calculations and Re-renders


      <h3>Total: \${totalPrice.toFixed(2)}</h3>
    </div>
  );
}`,
    explanation: "Without useMemo, the expensive calculation runs on every render of ProductList, even when the products array hasn't changed. This wastes CPU cycles and can cause noticeable UI lag."
  }}
  rightAnswer={{
    title: "With useMemo Optimization",
    code: `function ProductList({ products }) {
  // Only re-runs when products array changes
  const totalPrice = useMemo(
    () => calculateTotalPrice(products),
    [products]
  );

  return (
    <div>
      <h3>Total: \${totalPrice.toFixed(2)}</h3>
    </div>
  );
}`,
    explanation: "useMemo memoizes the calculation result and only re-runs when the products dependency changes. This prevents unnecessary expensive computations, significantly improving performance."
  }}
/>

### Why the Problem Occurs

In React functional components, the entire component function body is re-executed on every render. If you have a computationally expensive operation (like `calculateTotalPrice`) directly inside your component, it will run every time the component re-renders, regardless of whether the inputs to that calculation have actually changed.

In our scenario:
1.  The `App` component re-renders when its `count` state changes.
2.  `App` then re-renders `ProductList`.
3.  Inside `ProductList`, `calculateTotalPrice(products)` is called again, even though the `products` array (which is a constant in `App`) hasn't changed its reference or content. This leads to the unnecessary re-calculation.

### The Fix: Memoizing Values with `useMemo`

The `useMemo` hook is designed to optimize performance by memoizing the result of an expensive calculation. It takes two arguments:
1.  A function that performs the calculation.
2.  A dependency array.

`useMemo` will only re-run the calculation function and return a new value if any of the dependencies in the array have changed. Otherwise, it returns the previously memoized value.

Here is the corrected implementation:

```jsx


const calculateTotalPrice = (products) => {
  console.log('Calculating total price...');
  let total = 0;
  for (let i = 0; i < 10000000; i++) {
    total += Math.random();
  }
  return products.reduce((sum, product) => sum + product.price, 0);
};

function ProductList({ products }) {
  // FIX: Memoize the totalPrice calculation using useMemo.
  // It will only re-run if the 'products' array reference changes.
  const totalPrice = useMemo(() => calculateTotalPrice(products), [products]);

  return (
    <div style={{ border: '1px solid blue', padding: '10px', margin: '10px' }}>
      <h2>Product List</h2>
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name} - ${product.price.toFixed(2)}</li>
        ))}
      </ul>
      <h3>Total Price: ${totalPrice.toFixed(2)}</h3>
    </div>
  );
}

export default function App() {
  const [count, setCount] = useState(0);

  // The 'products' array is a constant, so its reference never changes.
  const products = [
    { id: 1, name: 'Laptop', price: 1200.00 },
    { id: 2, name: 'Mouse', price: 25.00 },
    { id: 3, name: 'Keyboard', price: 75.00 },
  ];

  return (
    <div style={{ padding: '20px' }}>
      <h1>App Component</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Force App Re-render (Count: {count})
      </button>
      <hr />
      <ProductList products={products} />
    </div>
  );
}
```
With `useMemo`, the `calculateTotalPrice` function will only be executed when the `products` array (its dependency) changes. Since the `products` array in `App` is a constant and its reference never changes, `totalPrice` will only be calculated once on the initial render, preventing redundant expensive computations.


---

### Quick Check

**What is the primary purpose of the `useMemo` hook?**

   A. To memoize a function, preventing it from being re-created on every render.
   B. To perform side effects after every render of a component.
-> C. **To memoize the result of an expensive calculation, re-running it only when its dependencies change.**
   D. To create a new state variable that can be shared across components.

<details>
<summary>See Answer</summary>

useMemo is specifically designed to memoize the result of expensive calculations. It only re-runs the calculation when dependencies change, helping optimize performance by avoiding unnecessary computations on every render.

</details>

---

### Question 39: This component re-renders unnecessarily. How do you prevent it?

**Type:** Debugging | **Category:** Performance

## The Scenario

You're working on a React application and have an `ExpensiveComponent` that performs a lot of calculations or renders complex UI. To monitor its performance, you've added a `console.log` that fires every time it re-renders.

This `ExpensiveComponent` receives a `user` object as a prop from its parent, `App`. You've noticed that `ExpensiveComponent` re-renders every time the `App` component re-renders (e.g., when you click a button in `App`), even if the `user` object's *content* (`id` and `name`) hasn't logically changed. This leads to unnecessary work and can degrade application performance.

## The Challenge

You've been given the code for the `App` and `ExpensiveComponent`.

1.  Explain why `ExpensiveComponent` re-renders when the `App` component re-renders, even if the `user` data is logically the same.
2.  Optimize the `App` component and `ExpensiveComponent` to prevent these unnecessary re-renders.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// An "expensive" component that logs every re-render
function ExpensiveComponent({ user }) {
  console.log('Re-rendering ExpensiveComponent...');
  // Simulate some heavy computation
  let sum = 0;
  for (let i = 0; i < 10000000; i++) {
    sum += i;
  }
  return (
    <div style={{ border: '1px solid red', padding: '10px', margin: '10px' }}>
      <h3>Expensive Component</h3>
      <p>User ID: {user.id}</p>
      <p>User Name: {user.name}</p>
      <p>Simulated heavy computation result: {sum}</p>
    </div>
  );
}

export default function App() {
  const [count, setCount] = useState(0); // State to force App re-renders

  // THE BUG: The user object is created inline on every render of App.
  // Its reference changes even if id and name are logically the same.
  const user = { id: 1, name: 'Alice' };

  return (
    <div style={{ padding: '20px' }}>
      <h1>App Component</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Force App Re-render (Count: {count})
      </button>
      <hr />
      <ExpensiveComponent user={user} />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a console-based test.
// 1. Run the code.
// 2. Observe the console. You will see "Re-rendering ExpensiveComponent..."
//    on initial render.
// 3. Click the "Force App Re-render" button.
// 4. With the bug, you will see "Re-rendering ExpensiveComponent..." logged again,
//    even though the user's ID and Name haven't changed.
//
// After your fix, clicking "Force App Re-render" should NOT cause
// "Re-rendering ExpensiveComponent..." to be logged.

console.log("Goal: Prevent ExpensiveComponent from re-rendering when its props are logically unchanged.");
```
  </Fragment>
</SmartCodeRunner>

## The Explanation: React's Reconciliation and Prop References

;
}`,
    explanation: "Creating the user object inline means a new object reference is created on every render of App. React sees this as a new prop value, causing ExpensiveComponent to re-render even though the data is logically the same."
  }}
  rightAnswer={{
    title: "Memoized Object + Memoized Component",
    code: `const MemoizedExpensiveComponent = memo(ExpensiveComponent);

function App() {
  const [count, setCount] = useState(0);

  // Stable reference with useMemo
  const user = useMemo(() => ({
    id: 1,
    name: 'Alice'
  }), []);

  return <MemoizedExpensiveComponent user={user} />;
}`,
    explanation: "useMemo ensures the user object reference remains stable across renders. React.memo prevents re-renders when props haven't shallowly changed. Together, they eliminate unnecessary re-renders."
  }}
/>

### Why the Problem Occurs

In React, a component re-renders when its state or props change. When a parent component re-renders, by default, all of its child components also re-render. React then performs a "reconciliation" process, comparing the new virtual DOM with the previous one to determine the minimal actual DOM updates.

The problem in the buggy code is subtle but critical:
1.  The `App` component re-renders when its `count` state changes.
2.  Inside `App`'s render function, the `user` object (`{ id: 1, name: 'Alice' }`) is created **inline**.
3.  Even though the `id` and `name` properties are always `1` and `'Alice'`, creating the object inline means that on *every* render of `App`, a **new object reference** for `user` is generated.
4.  When `App` passes this new `user` object (with a new reference) as a prop to `ExpensiveComponent`, React sees that `ExpensiveComponent` has received a "new" prop.
5.  Therefore, `ExpensiveComponent` re-renders, even though the data it contains is logically the same.

### The Fix: Memoization with `useMemo` and `React.memo`

To prevent this unnecessary re-render, we need to ensure that the `user` prop's reference only changes when its actual content changes. We can achieve this using two memoization techniques:

1.  **`useMemo` (in the parent component)**: Memoize the `user` object itself. This ensures its reference remains stable across `App` re-renders as long as its dependencies (which are constant in this case) don't change.
2.  **`React.memo` (on the child component)**: Wrap `ExpensiveComponent` with `React.memo`. This is a Higher-Order Component that will prevent `ExpensiveComponent` from re-rendering if its props haven't shallowly changed.

Here is the corrected implementation:

```jsx


// Wrap the expensive component with React.memo
const MemoizedExpensiveComponent = memo(function ExpensiveComponent({ user }) {
  console.log('Re-rendering ExpensiveComponent...');
  let sum = 0;
  for (let i = 0; i < 10000000; i++) {
    sum += i;
  }
  return (
    <div style={{ border: '1px solid red', padding: '10px', margin: '10px' }}>
      <h3>Expensive Component</h3>
      <p>User ID: {user.id}</p>
      <p>User Name: {user.name}</p>
      <p>Simulated heavy computation result: {sum}</p>
    </div>
  );
});

export default function App() {
  const [count, setCount] = useState(0);

  // FIX: Memoize the user object using useMemo.
  // The user object's reference will now only change if its dependencies change.
  // In this case, the dependencies are an empty array, so it's created once.
  const user = useMemo(() => ({ id: 1, name: 'Alice' }), []);

  return (
    <div style={{ padding: '20px' }}>
      <h1>App Component</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Force App Re-render (Count: {count})
      </button>
      <hr />
      {/* Use the memoized version of the component */}
      <MemoizedExpensiveComponent user={user} />
    </div>
  );
}
```
With `useMemo` ensuring a stable `user` object reference and `React.memo` preventing re-renders when props are shallowly equal, `ExpensiveComponent` will now only re-render when its `user` prop *actually* changes (i.e., its reference changes, which `useMemo` now controls).


---

### Quick Check

**Which of the following is NOT a direct cause for a React functional component to re-render?**

   A. Its state changes.
   B. Its props change (by reference or value).
   C. Its parent component re-renders.
-> D. **A sibling component re-renders.**

<details>
<summary>See Answer</summary>

A component re-renders when: (1) its own state changes, (2) its props change, or (3) its parent re-renders (by default). Sibling components re-rendering does not directly cause a component to re-render - they are independent in the component tree unless they share a common parent that passes changing props.

</details>

---

### Question 40: This component's state logic is complex. Refactor it with `useReducer`.

**Type:** Refactoring | **Category:** State Management

A developer has built a `ShoppingCart` component. The cart state includes a list of `items` (each with a `productId`, `name`, `price`, and `quantity`) and a `totalPrice`.

They are currently using multiple `useState` calls to manage these pieces of state. Adding, removing, or updating item quantities involves several `setItems` and `setTotalPrice` calls, making the logic spread out, hard to follow, and prone to inconsistencies.

You've been given the `ShoppingCart` component. Your task is to refactor its state management to use the **`useReducer` hook**.

1.  Centralize the cart's state logic (adding, removing, updating quantity) into a single `cartReducer` function.
2.  Replace the multiple `useState` calls with a single `useReducer` call.
3.  Ensure the component's functionality remains identical.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Helper to calculate total price
const calculateTotal = (items) => {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
};

export default function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [totalPrice, setTotalPrice] = useState(0);

  // Update total price whenever items change
  useEffect(() => {
    setTotalPrice(calculateTotal(items));
  }, [items]);

  const addItem = (product) => {
    setItems(prevItems => {
      const existingItem = prevItems.find(item => item.productId === product.id);
      if (existingItem) {
        return prevItems.map(item =>
          item.productId === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prevItems, { productId: product.id, name: product.name, price: product.price, quantity: 1 }];
    });
  };

  const removeItem = (productId) => {
    setItems(prevItems => prevItems.filter(item => item.productId !== productId));
  };

  const updateQuantity = (productId, newQuantity) => {
    setItems(prevItems =>
      prevItems.map(item =>
        item.productId === productId
          ? { ...item, quantity: newQuantity }
          : item
      )
    );
  };

  return (
    <div style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h1>Shopping Cart</h1>
      <h2>Items</h2>
      {items.length === 0 ? (
        <p>Your cart is empty.</p>
      ) : (
        <ul>
          {items.map(item => (
            <li key={item.productId}>
              {item.name} (x{item.quantity}) - ${item.price * item.quantity}
              <button onClick={() => updateQuantity(item.productId, item.quantity + 1)}>+</button>
              <button onClick={() => updateQuantity(item.productId, item.quantity - 1)}>-</button>
              <button onClick={() => removeItem(item.productId)}>Remove</button>
            </li>
          ))}
        </ul>
      )}
      <h3>Total: ${totalPrice.toFixed(2)}</h3>
      <hr />
      <h2>Products</h2>
      <button onClick={() => addItem({ id: 1, name: 'Laptop', price: 1200 })}>Add Laptop</button>
      <button onClick={() => addItem({ id: 2, name: 'Mouse', price: 25 })}>Add Mouse</button>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual and functional test.
// 1. Run the code.
// 2. Click "Add Laptop", "Add Mouse", "Add Laptop" again.
// 3. Observe the items in the cart and the total price.
// 4. Click "+", "-", "Remove" buttons.
//
// After your refactor using useReducer:
// 1. The visual output and functionality of the cart should remain identical.
// 2. The state management logic should be centralized within a reducer function.

console.log("Goal: Refactor complex shopping cart state management to use useReducer.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState, useEffect } from 'react';

// Helper to calculate total price
const calculateTotal = (items) => {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
};

export default function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [totalPrice, setTotalPrice] = useState(0);

  // Update total price whenever items change
  useEffect(() => {
    setTotalPrice(calculateTotal(items));
  }, [items]);

  const addItem = (product) => {
    setItems(prevItems => {
      const existingItem = prevItems.find(item => item.productId === product.id);
      if (existingItem) {
        return prevItems.map(item =>
          item.productId === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prevItems, { productId: product.id, name: product.name, price: product.price, quantity: 1 }];
    });
  };

  const removeItem = (productId) => {
    setItems(prevItems => prevItems.filter(item => item.productId !== productId));
  };

  const updateQuantity = (productId, newQuantity) => {
    setItems(prevItems =>
      prevItems.map(item =>
        item.productId === productId
          ? { ...item, quantity: newQuantity }
          : item
      )
    );
  };

  return (
    <div style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h1>Shopping Cart</h1>
      <h2>Items</h2>
      {items.length === 0 ? (
        <p>Your cart is empty.</p>
      ) : (
        <ul>
          {items.map(item => (
            <li key={item.productId}>
              {item.name} (x{item.quantity}) - \${item.price * item.quantity}
              <button onClick={() => updateQuantity(item.productId, item.quantity + 1)}>+</button>
              <button onClick={() => updateQuantity(item.productId, item.quantity - 1)}>-</button>
              <button onClick={() => removeItem(item.productId)}>Remove</button>
            </li>
          ))}
        </ul>
      )}
      <h3>Total: \${totalPrice.toFixed(2)}</h3>
      <hr />
      <h2>Products</h2>
      <button onClick={() => addItem({ id: 1, name: 'Laptop', price: 1200 })}>Add Laptop</button>
      <button onClick={() => addItem({ id: 2, name: 'Mouse', price: 25 })}>Add Mouse</button>
    </div>
  );
}`}
  seniorAnswer={`import React, { useReducer, useEffect } from 'react';

// 1. Define the initial state for the cart
const initialCartState = {
  items: [],
  totalPrice: 0,
};

// Helper to calculate total price (can be reused in reducer)
const calculateTotal = (items) => {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
};

// 2. Define the reducer function
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const product = action.payload;
      const existingItem = state.items.find(item => item.productId === product.id);
      let updatedItems;

      if (existingItem) {
        updatedItems = state.items.map(item =>
          item.productId === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        updatedItems = [...state.items, { productId: product.id, name: product.name, price: product.price, quantity: 1 }];
      }
      return {
        ...state,
        items: updatedItems,
        totalPrice: calculateTotal(updatedItems),
      };
    }
    case 'REMOVE_ITEM': {
      const productId = action.payload;
      const updatedItems = state.items.filter(item => item.productId !== productId);
      return {
        ...state,
        items: updatedItems,
        totalPrice: calculateTotal(updatedItems),
      };
    }
    case 'UPDATE_QUANTITY': {
      const { productId, newQuantity } = action.payload;
      const updatedItems = state.items.map(item =>
        item.productId === productId
          ? { ...item, quantity: newQuantity }
          : item
      );
      return {
        ...state,
        items: updatedItems,
        totalPrice: calculateTotal(updatedItems),
      };
    }
    default:
      throw new Error(\`Unhandled action type: \${action.type}\`);
  }
}

export default function ShoppingCart() {
  // 3. Replace useState with useReducer
  const [cartState, dispatch] = useReducer(cartReducer, initialCartState);
  const { items, totalPrice } = cartState;

  // Action dispatchers
  const addItem = (product) => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  };

  const removeItem = (productId) => {
    dispatch({ type: 'REMOVE_ITEM', payload: productId });
  };

  const updateQuantity = (productId, newQuantity) => {
    // Prevent quantity from going below 1, or remove if 0
    if (newQuantity <= 0) {
      dispatch({ type: 'REMOVE_ITEM', payload: productId });
    } else {
      dispatch({ type: 'UPDATE_QUANTITY', payload: { productId, newQuantity } });
    }
  };

  return (
    <div style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h1>Shopping Cart</h1>
      <h2>Items</h2>
      {items.length === 0 ? (
        <p>Your cart is empty.</p>
      ) : (
        <ul>
          {items.map(item => (
            <li key={item.productId}>
              {item.name} (x{item.quantity}) - \${item.price * item.quantity}
              <button onClick={() => updateQuantity(item.productId, item.quantity + 1)}>+</button>
              <button onClick={() => updateQuantity(item.productId, item.quantity - 1)}>-</button>
              <button onClick={() => removeItem(item.productId)}>Remove</button>
            </li>
          ))}
        </ul>
      )}
      <h3>Total: \${totalPrice.toFixed(2)}</h3>
      <hr />
      <h2>Products</h2>
      <button onClick={() => addItem({ id: 1, name: 'Laptop', price: 1200 })}>Add Laptop</button>
      <button onClick={() => addItem({ id: 2, name: 'Mouse', price: 25 })}>Add Mouse</button>
    </div>
  );
}`}
  juniorExplanation="The original ShoppingCart component uses multiple useState calls (items and totalPrice) and useEffect to keep totalPrice in sync. This approach can become cumbersome and error-prone when state logic is complex, involves multiple related pieces of state, or when the next state depends on the previous state. The logic for updating the cart is spread across several functions."
  seniorExplanation="The useReducer hook is an alternative to useState that is particularly well-suited for managing complex state logic. It's inspired by the Redux pattern and centralizes all state transitions in a single reducer function. Here's how useReducer works: 1) reducer(state, action): A pure function that takes the current state and an action object, and returns the newState. 2) initialState: The initial value of your state. 3) dispatch function: You call dispatch({ type: 'ACTION_TYPE', payload: ... }) to trigger state updates. This approach makes state updates more predictable, testable, and easier to reason about. By using useReducer, all the logic for updating the cart's state is now centralized in the cartReducer function. This makes the ShoppingCart component itself cleaner, and the state transitions are more explicit and easier to test."
  client:load
/>

---

### Question 41: These components duplicate logic. Refactor them using a Higher-Order Component.

**Type:** Refactoring | **Category:** Component Patterns

You're reviewing a codebase and notice a common pattern: several components need to fetch data from an API and display a loading state while the data is being retrieved.

Specifically, you have `UserDisplay` and `ProductDisplay` components. Both implement their own `useState` for data and loading, and their own `useEffect` for fetching. This leads to duplicated code and makes maintenance harder.

Your task is to refactor these components. Create a **Higher-Order Component (HOC)** called `withDataFetching` that encapsulates the common data fetching and loading state logic.

Apply this HOC to both `UserDisplay` and `ProductDisplay` so that they become purely presentational components, receiving their `data` and `loading` props from the HOC.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Mock API calls
const fetchUsers = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]);
    }, 1000);
  });
};

const fetchProducts = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{ id: 101, name: 'Laptop' }, { id: 102, name: 'Mouse' }]);
    }, 1500);
  });
};

// Component 1: Duplicates data fetching logic
function UserDisplay() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsers().then(data => {
      setUsers(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <div>Loading users...</div>;

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Component 2: Duplicates data fetching logic
function ProductDisplay() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchProducts().then(data => {
      setProducts(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <div>Loading products...</div>;

  return (
    <div>
      <h2>Products</h2>
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <UserDisplay />
      <hr />
      <ProductDisplay />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. You will see "Loading users..." and "Loading products..."
//    followed by the respective lists.
//
// After your refactor using an HOC:
// 1. The visual output should remain identical.
// 2. The `UserDisplay` and `ProductDisplay` components should no longer
//    contain the data fetching and loading state logic directly.

console.log("Goal: Refactor duplicated data fetching logic into a reusable HOC.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState, useEffect } from 'react';

// Mock API calls
const fetchUsers = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]);
    }, 1000);
  });
};

const fetchProducts = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{ id: 101, name: 'Laptop' }, { id: 102, name: 'Mouse' }]);
    }, 1500);
  });
};

// Component 1: Duplicates data fetching logic
function UserDisplay() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsers().then(data => {
      setUsers(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <div>Loading users...</div>;

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Component 2: Duplicates data fetching logic
function ProductDisplay() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchProducts().then(data => {
      setProducts(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <div>Loading products...</div>;

  return (
    <div>
      <h2>Products</h2>
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <UserDisplay />
      <hr />
      <ProductDisplay />
    </div>
  );
}`}
  seniorAnswer={`import React, { useState, useEffect } from 'react';

// Mock API calls
const fetchUsers = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]);
    }, 1000);
  });
};

const fetchProducts = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{ id: 101, name: 'Laptop' }, { id: 102, name: 'Mouse' }]);
    }, 1500);
  });
};

// The Higher-Order Component (HOC)
function withDataFetching(WrappedComponent, fetchFunction) {
  return function WithDataFetching(props) {
    const [data, setData] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      fetchFunction().then(fetchedData => {
        setData(fetchedData);
        setLoading(false);
      });
    }, []);

    // Pass data and loading as props to the wrapped component
    return <WrappedComponent {...props} data={data} loading={loading} />;
  };
}

// Refactored UserDisplay - now purely presentational
function UserDisplay({ data, loading }) {
  if (loading) return <div>Loading users...</div>;

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {data.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Refactored ProductDisplay - now purely presentational
function ProductDisplay({ data, loading }) {
  if (loading) return <div>Loading products...</div>;

  return (
    <div>
      <h2>Products</h2>
      <ul>
        {data.map(product => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Apply the HOC to enhance the components
const EnhancedUserDisplay = withDataFetching(UserDisplay, fetchUsers);
const EnhancedProductDisplay = withDataFetching(ProductDisplay, fetchProducts);

export default function App() {
  return (
    <div>
      <EnhancedUserDisplay />
      <hr />
      <EnhancedProductDisplay />
    </div>
  );
}`}
  juniorExplanation="The original code suffers from code duplication. The logic for managing loading state and fetching data is repeated in both UserDisplay and ProductDisplay. This makes the codebase harder to maintain and extend."
  seniorExplanation="A Higher-Order Component (HOC) is a function that takes a component and returns a new component with additional props or behavior. It's a powerful pattern for reusing component logic. We can create a withDataFetching HOC that will: 1) Take a WrappedComponent and a fetchFunction (e.g., fetchUsers, fetchProducts) as arguments. 2) Manage its own data and loading state. 3) Call the fetchFunction in a useEffect hook. 4) Pass the data and loading state as props to the WrappedComponent. This refactoring significantly reduces code duplication. Both UserDisplay and ProductDisplay are now simpler, 'dumb' components that only care about rendering. The data fetching logic is centralized in the withDataFetching HOC. Note: While HOCs are a valid pattern, custom Hooks are generally the preferred way to share logic between functional components in modern React, as they often lead to cleaner code without the 'wrapper hell' sometimes associated with HOCs."
  client:load
/>

---

### Question 42: These components duplicate logic. Refactor them using the Render Props pattern.

**Type:** Refactoring | **Category:** Component Patterns

You're working on a React application where you need to track the mouse position on the screen. You have two components: `MousePositionLogger` (which displays the coordinates) and `MouseEmojiTracker` (which displays an emoji at the mouse position).

Both components currently implement their own mouse tracking logic, including adding and removing event listeners and managing `x` and `y` state. This leads to duplicated code and makes it harder to maintain or add new components that also need mouse position.

Your task is to refactor these components. Create a single `MouseTracker` component that encapsulates the mouse tracking logic. Use the **Render Props pattern** to share the mouse position (`x`, `y`) with its children.

After refactoring, `MousePositionLogger` and `MouseEmojiTracker` should become simpler, purely presentational components that receive the mouse coordinates as props from the `MouseTracker`.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Component 1: Duplicates mouse tracking logic
function MousePositionLogger() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = (event) => {
    setX(event.clientX);
    setY(event.clientY);
  };

  useEffect(() => {
    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);

  return (
    <div style={{ border: '1px solid blue', padding: '20px', margin: '10px' }}>
      <h2>Mouse Position Logger</h2>
      <p>X: {x}, Y: {y}</p>
    </div>
  );
}

// Component 2: Duplicates mouse tracking logic
function MouseEmojiTracker() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = (event) => {
    setX(event.clientX);
    setY(event.clientY);
  };

  useEffect(() => {
    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);

  return (
    <div style={{
      position: 'absolute',
      left: x,
      top: y,
      fontSize: '30px',
      pointerEvents: 'none', // So it doesn't interfere with mouse events
    }}>
      🐭
    </div>
  );
}

export default function App() {
  return (
    <div style={{ height: '100vh', position: 'relative' }}>
      <MousePositionLogger />
      <MouseEmojiTracker />
      <p style={{ position: 'absolute', bottom: '10px', left: '10px' }}>
        Move your mouse around!
      </p>
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. You will see the mouse coordinates being logged
//    and an emoji following your mouse.
//
// After your refactor using the Render Props pattern:
// 1. The visual output should remain identical.
// 2. The `MousePositionLogger` and `MouseEmojiTracker` components should
//    no longer contain the mouse tracking logic directly.

console.log("Goal: Refactor duplicated mouse tracking logic into a reusable component using Render Props.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState, useEffect } from 'react';

// Component 1: Duplicates mouse tracking logic
function MousePositionLogger() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = (event) => {
    setX(event.clientX);
    setY(event.clientY);
  };

  useEffect(() => {
    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);

  return (
    <div style={{ border: '1px solid blue', padding: '20px', margin: '10px' }}>
      <h2>Mouse Position Logger</h2>
      <p>X: {x}, Y: {y}</p>
    </div>
  );
}

// Component 2: Duplicates mouse tracking logic
function MouseEmojiTracker() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const handleMouseMove = (event) => {
    setX(event.clientX);
    setY(event.clientY);
  };

  useEffect(() => {
    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);

  return (
    <div style={{
      position: 'absolute',
      left: x,
      top: y,
      fontSize: '30px',
      pointerEvents: 'none',
    }}>
      🐭
    </div>
  );
}

export default function App() {
  return (
    <div style={{ height: '100vh', position: 'relative' }}>
      <MousePositionLogger />
      <MouseEmojiTracker />
      <p style={{ position: 'absolute', bottom: '10px', left: '10px' }}>
        Move your mouse around!
      </p>
    </div>
  );
}`}
  seniorAnswer={`import React, { useState, useEffect } from 'react';

// The MouseTracker component using the Render Props pattern
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  };

  componentDidMount() {
    window.addEventListener('mousemove', this.handleMouseMove);
  }

  componentWillUnmount() {
    window.removeEventListener('mousemove', this.handleMouseMove);
  }

  render() {
    // The render prop is a function that receives the state (x, y)
    // and returns the JSX to be rendered.
    return this.props.render(this.state);
  }
}

// Refactored MousePositionLogger - now purely presentational
function MousePositionLogger({ x, y }) {
  return (
    <div style={{ border: '1px solid blue', padding: '20px', margin: '10px' }}>
      <h2>Mouse Position Logger</h2>
      <p>X: {x}, Y: {y}</p>
    </div>
  );
}

// Refactored MouseEmojiTracker - now purely presentational
function MouseEmojiTracker({ x, y }) {
  return (
    <div style={{
      position: 'absolute',
      left: x,
      top: y,
      fontSize: '30px',
      pointerEvents: 'none',
    }}>
      🐭
    </div>
  );
}

export default function App() {
  return (
    <div style={{ height: '100vh', position: 'relative' }}>
      {/* Use MouseTracker with a render prop for MousePositionLogger */}
      <MouseTracker
        render={({ x, y }) => (
          <MousePositionLogger x={x} y={y} />
        )}
      />

      {/* Use MouseTracker with a render prop for MouseEmojiTracker */}
      <MouseTracker
        render={({ x, y }) => (
          <MouseEmojiTracker x={x} y={y} />
        )}
      />
      <p style={{ position: 'absolute', bottom: '10px', left: '10px' }}>
        Move your mouse around!
      </p>
    </div>
  );
}`}
  juniorExplanation="The original code suffers from code duplication. The logic for tracking mouse position (managing x and y state, adding/removing event listeners) is repeated in both MousePositionLogger and MouseEmojiTracker. This makes the codebase harder to maintain and extend."
  seniorExplanation="The Render Props pattern is a technique for sharing code between React components using a prop whose value is a function. This function is called a 'render prop' because it's used to determine what to render. The component with the render prop (e.g., MouseTracker) encapsulates the shared behavior, and the consumer components (e.g., MousePositionLogger, MouseEmojiTracker) use the render prop to define their specific UI based on that behavior. This refactoring significantly reduces code duplication. The mouse tracking logic is now centralized in the MouseTracker component. MousePositionLogger and MouseEmojiTracker are simpler, 'dumb' components that only care about rendering the UI based on the x and y coordinates they receive. Note: While Render Props are a valid pattern, custom Hooks are generally the preferred way to share logic between functional components in modern React, as they often lead to cleaner code without the nesting sometimes associated with Render Props."
  client:load
/>

---

### Question 43: These components duplicate stateful logic. Refactor them using a custom Hook.

**Type:** Refactoring | **Category:** Hooks & Patterns

You're building a React application and notice a pattern: you have several components that need to manage a simple boolean `isOn` state and a function to `toggle` that state.

For example, you have a `Toggle` component that turns something on/off, and you're about to build a `VisibilityToggle` component that also needs the exact same `isOn` state and `toggle` function. You're about to copy-paste the `useState` and `toggle` logic, which is a clear sign of code duplication.

Your task is to refactor the application to eliminate this duplicated stateful logic.

1.  Create a **custom Hook** called `useToggle` that encapsulates the `isOn` state and the `toggle` function.
2.  Apply this `useToggle` Hook to both the `Toggle` component and the `VisibilityToggle` component.

The functionality of both components should remain identical, but the underlying code should be cleaner and more reusable.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Component 1: Manages its own boolean state
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  const handleToggle = () => {
    setIsOn(prevIsOn => !prevIsOn);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <h2>Toggle Component</h2>
      <p>Status: {isOn ? 'ON' : 'OFF'}</p>
      <button onClick={handleToggle}>Toggle</button>
    </div>
  );
}

// Component 2: Manages its own boolean state (duplicates logic from Toggle)
function VisibilityToggle() {
  const [isVisible, setIsVisible] = useState(false); // Same logic as isOn

  const handleToggleVisibility = () => {
    setIsVisible(prevIsVisible => !prevIsVisible);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <h2>Visibility Toggle Component</h2>
      <p>Content is: {isVisible ? 'Visible' : 'Hidden'}</p>
      <button onClick={handleToggleVisibility}>Toggle Visibility</button>
      {isVisible && <p>This content is now visible!</p>}
    </div>
  );
}

export default function App() {
  return (
    <div>
      <Toggle />
      <VisibilityToggle />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. Both "Toggle" and "Visibility Toggle" components
//    should function independently, turning ON/OFF and Visible/Hidden.
//
// After your refactor using a custom Hook:
// 1. The visual output and functionality of both components should remain identical.
// 2. The `Toggle` and `VisibilityToggle` components should no longer contain
//    the `useState` and `toggle` logic directly.

console.log("Goal: Refactor duplicated boolean toggle logic into a reusable custom Hook.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState } from 'react';

// Component 1: Manages its own boolean state
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  const handleToggle = () => {
    setIsOn(prevIsOn => !prevIsOn);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <h2>Toggle Component</h2>
      <p>Status: {isOn ? 'ON' : 'OFF'}</p>
      <button onClick={handleToggle}>Toggle</button>
    </div>
  );
}

// Component 2: Manages its own boolean state (duplicates logic from Toggle)
function VisibilityToggle() {
  const [isVisible, setIsVisible] = useState(false); // Same logic as isOn

  const handleToggleVisibility = () => {
    setIsVisible(prevIsVisible => !prevIsVisible);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <h2>Visibility Toggle Component</h2>
      <p>Content is: {isVisible ? 'Visible' : 'Hidden'}</p>
      <button onClick={handleToggleVisibility}>Toggle Visibility</button>
      {isVisible && <p>This content is now visible!</p>}
    </div>
  );
}

export default function App() {
  return (
    <div>
      <Toggle />
      <VisibilityToggle />
    </div>
  );
}`}
  seniorAnswer={`import React, { useState } from 'react';

// 1. Create the custom Hook: useToggle
function useToggle(initialValue = false) {
  const [isOn, setIsOn] = useState(initialValue);

  const toggle = () => {
    setIsOn(prevIsOn => !prevIsOn);
  };

  return [isOn, toggle]; // Return the state and the toggler function
}

// Component 1: Now uses the custom Hook
function Toggle() {
  const [isOn, toggle] = useToggle(false); // Use the custom Hook

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <h2>Toggle Component</h2>
      <p>Status: {isOn ? 'ON' : 'OFF'}</p>
      <button onClick={toggle}>Toggle</button>
    </div>
  );
}

// Component 2: Now uses the custom Hook
function VisibilityToggle() {
  const [isVisible, toggleVisibility] = useToggle(false); // Use the custom Hook

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '10px' }}>
      <h2>Visibility Toggle Component</h2>
      <p>Content is: {isVisible ? 'Visible' : 'Hidden'}</p>
      <button onClick={toggleVisibility}>Toggle Visibility</button>
      {isVisible && <p>This content is now visible!</p>}
    </div>
  );
}

export default function App() {
  return (
    <div>
      <Toggle />
      <VisibilityToggle />
    </div>
  );
}`}
  juniorExplanation="The original code suffers from code duplication. The logic for managing a boolean state (isOn / isVisible) and a function to toggle it (handleToggle / handleToggleVisibility) is repeated in both Toggle and VisibilityToggle. This makes the codebase harder to maintain and extend."
  seniorExplanation="Custom Hooks are a powerful feature in React that allow you to extract stateful logic from components, making it reusable across different components. A custom Hook is simply a JavaScript function whose name starts with 'use' and that calls other Hooks. We can create a useToggle custom Hook that encapsulates the useState for the boolean state and the function to toggle it. This refactoring significantly reduces code duplication. The boolean toggle logic is now centralized in the useToggle custom Hook. Both Toggle and VisibilityToggle are simpler, cleaner components that leverage this reusable logic."
  client:load
/>

---

### Question 44: This component manages loading state imperatively. Refactor it to use React Suspense.

**Type:** Refactoring | **Category:** Component Patterns

A developer has a `UserProfile` component that fetches user data from an API. Currently, they manage the loading state manually using `useState` and `useEffect`, displaying a "Loading..." message while waiting for the data.

They want to improve the user experience and simplify the code by using **React Suspense** to declaratively handle the loading state.

You've been given the `UserProfile` component and a mock API.

1.  Refactor the application to use `React.Suspense` to declaratively handle the loading state for the `UserProfile` component.
2.  The `UserProfile` component should "suspend" its rendering until the data is available.
3.  The `App` component should display a `fallback` UI while `UserProfile` is waiting for data.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


// Mock API call that simulates fetching user data
const fetchUserData = () => {
  console.log('Fetching user data...');
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id: 1, name: 'Alice', email: 'alice@example.com' });
    }, 2000); // Simulate network delay
  });
};

// UserProfile component with imperative loading state management
function UserProfile() {
  const [userData, setUserData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetchUserData().then(data => {
      setUserData(data);
      setLoading(false);
    });
  }, []);

  if (loading) {
    return <div>Loading user profile...</div>;
  }

  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', margin: '10px' }}>
      <h2>User Profile</h2>
      <p>ID: {userData.id}</p>
      <p>Name: {userData.name}</p>
      <p>Email: {userData.email}</p>
    </div>
  );
}

export default function App() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Application</h1>
      <UserProfile />
    </div>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual test.
// 1. Run the code. You will see "Loading user profile..." for 2 seconds,
//    then the user data.
//
// After your fix using React Suspense:
// 1. The initial display should show the fallback UI (e.g., "Loading data with Suspense...").
// 2. After 2 seconds, the UserProfile should appear.
// 3. The UserProfile component itself should no longer contain explicit
//    loading state management.

console.log("Goal: Refactor to use React Suspense for declarative loading states.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useState, useEffect } from 'react';

// Mock API call that simulates fetching user data
const fetchUserData = () => {
  console.log('Fetching user data...');
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id: 1, name: 'Alice', email: 'alice@example.com' });
    }, 2000); // Simulate network delay
  });
};

// UserProfile component with imperative loading state management
function UserProfile() {
  const [userData, setUserData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetchUserData().then(data => {
      setUserData(data);
      setLoading(false);
    });
  }, []);

  if (loading) {
    return <div>Loading user profile...</div>;
  }

  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', margin: '10px' }}>
      <h2>User Profile</h2>
      <p>ID: {userData.id}</p>
      <p>Name: {userData.name}</p>
      <p>Email: {userData.email}</p>
    </div>
  );
}

export default function App() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Application</h1>
      <UserProfile />
    </div>
  );
}`}
  seniorAnswer={`import React, { useState, useEffect, Suspense } from 'react';

// 1. Create a Suspense-compatible resource utility
// This is a simplified example. In a real app, you'd use a library like
// React Query, SWR, or Relay, which provide Suspense integration.
function createResource(promise) {
  let status = 'pending';
  let result;
  let suspender = promise.then(
    r => {
      status = 'success';
      result = r;
    },
    e => {
      status = 'error';
      result = e;
    }
  );

  return {
    read() {
      if (status === 'pending') {
        throw suspender; // Suspense catches this promise
      } else if (status === 'error') {
        throw result;
      } else if (status === 'success') {
        return result;
      }
    }
  };
}

// Mock API call that simulates fetching user data
const fetchUserData = () => {
  console.log('Fetching user data...');
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id: 1, name: 'Alice', email: 'alice@example.com' });
    }, 2000); // Simulate network delay
  });
};

// Create the resource once outside the component
const userResource = createResource(fetchUserData());

// UserProfile component now uses the resource directly
function UserProfile() {
  // 2. Read data from the resource. This will suspend if data is not ready.
  const userData = userResource.read();
  console.log('UserProfile rendered with data:', userData);

  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', margin: '10px' }}>
      <h2>User Profile</h2>
      <p>ID: {userData.id}</p>
      <p>Name: {userData.name}</p>
      <p>Email: {userData.email}</p>
    </div>
  );
}

export default function App() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Application</h1>
      {/* 3. Wrap UserProfile with Suspense and provide a fallback */}
      <Suspense fallback={<div>Loading data with Suspense...</div>}>
        <UserProfile />
      </Suspense>
    </div>
  );
}`}
  juniorExplanation="The original code manages the loading state imperatively. The UserProfile component explicitly uses useState for loading and useEffect to set loading to true before fetching and false after. This works, but it can become cumbersome in complex applications with many data dependencies."
  seniorExplanation="React Suspense provides a declarative way to handle loading states. It allows components to 'suspend' their rendering while they are waiting for some asynchronous operation (like data fetching or code splitting) to complete. When a component suspends, React looks up the component tree for the nearest <Suspense> boundary and renders its fallback prop. To integrate data fetching with Suspense, you need a Suspense-compatible data fetching solution. This typically involves a utility that wraps a Promise and 'throws' it when it's pending, allowing Suspense to catch it. With Suspense, the UserProfile component becomes simpler and more focused on rendering. It doesn't need to manage loading state explicitly. When userResource.read() is called and the data isn't ready, it 'throws' the promise, and React's Suspense boundary catches it, displaying the fallback UI until the promise resolves."
  client:load
/>

---

### Question 45: This form uses an uncontrolled input. Refactor it to be a controlled component.

**Type:** Refactoring | **Category:** Forms & Input

A developer has built a simple login form with an input field for the username and a submit button. They've implemented the username input as an **uncontrolled component**, meaning its value is managed by the DOM itself, and they access it using a `ref` only when the form is submitted.

Now, they want to add two new features:
1.  **Real-time validation**: Display an error message if the username input is empty.
2.  **Dynamic button state**: Disable the submit button if the username input is empty.

They are struggling to implement these features effectively because they can't easily get the input's value in real-time.

You've been given the `LoginForm` component. Your task is to refactor the username input from an uncontrolled component to a **controlled component**.

After refactoring, implement the real-time validation and dynamic button state:
*   Show an error message "Username cannot be empty" if the input is empty.
*   Disable the "Submit" button if the input is empty.

<SmartCodeRunner mode="run">
  <Fragment slot="code">
```jsx


export default function LoginForm() {
  const usernameInputRef = useRef(null);

  const handleSubmit = (event) => {
    event.preventDefault();
    const username = usernameInputRef.current.value;
    if (username.trim() === '') {
      alert('Username cannot be empty!');
      return;
    }
    alert(`Submitting username: ${username}`);
    // In a real app, you'd send this to an API
  };

  return (
    <form onSubmit={handleSubmit} style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h2>Login</h2>
      <label>
        Username:
        {/* THE BUG: This is an uncontrolled input.
            Its value is managed by the DOM, not React state. */}
        <input type="text" ref={usernameInputRef} />
      </label>
      {/* No real-time validation or dynamic button state */}
      <button type="submit" style={{ marginLeft: '10px' }}>
        Submit
      </button>
    </form>
  );
}
```
  </Fragment>
  <Fragment slot="output">
```jsx
// This is a visual and functional test.
// 1. Run the code.
// 2. Observe the input. It has no real-time feedback.
// 3. The "Submit" button is always enabled.
// 4. Try to submit an empty username. An alert will appear.
//
// After your fix:
// 1. When the input is empty, an error message "Username cannot be empty"
//    should appear below the input.
// 2. The "Submit" button should be disabled when the input is empty.
// 3. As you type, the error message should disappear, and the button should enable.

console.log("Goal: Implement real-time validation and dynamic button state by making the input controlled.");
```
  </Fragment>
</SmartCodeRunner>

<JuniorVsSenior
  juniorAnswer={`import React, { useRef } from 'react';

export default function LoginForm() {
  const usernameInputRef = useRef(null);

  const handleSubmit = (event) => {
    event.preventDefault();
    const username = usernameInputRef.current.value;
    if (username.trim() === '') {
      alert('Username cannot be empty!');
      return;
    }
    alert(\`Submitting username: \${username}\`);
    // In a real app, you'd send this to an API
  };

  return (
    <form onSubmit={handleSubmit} style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h2>Login</h2>
      <label>
        Username:
        {/* THE BUG: This is an uncontrolled input.
            Its value is managed by the DOM, not React state. */}
        <input type="text" ref={usernameInputRef} />
      </label>
      {/* No real-time validation or dynamic button state */}
      <button type="submit" style={{ marginLeft: '10px' }}>
        Submit
      </button>
    </form>
  );
}`}
  seniorAnswer={`import React, { useState } from 'react';

export default function LoginForm() {
  // FIX: Use useState to manage the input's value
  const [username, setUsername] = useState('');
  const [error, setError] = useState('');

  const handleUsernameChange = (event) => {
    const newValue = event.target.value;
    setUsername(newValue);
    // Real-time validation
    if (newValue.trim() === '') {
      setError('Username cannot be empty');
    } else {
      setError('');
    }
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    if (username.trim() === '') {
      setError('Username cannot be empty'); // Final check on submit
      return;
    }
    alert(\`Submitting username: \${username}\`);
  };

  // Determine if the button should be disabled
  const isSubmitDisabled = username.trim() === '';

  return (
    <form onSubmit={handleSubmit} style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h2>Login</h2>
      <label>
        Username:
        {/* FIX: Bind value to state and provide onChange handler */}
        <input
          type="text"
          value={username}
          onChange={handleUsernameChange}
        />
      </label>
      {/* Display real-time error message */}
      {error && <p style={{ color: 'red', fontSize: '0.8em' }}>{error}</p>}
      <button type="submit" style={{ marginLeft: '10px' }} disabled={isSubmitDisabled}>
        Submit
      </button>
    </form>
  );
}`}
  juniorExplanation="Uncontrolled Components: Form data is handled by the DOM itself. You use a ref to get their current value when you need it (e.g., on form submission). This is similar to how traditional HTML forms work. They are simpler for very basic use cases but offer less control."
  seniorExplanation="Controlled Components: Form data is handled by React state. The input's value is always driven by React state, and any changes to the input are managed through event handlers (typically onChange). React state becomes the 'single source of truth' for the input's value. To make the input a controlled component, you need to: 1) Declare a state variable (e.g., username) using useState. 2) Bind the input's value prop to this state variable. 3) Provide an onChange handler that updates the state variable with the input's new value. Once the input is controlled, you can easily implement real-time validation and dynamic button states by checking the username state variable."
  client:load
/>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

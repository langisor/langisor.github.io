# Mastering React Custom Hooks

> Original Article: [https://devdreaming.com/blogs/react-hooks-fundamental-guide-for-beginners](https://devdreaming.com/blogs/react-hooks-fundamental-guide-for-beginners)

<!-- toc -->

- [Mastering React Custom Hooks](#mastering-react-custom-hooks)
  - [History](#history)
  - [What Are React Hooks?](#what-are-react-hooks)
    - [Why are Hooks important for beginners?](#why-are-hooks-important-for-beginners)
    - [The Two Golden Rules of Hooks](#the-two-golden-rules-of-hooks)
  - [The Core Hooks: useState and useEffect](#the-core-hooks-usestate-and-useeffect)
    - [useState: Managing Component State](#usestate-managing-component-state)
      - [useState with Complex State](#usestate-with-complex-state)
      - [Functional Updates in useState](#functional-updates-in-usestate)
    - [useEffect: Managing Side Effects](#useeffect-managing-side-effects)
      - [The Dependency Array](#the-dependency-array)
      - [Common useEffect Patterns](#common-useeffect-patterns)
  - [Additional Core React Hooks](#additional-core-react-hooks)
    - [1. useContext: Managing Global State](#1-usecontext-managing-global-state)
    - [2. useReducer: Complex State Logic](#2-usereducer-complex-state-logic)
    - [3. useRef: Persisting Values Between Renders](#3-useref-persisting-values-between-renders)
  - [Performance Optimization Hooks](#performance-optimization-hooks)
    - [useMemo: Memoizing Expensive Calculations](#usememo-memoizing-expensive-calculations)
    - [useCallback: Memoizing Functions](#usecallback-memoizing-functions)
  - [Building Custom Hooks](#building-custom-hooks)
    - [More Custom Hook Examples](#more-custom-hook-examples)
      - [1. useFetch: Simplified Data Fetching](#1-usefetch-simplified-data-fetching)
      - [2. useLocalStorage: Persistent State](#2-uselocalstorage-persistent-state)
  - [React Hooks Best Practices in 2025](#react-hooks-best-practices-in-2025)
    - [1. Rules of Hooks](#1-rules-of-hooks)
    - [3. Create Custom Hooks for Reusable Logic](#3-create-custom-hooks-for-reusable-logic)
    - [4. Optimize Expensive Calculations with useMemo and useCallback](#4-optimize-expensive-calculations-with-usememo-and-usecallback)
    - [5. Manage Complex State with useReducer](#5-manage-complex-state-with-usereducer)
    - [6. Separate Data Fetching from UI](#6-separate-data-fetching-from-ui)
  - [Advanced Hook Patterns](#advanced-hook-patterns)
    - [Composition of Hooks](#composition-of-hooks)
    - [Context + Reducer Pattern](#context--reducer-pattern)
  - [React Hooks in 2025: What's New?](#react-hooks-in-2025-whats-new)
    - [Improved Performance](#improved-performance)
    - [Enhanced DevTools Support](#enhanced-devtools-support)
    - [Integration with Concurrent Mode](#integration-with-concurrent-mode)
    - [TypeScript Improvements](#typescript-improvements)
  - [Common Challenges and Solutions](#common-challenges-and-solutions)
    - [1. Why Does My useEffect Hook Keep Running Infinitely?](#1-why-does-my-useeffect-hook-keep-running-infinitely)
    - [2. Why Isn't My State Updating Correctly in React Hooks?](#2-why-isnt-my-state-updating-correctly-in-react-hooks)
    - [3. Do I Need useMemo and useCallback for Everything in React?](#3-do-i-need-usememo-and-usecallback-for-everything-in-react)
    - [4. How Do I Fix the "React Hook useEffect has a missing dependency" Warning?](#4-how-do-i-fix-the-react-hook-useeffect-has-a-missing-dependency-warning)
  - [Conclusion](#conclusion)

<!-- tocstop -->

## History

When I first started learning React, class components were the norm, requiring a lot of code. Then, functional components were introduced, which changed everything. As React continues to evolve, one of its most transformative features has been **React Hooks**, introduced in React 16.8 in February 2019. Now in 2025, **React Hooks** have become an essential part of modern React development, fundamentally changing how developers work with state and side effects in functional components.

Before **React Hooks**, developers had to use class components to manage state and lifecycle methods. This often led to complex, hard-to-maintain code with logic scattered across different lifecycle methods. **React Hooks** solved this problem by allowing developers to use state and other React features in functional components, leading to cleaner, more reusable code.

In this comprehensive guide, we'll explore everything you need to know about **React Hooks**. Whether you're a developer with basic knowledge of React looking to deepen your understanding or a beginner trying to grasp the fundamentals, this guide will walk you through the core concepts, best practices, and advanced techniques to master **React Hooks**.

## What Are React Hooks?

> **React Hooks** are functions that let you "hook into" React state and lifecycle features from functional components.

They were introduced to solve problems that developers faced with class components, such as reusing stateful logic between components, organizing related code that was split across different lifecycle methods, and dealing with confusing class binding. React Hooks provide a more direct API to the React concepts you already know such as props, state, context, refs, and lifecycle events. Hooks don't fundamentally change how React works; rather, they offer a more ergonomic way to access React's powerful features.

At their core, **React Hooks** allow you to:

- Manage state within functional components

- Perform side effects in a controlled way

- Reuse stateful logic across multiple components

- Access context in functional components

- Create custom hooks to share logic between components

In 2025, **React Hooks** have matured considerably, with the React team and community continually improving their performance and adding new capabilities. Let's explore the fundamental hooks that form the foundation of React hooks system.

### Why are Hooks important for beginners?

- **Simpler Code:** Hooks help you write less code to achieve the same results.

- **Easier to Learn:** For many, Hooks are easier to understand than the intricacies of class component lifecycle methods.

- **Modern React:** The React community and the React team are focusing on Hooks as the primary way to build components. Learning Hooks is essential for working with modern React codebases.

- **Reusable Logic:** Create custom hooks to use through out the projects

### The Two Golden Rules of Hooks

There are two important rules about where you can use Hooks, but don't get bogged down in them right now. Just be aware of them:

1. **Top Level Only:** Use Hooks at the top level of your function component, before any return statement. Don't put them inside loops, conditions, or nested functions.

2. **React Functions Only:** Use Hooks inside React function components or custom Hooks (which are also functions).

React provides an ESLint plugin that will warn you if you break these rules, so you'll get help as you code!

## The Core Hooks: useState and useEffect

### useState: Managing Component State

The `useState` hook is the most basic and frequently used hook in React. It allows you to add state to functional components, something that was previously only possible with class components.

```tsx

import React, { useState } from 'react';

function Counter() {
 // Declare a state variable called "count" with initial value of 0
 
 const [count, setCount] = useState(0);
 
 return (
  <div>
   <p>You clicked {count} times</p>
   <button onClick={() => setCount(count + 1)}>
    Click me
   </button>
  </div>
 );
}
```

In this example, `useState` returns a pair: the current state value (`count`) and a function to update it (`setCount`). The argument passed to `useState` is the initial state value. When you call `setCount`, React re-renders the component with the new state value.

#### useState with Complex State

While the example above uses a simple number as state, `useState` can handle more complex data structures like objects and arrays. However, unlike `this.setState` in class components, `useState` doesn't automatically merge update objects.

```tsx

import React, { useState } from 'react';

function UserProfile() {
  
  const [user, setUser] = useState({
    name: 'John',
    email: 'john@example.com',
    preferences: {
      darkMode: false,
      notifications: true
    }
  });

  // Update a nested property in state
  const toggleDarkMode = () => {
    setUser({
      ...user,
      preferences: {
        ...user.preferences,
        darkMode: !user.preferences.darkMode
      }
    });
  };

  return (
    <div>
      <h2>{user.name}'s Profile</h2>
      <p>Email: {user.email}</p>
      <p>Dark Mode: {user.preferences.darkMode ? 'On' : 'Off'}</p>
      <button onClick={toggleDarkMode}>Toggle Dark Mode</button>
    </div>
  );
}
```

Notice how we have to use the spread operator (`...`) to maintain the existing state while updating only specific parts. This **immutable** update pattern is common in React and helps ensure predictable [state management](https://devdreaming.com/blogs/simplify-state-management-with-reactjs-context-api).

#### Functional Updates in useState

When new state depends on previous state, React recommends using the functional update form of `useState`:

```tsx

function Counter() {
  
  const [count, setCount] = useState(0);

  // Using functional update form
  const increment = () => {
    setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );

}
```

This approach ensures you're always working with the most current state value, which is essential in scenarios with multiple state updates or asynchronous code.

### useEffect: Managing Side Effects

The `useEffect` hook lets you perform side effects in functional components. Side effects can include data fetching, subscriptions, or manually changing the DOM.

```tsx

import React, { useState, useEffect } from 'react';

function UserData() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://api.example.com/user');
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        if (error) {
          console.error('Error fetching user data:', error);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
    // Cleanup function
    return () => {
      // add cleanup code here such as window.removeEventListener
    };    
  }, []); // Empty dependency array means this effect runs once on mount

  if (loading) return <div>Loading...</div>;

  if (!user) return <div>No user data found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

This example demonstrates a common pattern of using `useEffect` for data fetching. The empty dependency array `[]` means the effect runs only when the component mounts. The function returned from the effect serves as a cleanup mechanism, which runs when the component unmounts or before the effect runs again.

#### The Dependency Array

The dependency array is a crucial part of `useEffect`. It determines when the effect should run:

- **Empty array (****`[]`****)**: The effect runs once after the initial render

- **Variables in the array (****`[var1, var2]`****)**: The effect runs after the initial render and whenever any dependency (var1 or var2) changes.

- **No array**: The effect runs after every render

```tsx

function SearchResults({ query }) {

  const [results, setResults] = useState([]);

  useEffect(() => {

    if (query.trim() === '') return;

    const fetchResults = async () => {
      const response = await fetch(`https://api.example.com/search?q=${query}`);
      const data = await response.json();
      setResults(data);
    };

    fetchResults();

  }, [query]); // Effect runs when query changes

  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
}
```

In the above example, the effect runs whenever the `query` prop changes, fetching new search results.

#### Common useEffect Patterns

**1\. Data Fetching**

```tsx

useEffect(() => {

 let isMounted = true;

 const fetchData = async () => {
  const data = await fetchSomeData();
  if (isMounted) setData(data);
 };

 fetchData();

 return () => { isMounted = false };

}, [dependencyValue]);
```

**2\. Subscriptions**

```tsx

useEffect(() => {

 const subscription = subscribeToEvent(eventId, handleEvent);
 return () => unsubscribe(subscription);

}, [eventId]);
```

**3\. DOM Manipulations**

```tsx

useEffect(() => {

 const element = document.getElementById('some-element');
 element.style.opacity = '1';

 return () => {
  element.style.opacity = '0';
 };

}, []);
```

## Additional Core React Hooks

Here is few other React hooks which are widely used along with `useState` and `useEffect` hooks.

### 1\. useContext: Managing Global State

The `useContext` hook provides a way to pass data through the component tree without manually passing props at every level. This is especially useful for theme settings, user authentication, or language preferences. Basically using `useContext` hook you can access [React's context APIs](https://devdreaming.com/blogs/simplify-state-management-with-reactjs-context-api).

```tsx


import React, { createContext, useContext, useState } from 'react';

// Create a context
const ThemeContext = createContext();

// Provider component
function ThemeProvider({ children }) {

 const [theme, setTheme] = useState('light');

 const toggleTheme = () => {
  setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
 };

 return (
  <ThemeContext.Provider value={{ theme, toggleTheme }}>
   {children}
  </ThemeContext.Provider>
 );
}


// Consumer component
function ThemedButton() {

 // Use the context
 const { theme, toggleTheme } = useContext(ThemeContext);

 return (
  <button
   onClick={toggleTheme}
   style={{
    background: theme === 'light' ? '#fff' : '#333',
    color: theme === 'light' ? '#333' : '#fff',
    padding: '10px 15px',
    border: `1px solid ${theme === 'light' ? '#333' : '#fff'}`
   }}
  >
   Toggle Theme
  </button>
 );
}

// App that uses the theme
function App() {

 return (
  <ThemeProvider>
   <div style={{ padding: '20px' }}>
    <h1>Context Example</h1>
    <ThemedButton />
   </div>
  </ThemeProvider>
 );
}
```

This pattern creates a "theme context" that can be accessed by any component within the `ThemeProvider`, eliminating the need to pass the theme through multiple layers of components.

### 2\. useReducer: Complex State Logic

For more complex state logic, `useReducer` provides an alternative to `useState`. Inspired by Redux, it's particularly useful when the next state depends on the previous state or when state transitions are complex. Checkout following example:

```tsx

import React, { useReducer } from 'react';

// Reducer function
function todoReducer(state, action) {
  
  switch (action.type) {
    
    case 'ADD_TODO':
      
      return [...state, {
        id: Date.now(),
        text: action.payload,
        completed: false
      }];
      
    case 'TOGGLE_TODO':
      
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
      
    case 'DELETE_TODO':
      
      return state.filter(todo => todo.id !== action.payload);
      
    default:
      
      return state;
      
  }
}

function TodoApp() {
  
  const [todos, dispatch] = useReducer(todoReducer, []);
  
  const [text, setText] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (!text.trim()) return;
    dispatch({ type: 'ADD_TODO', payload: text });
    setText('');
  };
  
  return (
  
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={text}
          onChange={(e) => setText(e.target.value)}
          placeholder="Add a todo"
        />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {todos.map(todo => (
          <li
            key={todo.id}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
          
            <span onClick={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}>
              {todo.text}
            </span>
            
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
              Delete
            </button>
            
          </li>
        ))}
        
      </ul>
    </div>
  );
}
```

The `useReducer` hook is preferred over `useState` when:

- State logic is complex involving multiple sub-values

- The next state depends on the previous one

- You want to optimize performance for components that trigger deep updates

### 3\. useRef: Persisting Values Between Renders

The `useRef` hook creates a mutable reference that persists across renders. Unlike state, changing a ref doesn't trigger a re-render.

```tsx

import React, { useRef, useState, useEffect } from 'react';


function StopwatchWithFocus() {

  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const timerIdRef = useRef(null);
  const inputRef = useRef(null);

  useEffect(() => {
    if (isRunning) {
      timerIdRef.current = setInterval(() => {
        setTime(prevTime => prevTime + 1);
      }, 1000);

    } else if (timerIdRef.current) {
      clearInterval(timerIdRef.current);
    }

    return () => {
      if (timerIdRef.current) {
        clearInterval(timerIdRef.current);
      }
    };
  }, [isRunning]);

  const handleStartStop = () => {
    setIsRunning(!isRunning);
  };

  const handleReset = () => {

    setIsRunning(false);
    setTime(0);

    // Focus the input after reset

    inputRef.current.focus();

  };

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;

    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;

  };

  return (

    <div>

      <h2>Stopwatch: {formatTime(time)}</h2>

      <button onClick={handleStartStop}>
        {isRunning ? 'Stop' : 'Start'}
      </button>

      <button onClick={handleReset}>Reset</button>

      <div>

        <input
          ref={inputRef}
          type="text"
          placeholder="Notes about this timer session..."
        />

      </div>
    </div>
  );
}
```

In this example, `useRef` serves two different purposes:

1. `timerIdRef` holds the interval ID, which needs to persist between renders but doesn't affect the UI.

2. `inputRef` provides a reference to the DOM input element, allowing us to programmatically focus it.

## Performance Optimization Hooks

There are few hooks that improve the performance of your React app, you must use these hook wherever it's applicable.

### useMemo: Memoizing Expensive Calculations

The `useMemo` hook helps optimize performance by memoizing expensive calculations so they only recompute when dependencies change.

```tsx

import React, { useState, useMemo } from 'react';

function ExpensiveCalculation({ numbers }) {

  const [filter, setFilter] = useState('');

  // Memoize expensive calculation
  const filteredAndSortedNumbers = useMemo(() => {

    console.log('Computing filtered and sorted numbers...');

    // Simulating expensive operation
    const filtered = numbers.filter(n =>
      n.toString().includes(filter)
    );
    // Sort is expensive for large arrays
    return [...filtered].sort((a, b) => a - b);
  }, [numbers, filter]); // Only recompute when numbers or filter changes

  return (
    <div>
      <input
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter numbers..."
      />

      <h3>Filtered and Sorted Numbers:</h3>

      <ul>
        {filteredAndSortedNumbers.map(n => (
          <li key={n}>{n}</li>
        ))}
      </ul>

    </div>
  );
}
```

This optimization is particularly valuable for:

- Expensive calculations that don't need to be recalculated on every render

- Preventing unnecessary re-renders of child components that rely on computed values

### useCallback: Memoizing Functions

The `useCallback` hook memoizes function definitions, which is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders.

```tsx

import React, { useState, useCallback } from 'react';

// Child component that uses React.memo
const ItemList = React.memo(({ items, onItemClick }) => {

  console.log('ItemList rendered');

  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});


function ShoppingList() {
  const [items, setItems] = useState([
    { id: 1, name: 'Apples' },
    { id: 2, name: 'Bananas' },
    { id: 3, name: 'Cherries' }
  ]);

  const [selectedId, setSelectedId] = useState(null);

  // Without useCallback, this function would be recreated on every render
  // causing ItemList to re-render unnecessarily
  const handleItemClick = useCallback((id) => {
    setSelectedId(id);
    console.log(`Item ${id} selected`);
  }, []); // Empty dependency array means this function only changes if dependencies change

  return (

    <div>
      <h2>Shopping List</h2>
      <ItemList items={items} onItemClick={handleItemClick} />
      {selectedId && <p>You selected item with ID: {selectedId}</p>}
    </div>

  );

}
```

The `useCallback` hook is essential when:

- Passing callbacks to optimized child components using `React.memo`

- Creating event handlers that depend on props or state

- Working with custom hooks that rely on function equality

## Building Custom Hooks

One of the most powerful features of **React Hooks** is the ability to create custom hooks, allowing you to extract component logic into reusable functions.

```tsx

import { useState, useEffect } from 'react';

// Custom hook for window dimensions
function useWindowSize() {

  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {

    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    // Add event listener
    window.addEventListener('resize', handleResize);

    // Clean up
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Empty array means only run on mount and unmount
  return windowSize;
}
```

```tsx

// Using the above custom hook
function ResponsiveComponent() {
  const size = useWindowSize();

  return (

    <div>
      <h2>Window Size:</h2>
      <p>Width: {size.width}px, Height: {size.height}px</p>
      {size.width < 768 ? (
        <p>Mobile view</p>
      ) : (
        <p>Desktop view</p>
      )}
    </div>
  );
}
```

### More Custom Hook Examples

Here are a few more examples of useful custom hooks:

#### 1\. useFetch: Simplified Data Fetching

```tsx

function useFetch(url, options = {}) {

  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {

    let isMounted = true;
    const controller = new AbortController();

    const fetchData = async () => {
      try {

        setLoading(true);

        const response = await fetch(url, {
          ...options,
          signal: controller.signal
        });
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        const result = await response.json();
        if (isMounted) {
          setData(result);
          setError(null);
        }
      } catch (err) {
        if (isMounted && err.name !== 'AbortError') {
          setError(err.message);
          setData(null);
        }
      } finally {
        if (isMounted) setLoading(false);
      }
    };
    fetchData();

    // Cleanup
    return () => {
      isMounted = false;
      controller.abort();
    };

  }, [url, JSON.stringify(options)]);

  return { data, loading, error };

}
```

#### 2\. useLocalStorage: Persistent State

```tsx

function useLocalStorage(key, initialValue) {

  // Get from local storage then parse
  const getStoredValue = () => {

    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } 
    catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  };

  // State to store our value
  const [storedValue, setStoredValue] = useState(getStoredValue);

  // Return a wrapped version of useState's setter function that
  // persists the new value to localStorage
  const setValue = (value) => {

    try {
      // Allow value to be a function like in useState
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      // Save state
      setStoredValue(valueToStore);
      // Save to localStorage
      window.localStorage.setItem(key, JSON.stringify(valueToStore));

    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue];
}
```

## React Hooks Best Practices in 2025

Over the years, the React community has established solid best practices for working with hooks. Here are some key recommendations:

### 1\. Rules of Hooks

We already learned the top 2 rules at the beginning of this article. You should Always follow the [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks):

- Only call hooks at the top level of your React functions

- Don't call hooks inside loops, conditions, or nested functions

- Only call hooks from React function components or custom hooks

Use hooks to organize related logic together, rather than splitting it across different lifecycle methods as in class components.

```tsx


// Prefer this
function UserProfile({ userId }) {

  const { data, loading } = useFetch(`/api/users/${userId}`);
  const { settings, updateSettings } = useUserSettings(userId);

  // All user-related logic stays together
  // ...
}

// Instead of spreading across lifecycle methods like in class components
```

### 3\. Create Custom Hooks for Reusable Logic

Extract logic into custom hooks when it's used across multiple components:

- Without custom hook (repeated in multiple components):

```tsx

function Component1() {

  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {

    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {

      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);

    };

  }, []);

  // ...
}
```

- With custom hook:

```tsx

function useOnlineStatus() {

  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {

    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {

      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);

    };

  }, []);

  return isOnline;
}
```

- Now usage is simple in any component:

```tsx

function Component1() {
  const isOnline = useOnlineStatus();
  // ...
}
```

### 4\. Optimize Expensive Calculations with useMemo and useCallback

Use `useMemo` for expensive computations and `useCallback` for function memoization, but don't overuse them for simple operations.

### 5\. Manage Complex State with useReducer

For complex state logic, prefer `useReducer` over multiple `useState` calls:

- Instead of multiple useState calls:

```tsx

function ComplexForm() {

  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);
  // Many handlers that update different pieces of state
  // ...
}
```

- Prefer useReducer for complex state:

```tsx

function formReducer(state, action) {

  switch (action.type) {

    case 'UPDATE_FIELD':

      return { ...state, [action.field]: action.value };

    case 'SET_ERRORS':

      return { ...state, errors: action.errors };

    case 'START_SUBMIT':

      return { ...state, isSubmitting: true, isSuccess: false };

    case 'SUBMIT_SUCCESS':

      return { ...state, isSubmitting: false, isSuccess: true };

    // ...other cases
    default:
      return state;
  }
}

function ComplexForm() {

  const [state, dispatch] = useReducer(formReducer, {

    name: '',
    email: '',
    password: '',
    errors: {},
    isSubmitting: false,
    isSuccess: false

  });

  // Unified state management with actions
  // ...
}
```

### 6\. Separate Data Fetching from UI

Extract data fetching logic into custom hooks to keep components focused on UI rendering:

```tsx

// Custom hook for data fetching
function useUser(userId) {

  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Fetching logic...
  }, [userId]);

  return { user, loading, error };
}

// Component focused on UI
function UserProfile({ userId }) {

  const { user, loading, error } = useUser(userId);

  if (loading) return <Spinner />;

  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      <h2>{user.name}</h2>
      {/* Other UI elements */}
    </div>
  );

}
```

## Advanced Hook Patterns

As you become more familiar with basic hooks like `useState`, `useEffect`, and `useContext`, you can explore **advanced hook patterns** that enable better organization, reusability, and more scalable code. These patterns allow you to compose, share, and manage logic in ways that enhance the readability and maintainability of your application.

### Composition of Hooks

Compose multiple hooks together to create more powerful abstractions:

```tsx

function useUserDataWithStatus(userId) {

  const { data: user, loading: userLoading, error: userError } = useFetch(`/api/users/${userId}`);
  const isOnline = useOnlineStatus();

  return {
    user,
    loading: userLoading,
    error: userError,
    isOnline
  };

}
```

### Context + Reducer Pattern

Combine context and reducers for global state management:

```tsx

// Create context
const AppStateContext = createContext();
const AppDispatchContext = createContext();

// Reducer function
function appReducer(state, action) {

  switch (action.type) {

    case 'SET_USER':
      return { ...state, user: action.payload };

    case 'TOGGLE_THEME':
      return { ...state, darkMode: !state.darkMode };

    // other cases
    default:
      return state;
  }
}
  
// Provider component
function AppProvider({ children }) {

  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    darkMode: false,
    // other global state
  });

  return (
    <AppStateContext.Provider value={state}>
      <AppDispatchContext.Provider value={dispatch}>
        {children}
      </AppDispatchContext.Provider>
    </AppStateContext.Provider>
  );
}

// Custom hooks to use the context
function useAppState() {

  const context = useContext(AppStateContext);

  if (context === undefined) {
    throw new Error('useAppState must be used within an AppProvider');
  }

  return context;

}

function useAppDispatch() {

  const context = useContext(AppDispatchContext);

  if (context === undefined) {
    throw new Error('useAppDispatch must be used within an AppProvider');
  }
  return context;
}
```

This pattern provides a lightweight alternative to [Redux](https://devdreaming.com/blogs/category/redux) for many applications.

## React Hooks in 2025: What's New?

As React continues to evolve, several enhancements have been made to hooks in recent years:

### Improved Performance

The React team has continuously optimized the internals of hooks for better performance:

- Reduced overhead for each hook call

- Better batching of state updates

- Improved memory usage patterns

### Enhanced DevTools Support

React DevTools now provide better debugging support for hooks:

- Inspect hook values during development

- Track which components are causing re-renders

- View dependency arrays and their changes

### Integration with Concurrent Mode

Hooks work seamlessly with React's Concurrent Mode features:

- Suspense for data fetching

- Transitions for state updates

- Prioritization of different updates

### TypeScript Improvements

TypeScript integration with hooks has improved dramatically:

- Better type inference for hook return values

- Enhanced autocomplete for hook parameters

- Stricter typing for custom hooks

## Common Challenges and Solutions

### 1\. Why Does My useEffect Hook Keep Running Infinitely?

A common issue is creating infinite loops with `useEffect`:

```tsx

// Problematic code
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // This creates an infinite loop!
  useEffect(() => {
    fetchResults(query).then(data => setResults(data));
  }, [results]); // results change triggers effect, which updates results!
  // ...
}
```

**Solution:** Ensure your dependency array only includes values that, when changed, should trigger the effect:

```tsx

// Fixed code
useEffect(() => {
  fetchResults(query).then(data => setResults(data));
}, [query]); // Only refetch when query changes
```

### 2\. Why Isn't My State Updating Correctly in React Hooks?

Closures in JavaScript can lead to stale state in event handlers:

```tsx

function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      // This uses the initial count value (0) in the closure
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []); // Empty deps array means this only runs on mount
  // count will only ever be 1!
}
```

**Solution:** Use the functional update form of state setters:

```tsx

useEffect(() => {
  const timer = setInterval(() => {
    // This always uses the latest count value
    setCount(prevCount => prevCount + 1);
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

### 3\. Do I Need useMemo and useCallback for Everything in React?

A common mistake is wrapping everything in `useMemo` and `useCallback`:

```tsx

// Excessive optimization
function UserList({ users }) {
  // Unnecessary for simple operations
  const displayUsers = useMemo(() => {
    return users.map(user => (
      <li key={user.id}>{user.name}</li>
    ));
  }, [users]);
  
  // Unnecessary for this simple function
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return (
    <div>
      <h2>User List</h2>
      <ul>{displayUsers}</ul>
      <button onClick={handleClick}>Click Me</button>
    </div>
  );
}
```

**Solution:** Only use `useMemo` and `useCallback` when there's a clear performance benefit:

- For expensive calculations

- When passing callbacks to optimized components using `React.memo`

- When dependencies change frequently

### 4\. How Do I Fix the "React Hook useEffect has a missing dependency" Warning?

Managing dependencies can be tricky:

```tsx
function SearchComponent({ onSearch, defaultQuery }) {
  const [query, setQuery] = useState(defaultQuery);
  
  // Missing dependencies
  useEffect(() => {
    const results = searchAPI(query);
    onSearch(results);
  }, []); // Missing dependencies: query, onSearch
}
```

**Solution:** Use ESLint's `react-hooks/exhaustive-deps` rule and address all warnings:

```tsx

useEffect(() => {
  const results = searchAPI(query);
  onSearch(results);
}, [query, onSearch]); // All dependencies included
```

If `onSearch` changes too often, consider stabilizing it with `useCallback` in the parent component.

## Conclusion

**React Hooks** have fundamentally changed how we build React applications, enabling more concise, maintainable, and reusable code. In 2025, they remain the preferred approach for managing state and side effects in React components, with ongoing improvements in performance, developer experience, and integration with other React features.

By mastering hooks, you can:

- Write more expressive and concise components

- Easily reuse stateful logic between components

- Organize related code together

- Create powerful abstractions with custom hooks

- Take advantage of the latest React features and optimizations

Whether you're a beginner just starting with React or an experienced developer looking to deepen your understanding, getting comfortable with hooks is essential for modern React development. Start with the basics `useState` and `useEffect` and gradually explore more advanced hooks and patterns as you build more complex applications.

Remember that the true power of hooks comes not just from using them individually, but from composing them together to create elegant solutions to complex problems. Happy coding!

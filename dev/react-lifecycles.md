# Quick Reference: The React Component Lifecycle

React components go through a "lifecycle" of phases: mounting, updating, and unmounting.

Understanding these phases and their associated methods is crucial for effective component management, data fetching, and performance optimization.

Here's a breakdown of the key phases and methods:

## **I. Mounting Phase (Component is created and inserted into the DOM)**

These methods are called in order when an instance of a component is being created and inserted into the DOM:

1. **`constructor(props)`**

   - **Purpose:** Initializes state, binds event handlers.
   - **When it's called:** First method called.
   - **Do's:**
     - Initialize `this.state` directly.
     - Bind event handler methods to the instance using `this.method = this.method.bind(this)`.
     - Call `super(props)` as the first statement.
   - **Don'ts:**
     - Cause side effects (e.g., data fetching, subscriptions).
     - Call `setState()`.

2. **`static getDerivedStateFromProps(props, state)`**

   - **Purpose:** Updates state based on changes in props (rarely used).
   - **When it's called:** Before every render, both on initial mount and subsequent updates.
   - **Do's:**
     - Return an object to update state, or `null` to update nothing.
     - This method is static and doesn't have access to `this`.
   - **Don'ts:**
     - Cause side effects.
     - Access `this`.
     - Use for data fetching.
   - **Note:** Use with caution. Often, a full re-render is more appropriate than deriving state from props.

3. **`render()`**

   - **Purpose:** Renders the component's UI.
   - **When it's called:** After `getDerivedStateFromProps` and before the component is mounted.
   - **Do's:**
     - Return JSX (React elements, arrays, fragments, portals, strings, or numbers).
     - It should be a pure function: given the same props and state, it should always return the same result.
   - **Don'ts:**
     - Modify component state (`setState()`).
     - Cause side effects (e.g., interacting with the browser DOM directly, data fetching).

4. **`componentDidMount()`**
   - **Purpose:** Performs side effects after the component has been rendered and mounted to the DOM.
   - **When it's called:** Immediately after the component is mounted (inserted into the DOM tree).
   - **Do's:**
     - Perform data fetching (e.g., AJAX requests).
     - Set up subscriptions (e.g., websockets, event listeners).
     - Interact with the DOM directly (e.g., integrating with third-party libraries).
     - Call `setState()` (will trigger an extra render, but happens before the browser updates the screen).
   - **Don'ts:**
     - Unnecessary `setState()` calls that could be done in `constructor`.

---

## **II. Updating Phase (Component is re-rendered due to prop or state changes)**

These methods are called when a component is being re-rendered due to changes in props or state:

1. **`static getDerivedStateFromProps(nextProps, prevState)`**

   - **Purpose:** (Already explained above) Called before every render, including updates.

2. **`shouldComponentUpdate(nextProps, nextState)`**

   - **Purpose:** Optimizes performance by preventing unnecessary re-renders.
   - **When it's called:** Before rendering when new props or state are being received.
   - **Do's:**
     - Return `true` if the component should re-render.
     - Return `false` if the component should _not_ re-render.
     - Compare `this.props` with `nextProps` and `this.state` with `nextState`.
   - **Don'ts:**
     - Cause side effects.
     - Deep equality checks can be expensive; use with care.
   - **Note:** `PureComponent` implements a shallow comparison in `shouldComponentUpdate` automatically.

3. **`render()`**

   - **Purpose:** (Already explained above) Renders the updated UI.

4. **`getSnapshotBeforeUpdate(prevProps, prevState)`**

   - **Purpose:** Captures information from the DOM _before_ it's potentially changed.
   - **When it's called:** Immediately before the most recently rendered output is committed to the DOM.
   - **Do's:**
     - Return a value (e.g., scroll position) that will be passed as the third argument to `componentDidUpdate`.
     - This is useful for preserving scroll position in chat applications.
   - **Don'ts:**
     - Cause side effects.
   - **Note:** This method is rarely used.

5. **`componentDidUpdate(prevProps, prevState, snapshot)`**
   - **Purpose:** Performs side effects after the component has updated and re-rendered.
   - **When it's called:** Immediately after updating occurs.
   - **Do's:**
     - Perform network requests (e.g., fetch new data based on prop changes).
     - Interact with the DOM based on the updated state/props.
     - Compare `prevProps` and `prevState` to `this.props` and `this.state` to conditionally trigger side effects.
     - Call `setState()` _conditionally_ (to avoid infinite loops).
   - **Don'ts:**
     - Unconditional `setState()` calls.

---

## **III. Unmounting Phase (Component is removed from the DOM)**

This method is called when a component is being removed from the DOM:

1. **`componentWillUnmount()`**
   - **Purpose:** Performs cleanup actions before the component is destroyed.
   - **When it's called:** Immediately before a component is unmounted and destroyed.
   - **Do's:**
     - Invalidate timers (e.g., `clearTimeout`, `clearInterval`).
     - Cancel network requests.
     - Remove event listeners.
     - Clean up any subscriptions created in `componentDidMount`.
   - **Don'ts:**
     - Call `setState()` (the component will not be re-rendered).

---

## **IV. Error Handling (Introduced in React 16)**

These methods are called when there is an error during rendering, in a lifecycle method, or in a constructor of any child component:

1. **`static getDerivedStateFromError(error)`**

   - **Purpose:** Renders a fallback UI after an error has been thrown by a descendant component.
   - **When it's called:** When a descendant component throws an error.
   - **Do's:**
     - Return an object to update state (e.g., set an `hasError` state to `true`).
     - This method is static and doesn't have access to `this`.
   - **Don'ts:**
     - Cause side effects.

2. **`componentDidCatch(error, info)`**
   - **Purpose:** Logs error information.
   - **When it's called:** After an error has been thrown by a descendant component.
   - **Do's:**
     - Log the error to an error reporting service.
     - Can call `setState()` to render a fallback UI.
   - **Note:** Error boundaries only catch errors in the render, lifecycle methods, and constructors of their children. They don't catch errors within event handlers.

---

**Key Considerations:**

- **Functional Components & Hooks:** For functional components, React Hooks (`useState`, `useEffect`, `useContext`, etc.) provide a way to manage state and side effects, essentially replacing the class component lifecycle methods in a more functional paradigm. `useEffect` is the primary hook for handling side effects that map to `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.
- **Performance:** Understanding the lifecycle helps in optimizing performance by preventing unnecessary re-renders (e.g., `shouldComponentUpdate` or `React.memo` for functional components).
- **Side Effects:** Be mindful of where you place side effects. They generally belong in `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` (or `useEffect` for functional components).
- **Legacy Methods:** Some older lifecycle methods (`componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate`) are considered unsafe in modern React and should be avoided, as they can lead to subtle bugs, especially with concurrent mode.

Here are 10 quizzes based on the provided explanations of `useReducer` with TypeScript, designed to test different aspects of the reader's understanding.

-----

### Quiz 1: Multiple Choice (Core Concepts)

**Question:** Which of the following is NOT a core component of the `useReducer` hook?
A) State
B) Action
C) Dispatch
D) Selector

**Correct Answer:** D) Selector

-----

### Quiz 2: True/False (TypeScript Benefits)

**Question:** True or False: Using `useReducer` with TypeScript primarily helps in catching runtime errors related to state and action types.

**Correct Answer:** False. While it can prevent runtime errors, its primary benefit is catching *compile-time* errors due to type mismatches.

-----

### Quiz 3: Fill in the Blanks (Signature)

**Question:** Complete the `useReducer` signature:
`const [______, ______] = useReducer(______, ______, init);`

**Correct Answer:** `const [state, dispatch] = useReducer(reducer, initialState, init);`

-----

### Quiz 4: Code Snippet Analysis (State Typing)

**Question:** Consider the following TypeScript interface for a shopping cart item:

```typescript
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}
```

If you wanted to define the initial state for a `useReducer` cart where there are no items, which of the following would be the most appropriate type for `initialCartState`?

A) `const initialCartState: object = {};`
B) `const initialCartState: CartItem[] = [];`
C) `const initialCartState: { items: CartItem[]; totalItems: number; totalPrice: number; } = { items: [], totalItems: 0, totalPrice: 0 };`
D) `const initialCartState: any = { items: [], totalItems: 0, totalPrice: 0 };`

**Correct Answer:** C) `const initialCartState: { items: CartItem[]; totalItems: number; totalPrice: number; } = { items: [], totalItems: 0, totalPrice: 0 };` (This correctly represents the `CartState` interface defined in the example.)

-----

### Quiz 5: Short Answer (Action Payload)

**Question:** In the `CounterAction` type below, what is the purpose of the `payload` property for the `increment` action?

```typescript
type CounterAction =
  | { type: 'increment'; payload: number }
  | { type: 'decrement'; payload: number }
  | { type: 'reset' };
```

**Correct Answer:** The `payload` property is used to carry additional data needed by the reducer to perform the action. In this case, for `increment`, it specifies *by how much* the count should be increased.

-----

### Quiz 6: Identify the Error (Reducer Purity)

**Question:** Examine the following reducer function snippet. What principle of reducer functions is potentially violated, and why?

```typescript
function buggyReducer(state: { value: number }, action: { type: 'ADD_API_DATA' }): { value: number } {
  // Simulating an API call
  fetch('/api/data')
    .then(res => res.json())
    .then(data => {
      state.value += data.amount; // Direct modification
    });
  return state;
}
```

**Correct Answer:** This reducer violates the principle of **purity**.

1. **Side Effect:** It performs a `fetch` (an asynchronous operation with side effects). Reducers should be pure functions and not cause side effects.
2. **Immutability:** It directly modifies the `state` object (`state.value += data.amount;`) instead of returning a new state object. Reducers must return new state objects, ensuring immutability.

-----

### Quiz 7: Scenario Application (When to use `useReducer`)

**Question:** You are building a complex dashboard with multiple filters (e.g., date range, category, status). When a filter changes, several parts of the dashboard state need to be updated and potentially re-fetched from an API. Would `useState` or `useReducer` be a more suitable choice for managing this dashboard's filter state, and why?

**Correct Answer:** `useReducer` would be a more suitable choice.
**Reasoning:**

* **Complex State Logic:** Managing multiple related filters and their interactions, along with derived states (like whether a filter is active), fits `useReducer`'s strength.
* **Related State Updates:** When one filter changes, it might trigger updates to other aspects of the state, which is easier to coordinate within a single reducer.
* **Centralized Logic:** All filter manipulation logic (applying, clearing, validating) can be centralized in one reducer function, making it more organized and testable than scattered `useState` calls.

-----

### Quiz 8: Matching (Action Type to Purpose)

**Question:** Match the `CartAction` type to its primary purpose in the Shopping Cart example:

1. `{ type: 'ADD_ITEM'; payload: { item: Omit<CartItem, 'quantity'>; quantity?: number } }`
2. `{ type: 'REMOVE_ITEM'; payload: { id: string } }`
3. `{ type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }`

A) Adjusts the number of a specific product already in the cart.
B) Completely empties the shopping cart.
C) Adds a new product to the cart, or increases its quantity if already present.
D) Deletes a product entirely from the cart.

**Correct Answer:**

1. C) Adds a new product to the cart, or increases its quantity if already present.
2. D) Deletes a product entirely from the cart.
3. A) Adjusts the number of a specific product already in the cart.

-----

### Quiz 9: Fill in the Code (TypeScript Type Assertion)

**Question:** In the `RegistrationForm` example, when accessing `e.target.name` in `handleInputChange`, why is `as keyof Omit<FormState, 'isFormValid'>` used?

```typescript
const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    dispatch({ type: 'UPDATE_FIELD', payload: { fieldName: name as keyof Omit<FormState, 'isFormValid'>, value } });
};
```

**Correct Answer:** The `name` property from `e.target` (an HTML input element) is of type `string`. However, the `fieldName` property in our `UPDATE_FIELD` action's payload is strictly typed as `keyof Omit<FormState, 'isFormValid'>` (i.e., 'username', 'email', or 'password'). The type assertion `as keyof Omit<FormState, 'isFormValid'>` is used to tell TypeScript that, at this point, we are confident the `string` value from `name` will indeed be one of the allowed keys for our form state, allowing the type checker to pass without error.

-----

### Quiz 10: Open-Ended Discussion (When `useState` is better)

**Question:** While `useReducer` is powerful, there are situations where `useState` is simpler and more appropriate. Describe a scenario where you would explicitly choose `useState` over `useReducer` and briefly explain why.

**Correct Answer:** (Any reasonable answer demonstrating understanding of `useState`'s simplicity)
**Example Answer:** I would choose `useState` for managing a simple boolean state, such as `isLoading` for an API call, or `isModalOpen` for a modal dialog.
**Reasoning:** For such simple states, the overhead of defining action types and a reducer function for `useReducer` would be unnecessary. `useState` directly allows setting the new state with a single line (`setIsLoading(true)`) which is much more concise for trivial state changes.

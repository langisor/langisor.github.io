## Example 1: Form Management with Validation

Managing form input fields, especially with validation, can become complex with multiple `useState` hooks. `useReducer` provides a cleaner way to handle all form-related state and validation logic.

### Scenario: User Registration Form

We'll create a simple registration form with fields for `username`, `email`, and `password`, along with basic validation.

#### 1\. Define State Types

The form state will hold the value of each input and any validation errors associated with them.

```typescript
// types.ts
interface FormState {
  username: {
    value: string;
    error: string | null;
    touched: boolean; // To track if the field has been interacted with
  };
  email: {
    value: string;
    error: string | null;
    touched: boolean;
  };
  password: {
    value: string;
    error: string | null;
    touched: boolean;
  };
  isFormValid: boolean;
}
```

#### 2\. Define Action Types

We'll need actions for updating field values, setting errors, marking fields as touched, and resetting the form.

```typescript
// types.ts
type FormAction =
  | { type: 'UPDATE_FIELD'; payload: { fieldName: keyof Omit<FormState, 'isFormValid'>; value: string } }
  | { type: 'SET_ERROR'; payload: { fieldName: keyof Omit<FormState, 'isFormValid'>; error: string | null } }
  | { type: 'TOUCH_FIELD'; payload: { fieldName: keyof Omit<FormState, 'isFormValid'> } }
  | { type: 'RESET_FORM' };
```

* `fieldName: keyof Omit<FormState, 'isFormValid'>`: This ensures that `fieldName` can only be one of 'username', 'email', or 'password'. `Omit<FormState, 'isFormValid'>` excludes `isFormValid` because it's derived, not a direct field.

#### 3\. Create the Reducer Function

The reducer will handle all state changes and also the validation logic.

```typescript
// formReducer.ts
import { FormState, FormAction } from './types'; // Assuming types.ts

// Basic validation functions
const validateUsername = (username: string): string | null => {
  if (username.length < 3) return 'Username must be at least 3 characters long.';
  return null;
};

const validateEmail = (email: string): string | null => {
  if (!/\S+@\S+\.\S+/.test(email)) return 'Invalid email format.';
  return null;
};

const validatePassword = (password: string): string | null => {
  if (password.length < 6) return 'Password must be at least 6 characters long.';
  return null;
};

const validateForm = (state: FormState): boolean => {
  return (
    state.username.error === null &&
    state.email.error === null &&
    state.password.error === null &&
    state.username.touched && // All fields must be touched for full validation
    state.email.touched &&
    state.password.touched
  );
};

export const initialFormState: FormState = {
  username: { value: '', error: null, touched: false },
  email: { value: '', error: null, touched: false },
  password: { value: '', error: null, touched: false },
  isFormValid: false,
};

export function formReducer(state: FormState, action: FormAction): FormState {
  let newState = { ...state };

  switch (action.type) {
    case 'UPDATE_FIELD':
      const { fieldName, value } = action.payload;
      newState = {
        ...state,
        [fieldName]: {
          ...state[fieldName],
          value,
          error: null, // Clear error on new input
        },
      };
      break;

    case 'SET_ERROR':
      newState = {
        ...state,
        [action.payload.fieldName]: {
          ...state[action.payload.fieldName],
          error: action.payload.error,
        },
      };
      break;

    case 'TOUCH_FIELD':
      newState = {
        ...state,
        [action.payload.fieldName]: {
          ...state[action.payload.fieldName],
          touched: true,
        },
      };
      break;

    case 'RESET_FORM':
      return initialFormState;

    default:
      const exhaustiveCheck: never = action;
      throw new Error(`Unhandled action type: ${exhaustiveCheck}`);
  }

  // Re-validate and update isFormValid after every state change
  const usernameError = validateUsername(newState.username.value);
  const emailError = validateEmail(newState.email.value);
  const passwordError = validatePassword(newState.password.value);

  newState.username.error = newState.username.touched ? usernameError : null;
  newState.email.error = newState.email.touched ? emailError : null;
  newState.password.error = newState.password.touched ? passwordError : null;

  newState.isFormValid = validateForm(newState);

  return newState;
}
```

* Notice how the validation logic is integrated directly into the reducer. After any state update (via `UPDATE_FIELD`, `SET_ERROR`, `TOUCH_FIELD`), the `isFormValid` status is re-evaluated.
* Errors are only displayed if the field has been `touched`.

#### 4\. Use `useReducer` in your Component

```typescript
// RegistrationForm.tsx
import React, { useReducer } from 'react';
import { initialFormState, formReducer } from './formReducer'; // Assuming formReducer.ts
import { FormState } from './types'; // Assuming types.ts

function RegistrationForm() {
  const [formState, dispatch] = useReducer(formReducer, initialFormState);

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    dispatch({ type: 'UPDATE_FIELD', payload: { fieldName: name as keyof Omit<FormState, 'isFormValid'>, value } });
  };

  const handleBlur = (e: React.FocusEvent<HTMLInputElement>) => {
    const { name } = e.target;
    dispatch({ type: 'TOUCH_FIELD', payload: { fieldName: name as keyof Omit<FormState, 'isFormValid'> } });
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Ensure all fields are marked as touched on submit attempt
    dispatch({ type: 'TOUCH_FIELD', payload: { fieldName: 'username' } });
    dispatch({ type: 'TOUCH_FIELD', payload: { fieldName: 'email' } });
    dispatch({ type: 'TOUCH_FIELD', payload: { fieldName: 'password' } });

    if (formState.isFormValid) {
      console.log('Form Submitted!', {
        username: formState.username.value,
        email: formState.email.value,
        password: formState.password.value,
      });
      dispatch({ type: 'RESET_FORM' });
    } else {
      console.log('Form is invalid. Please check errors.');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formState.username.value}
          onChange={handleInputChange}
          onBlur={handleBlur}
        />
        {formState.username.touched && formState.username.error && (
          <p style={{ color: 'red' }}>{formState.username.error}</p>
        )}
      </div>

      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formState.email.value}
          onChange={handleInputChange}
          onBlur={handleBlur}
        />
        {formState.email.touched && formState.email.error && (
          <p style={{ color: 'red' }}>{formState.email.error}</p>
        )}
      </div>

      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formState.password.value}
          onChange={handleInputChange}
          onBlur={handleBlur}
        />
        {formState.password.touched && formState.password.error && (
          <p style={{ color: 'red' }}>{formState.password.error}</p>
        )}
      </div>

      <button type="submit" disabled={!formState.isFormValid}>
        Register
      </button>
      <button type="button" onClick={() => dispatch({ type: 'RESET_FORM' })}>
        Reset
      </button>
    </form>
  );
}

export default RegistrationForm;
```

This example demonstrates how `useReducer` can centralize all form-related state and complex validation logic, making the component cleaner and the state transitions more predictable.

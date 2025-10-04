# Typescript Collections

-----

## Core TypeScript Collection Features

| Collection | Description | Key Features | TypeScript Typing | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Array** | An ordered list of elements. | Dynamic size, index-based access, rich set of methods (`map`, `filter`, `reduce`). | `T[]` or `Array<T>` | Lists, sequential data, iterating/transforming data. |
| **Tuple** | An array with a **fixed number** of elements, where each element can have a **different type**. | Fixed length, positional typing, specific types for each index. | `[Type1, Type2, Type3]` | Return values from functions (e.g., React `useState`), small, fixed-structure data records. |
| **Map** | A collection of keyed data items. | Keys can be **any type** (objects, primitives), preserves **insertion order**, efficient lookups. | `Map<KeyType, ValueType>` | Storing key-value data where keys might be non-strings, or when iteration order matters. |
| **Set** | A collection of unique values. | Automatically prevents duplicate values, efficient membership testing (`has`). | `Set<Type>` | De-duplication, checking for existence of an item. |
| **Dictionary** | A conceptual type representing a key-value collection, typically using an object. | Simple literal syntax (`{}`). Keys are usually **strings or symbols** (or numbers coerced to strings). | `{ [key: string]: ValueType }` or `Record<string, ValueType>` | Simple configuration objects, fixed property sets, JSON-serializable key-value data. |

-----

## Comparison of Key-Value Collections

The choice between a **Dictionary** (plain object) and a **Map** is crucial in TypeScript for key-value storage:

| Feature | Dictionary (Object) | Map |
| :--- | :--- | :--- |
| **Key Type** | Primarily `string` or `symbol` (number keys are coerced to strings). | **Any type**, including objects and functions, which are compared by reference. |
| **Insertion Order** | Not guaranteed (though modern JS engines often maintain it for non-integer keys). | **Guaranteed** to maintain insertion order. |
| **Iteration** | Requires extra steps (`Object.keys()`, `Object.entries()`, `for...in` which iterates over prototype properties). | Directly iterable (`for...of`, `forEach`, `keys()`, `values()`, `entries()`). |
| **Size** | Manual calculation (`Object.keys().length`). | Direct property (`.size`). |
| **Performance** | Can be slightly faster for small, static string-keyed collections. | **Faster** for frequent additions/deletions and large datasets, and for lookups where the key type isn't a string. |

-----

## Collection Conversion Techniques

Converting between collections is common. Here are the most frequent patterns:

| Conversion | Code Example |
| :--- | :--- |
| **Array to Set** (for de-duplication) | `const uniqueNumbers = new Set<number>(myArray);` |
| **Set to Array** | `const arrayFromSet = [...mySet];` (using the spread operator) |
| **Array of Tuples to Map** | `const myMap = new Map<string, number>([['a', 1], ['b', 2]]);` |
| **Map to Array of Tuples** | `const tupleArray = [...myMap.entries()]; // [['a', 1], ['b', 2]]` |
| **Dictionary to Map** | `const myMap = new Map<string, number>(Object.entries(myDictionary));` |
| **Map to Dictionary** (requires string/number keys) | `const myDictionary = Object.fromEntries(myMap.entries());` |
| **Array/Dictionary to Tuple** | Conversion is via type assertion/assignment, as Tuples are a compile-time type: `const userTuple: [number, string] = [1, 'Alice'];` |

-----

## Quick Reference: Essential Collection Methods

This table summarizes the most frequently used methods for managing and accessing data in core collections.

| Collection | Action | Method/Property | TypeScript Example |
| :--- | :--- | :--- | :--- |
| **Array** | Read/Access | `myArray[index]` | `const val = myArray[0];` |
| | Size | `.length` | `const count = myArray.length;` |
| | Add (End) | `.push(value)` | `myArray.push(5);` |
| | Iterate/Transform | `.map()`, `.filter()`, `.reduce()` | `myArray.map(x => x * 2);` |
| | Check Presence | `.includes(value)` | `const has = myArray.includes(10);` |
| **Tuple** | Read/Access | `myTuple[index]` | `const name = myTuple[1];` |
| | Size | `.length` | `const size = myTuple.length;` (Fixed at compile time) |
| **Map** | Add/Update | `.set(key, value)` | `myMap.set('id', 123);` |
| | Read/Access | `.get(key)` | `const user = myMap.get('id');` |
| | Size | `.size` | `const count = myMap.size;` |
| | Check Key | `.has(key)` | `const exists = myMap.has('id');` |
| | Remove | `.delete(key)` | `myMap.delete('id');` |
| **Set** | Add | `.add(value)` | `mySet.add(42);` |
| | Size | `.size` | `const count = mySet.size;` |
| | Check Value | `.has(value)` | `const exists = mySet.has(42);` |
| | Remove | `.delete(value)` | `mySet.delete(42);` |
| **Dictionary** | Add/Update | `myDict[key] = value` | `myDict['name'] = 'Bob';` |
| | Read/Access | `myDict[key]` | `const name = myDict['name'];` |
| | Size | `Object.keys(myDict).length` | `const count = Object.keys(myDict).length;` |
| | Check Key | `myDict.hasOwnProperty(key)` | `const has = myDict.hasOwnProperty('name');` |

-----

## Collections in React with TypeScript

Collections are essential in React for managing state and rendering dynamic content. TypeScript adds safety by ensuring the structure of these collections is always correct.

### 1\. Array for Rendering Lists

Arrays are the primary tool for rendering lists of components using the `.map()` function.

```tsx
interface Product {
  id: number;
  name: string;
  price: number;
}

// Typing the props to ensure 'products' is an array of Product
const ProductList: React.FC<{ products: Product[] }> = ({ products }) => {
  return (
    <ul>
      {products.map((product) => (
        // Key is crucial for React's reconciliation
        <li key={product.id}>
          {product.name} - ${product.price.toFixed(2)}
        </li>
      ))}
    </ul>
  );
};
```

### 2\. Tuple for `useState` Hook

The `useState` hook natively returns a tuple, allowing TypeScript to track the specific types of the state value and the setter function.

```tsx
// TypeScript infers the type as [string, React.Dispatch<React.SetStateAction<string>>]
const [username, setUsername] = useState<string>('Guest');

// If you need a more complex, fixed-size state:
type TimerState = [time: number, isRunning: boolean];

const useTimer = (initialTime: number): [time: number, start: () => void] => {
  const [state, setState] = useState<TimerState>([initialTime, false]);
  
  // Destructuring a tuple allows TypeScript to know the exact type of each element
  const [time, isRunning] = state; 
  
  // ... timer logic
  return [time, () => setState([time, true])];
};
```

### 3\. Map for Efficient Lookups in State

`Map` is ideal for component state where you need to look up data by a complex key or perform frequent key lookups, especially if the keys are objects.

```tsx
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// Map key is string (Task ID), Map value is Task object
type TaskMap = Map<string, Task>;

const TaskManager: React.FC = () => {
  const [tasks, setTasks] = useState<TaskMap>(new Map());

  const addTask = (task: Task) => {
    // Creating a new Map to ensure immutability for React state update
    setTasks(prev => new Map(prev).set(task.id, task));
  };

  const getTaskTitle = (id: string) => {
    // Efficient O(1) lookup
    return tasks.get(id)?.title || 'Task not found';
  };
  
  // Use Map.values() and Array.from() to iterate for rendering
  return (
    <ul>
      {Array.from(tasks.values()).map(task => (
        <li key={task.id}>{task.title}</li>
      ))}
    </ul>
  );
};
```

### 4\. Dictionary/Object for Component Props

Dictionary types (via interfaces or `Record`) are commonly used to type component **props** where the key represents a configuration property.

```tsx
// Using Record utility type for a Dictionary with string keys and JSX.Element values
type IconMap = Record<string, React.ReactNode>;

interface IconProps {
  icons: IconMap;
  currentIconKey: string;
}

const IconDisplay: React.FC<IconProps> = ({ icons, currentIconKey }) => {
  // Accessing the dictionary value via string index
  const IconComponent = icons[currentIconKey]; 

  if (!IconComponent) {
    return <span>Unknown Icon</span>;
  }

  return <div>{IconComponent}</div>;
};

// Example usage
// const myIcons: IconMap = { home: <HomeIcon />, settings: <SettingsIcon /> };
// <IconDisplay icons={myIcons} currentIconKey="home" />
```

# React _Rendering_ and _Commit_ Steps

#### Step 1: `Triggering` a render (delivering the guest’s order to the kitchen)

- Two reasons for a component to render
    1. It’s the component’s **initial render.**
    2. The component’s (or one of its ancestors’) **state has been updated.**

#### Step 2: `Rendering` the component (preparing the order in the kitchen)

#### Step 3: `Committing` to the `DOM` (placing the order on the table)

---

**React Rendering Cycle (T.R.C)Visual Explain:**

![React Rendering Cycle](https://cdn.hashnode.com/res/hashnode/image/upload/v1676932402382/234dbd06-c2d2-4a4f-b3d4-9ad9733036bf.png?auto=compress,format&format=webp)

1. **Triggering** a render( taking the order from customers and presenting to cooks)

2. **Rendering** the component (preparing the order in the kitchen)

3. **Committing** to the DOM( placing the order in the table)

4. In the above image the waiter take an order form customer and present to cooks which is a triggering a render

5. Cook starts to prepare the order in the kitchen which is a Rendering the component

6. At last Waiter takes the order and place it on the customer table which is committing to the DOM

---


> **Reconciliation** occur during re-render when props changes or state update it trigger the render() function one more time and return a different tree of React element.
>> ![Reconciliation](https://cdn.hashnode.com/res/hashnode/image/upload/v1676935954080/6dc4a9cb-b3d4-45ff-b492-df0c450302a6.jpeg?auto=compress,format&format=webp)


**According to** [](https://reactjs.org/docs/reconciliation.html)[reactjs.org](http://reactjs.org) React implements a heuristic 0(n) algorithm based on this assumptions:

-   Elements that changed types must be recreated
    
-   Changes within the attributes of an element are replaced, without unmounting the element
    
-   Upgrades within the element's children recreate all children
    
-   Updates within child elements that use key as attributes are compared and only new items are represented (Tips: Don't use Math.random() while assigning the keys to element)

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676951783467/517f121c-8599-4872-8692-79e6b04b8022.png?auto=compress,format&format=webp)

---

**below from Gemini AI**

**The React rendering and commit process can be broken down into three main steps:**

- **Triggering a Render:**

    This initiates the rendering process. A render is triggered in two primary scenarios:

  - **Initial Render:** When the application first loads, the root component is rendered, and React builds the initial UI.
  - **State or Prop Updates:** When a component's state is updated using `setState` (for class components) or `useState` (for functional components), or when its props change, React queues a re-render for that component and its descendants(الأحفاد - هابط).

- **Rendering the Component:**

    During this phase, React performs the following:

  - **Calling Component Functions:** React calls the `render` method for class components or _executes the function body for functional components_. This calculates the new virtual `DOM` representation of the component based on its current state and props.
  - **Reconciliation (ملائمة - تسوية):** React compares the newly generated virtual `DOM` tree with the previous one. This "diffing" process identifies the minimal set of changes required to update the actual `DOM`. This is a crucial step for performance, as it avoids unnecessary `DOM` manipulations.

- **Committing to the `DOM`:**

    This is the final phase where the identified changes are applied to the real `DOM`:

  - **DOM Updates:** React efficiently applies the calculated changes (additions, updates, or deletions of elements) to the browser's actual `DOM`.
  - **Browser Repaint:** Once the `DOM` is updated, the browser detects these changes and repaints the screen to reflect the new UI, making the changes visible to the user.

In essence, React's rendering and commit process ensures that UI updates are efficient and performant(أداء) by leveraging(الإستفادة) the virtual `DOM` and a smart reconciliation algorithm to minimize direct `DOM` manipulations.

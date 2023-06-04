# React-hooks


### 1. useMemo() hook
useMemo() is a built-in React hook that accepts 2 arguments — a function compute that computes a result, and the depedencies array:
```javascript
const memoizedResult = useMemo(compute, dependencies);
```
During initial rendering, useMemo(compute, dependencies) invokes compute, memoizes the calculation result, and returns it to the component.

If the dependencies don't change during the next renderings, then useMemo() doesn't invoke compute, but returns the memoized value.

But if the dependencies change during re-rendering, then useMemo() invokes compute, memoizes the new value, and returns it.

That's the essence of useMemo() hook.

If your computation callback uses props or state values, then be sure to indicate these values as dependencies:

```javascript
const memoizedResult = useMemo(() => {
  return expensiveFunction(propA, propB);
}, [propA, propB]);
```
Now let's see how useMemo() works in an example.


```javascript

import React, { useState, useMemo } from 'react'

function Counter() {
	const [counterOne, setCounterOne] = useState(0)
	const [counterTwo, setCounterTwo] = useState(0)

	const incrementOne = () => {
		setCounterOne(counterOne + 1)
	}

	const incrementTwo = () => {
		setCounterTwo(counterTwo + 1)
  }

  const isEven = useMemo(() => {
    let i = 0
    while (i < 2000000000) i++
    return counterOne % 2 === 0
  }, [counterOne])

	return (
		<div>
			<div>
        <button onClick={incrementOne}>Count One - {counterOne}</button>
        <span>{isEven ? 'Even' : 'Odd'}</span>
			</div>
			<div>
        <button onClick={incrementTwo}>Count Two - {counterTwo}</button>
			</div>
		</div>
	)
}

export default Counter
```

### useCallback() Hooks
Improving performance In React applications includes preventing unnecessary renders and reducing the time a render takes to propagate. useCallback() can help us prevent some unnecessary renders and therefore gain a performance boost.

In this post, we’ll take a close look at the useCallback() hook and how to properly use it to write better React code.

The useCallback() hook
Going back to React, when a component re-renders, every function inside of the component is recreated and therefore these functions’ references change between renders.

useCallback(callback, dependencies) will return a memoized instance of the callback that only changes if one of the dependencies has changed. This means that instead of recreating the function object on every re-render, we can use the same function object between renders.

```javascript
const memoized = useCallback(() => {
   // the callback function to be memoized
 },
  // dependencies array
[]);
```

### Use case scenario
Passing callbacks to optimized child components
useCallback is especially useful to prevent unnecessary renders when passing callbacks to optimized child components that rely on reference equality. Imagine you have a component that renders a list of items:

```javascript

function MyList({ handler }) {
  const items = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
  useEffect(() => {
    console.log("Child Component redered");
  }, []);

  return (
    <>
      {items.map((item, index) => {
        return (
          <div key={index} onClick={handler}>
            {item}
          </div>
        );
      })}
    </>
  );
}

export default React.memo(MyList);

```
To prevent unnecessary expensive list re-renderings, you wrap it into React.memo().

The parent component<ParentComponent> provides a handler function to the child component <MyList>:

```javascript
	
export default function ParentComponent() {
  const [state, setState] = useState(false);
  const [dep] = useState(false);
  console.log("Parent Component redered");

  const handler = useCallback(
    (event) => {
      console.log("You clicked ", event.currentTarget);
    },
    // eslint-disable-next-line react-hooks/exhaustive-deps
    [dep],
  );
  const statehanddler = () => {
    setState(!state);
  };
  return (
    <>
      <button onClick={statehanddler}>Change State Of Parent Component</button>
      <MyList handler={handler} />
    </>
  );
```
	
handler callback is memoized by useCallback(). As long as dep is the same, `useCallback()` returns the same function object. When <ParentComponent> re-renders, the handler function object remains the same and doesn’t break the memorization of `<MyList>`.
	
	
 ### When you should not use useCallback()
	
Let’s make sure we don’t go overboard. useCallback() has its downsides, primarily code complexity. There are a lot of situations where adding useCallback() doesn’t make sense and you just have to accept function recreation.

`useCallback()` has its performance drawbacks, as it still has to run on every component re-render.
In this example, useCallback() is not helping optimization, since we’re creating the clickHandler function on every render anyways; actually, the optimization costs more than not having the optimization.
	
```javascript
export default function MyComponent() {
    
  // poor usage of useCallback()
  const clickHandler = useCallback(() => {
    // handle the click event
  }, []);

  return <ButtonWrapper onClick={clickHandler} />;
}

const ButtonWrapper = ({ clickHandler }) => {
  return <button onClick={clickHandler}>Child Component</button>;
};
```
	
### Conclusion
useCallback(callback, dependencies) can be used like useMemo(), but it memoizes functions instead of values, to prevent recreation upon every render. allowing you to avoid unnecessary re-rendering which makes your application more efficient.

When thinking about performance upgrades, always measure (or profile) your component speed before the optimization process. Optimization increases complexity, and as a developer, you should always make sure that the tradeoff is worthwhile.
	
### React useRef() Hook

React.useRef() hook to create persisted mutable values (also known as references or refs), as well as access DOM elements.
1. Mutable values
useRef(initialValue) is a built-in React hook that accepts one argument as the initial value and returns a reference (aka ref). A reference is an object having a special property current.
	
```javascript
	import { useRef } from 'react';

function MyComponent() {
  const initialValue = 0;
  const reference = useRef(initialValue);

  const someHandler = () => {
    // Access reference value:
    const value = reference.current;

    // Update reference value:
    reference.current = newValue;
  };

  // ...
}
```
reference.current accesses the reference value, and reference.current = newValue updates the reference value. Pretty simple.

There are 2 rules to remember about references:

The value of the reference is persisted (remains unchanged) between component re-renderings;
Updating a reference doesn't trigger a component re-rendering.
	
1.1 Use case: logging button clicks
The component LogButtonClicks uses a reference to store the number of clicks on a button:
	
```javascript
	import { useRef } from 'react';

function LogButtonClicks() {
  const countRef = useRef(0);
  
  const handle = () => {
    countRef.current++;
    console.log(`Clicked ${countRef.current} times`);
  };

  console.log('I rendered!');

  return <button onClick={handle}>Click me</button>;
}
```
const countRef = useRef(0) creates a reference countRef initialized with 0.

When the button is clicked, handle callback is invoked and the reference value is incremented: countRef.current++. Then the reference value is logged to the console.

Updating the reference value countRef.current++ doesn't trigger component re-rendering. This is demonstrated by the fact that 'I rendered!' is logged to the console just once, at initial rendering, and no re-rendering happens when the reference is updated.
	
### Reference and state diff
Let's reuse the component LogButtonClicks from the previous section, but this time use useState() hook to count the number of button clicks:
	
	
```javascript
	import { useState } from 'react';

function LogButtonClicks() {
  const [count, setCount] = useState(0);
  
  const handle = () => {
    const updatedCount = count + 1;
    console.log(`Clicked ${updatedCount} times`);
    setCount(updatedCount);
  };

  console.log('I rendered!');

  return <button onClick={handle}>Click me</button>;
}
```
	
Open the demo and click the button. Each time you click, you will see in the console the message 'I rendered!' — meaning that each time the state is updated, the component re-renders.

So, the 2 main differences between reference and state:

Updating a reference doesn't trigger re-rendering, while updating the state makes the component re-render;
The reference update is synchronous (the updated reference value is available right away), while the state update is asynchronous (the state variable is updated after re-rendering).
From a higher point of view, references store infrastructure data of side-effects, while the state stores information that is directly rendered on the screen.
	
### 1.2 Use case: implementing a stopwatch
You can store inside a reference infrastructure data of side effects: timer ids, socket ids, etc.

The component Stopwatch uses setInterval(callback, time) timer function to increase each second the counter of a stopwatch. The timer id is stored in a reference timerIdRef:

```javascript
import { useRef, useState, useEffect } from 'react';

function Stopwatch() {
  const timerIdRef = useRef(0);
  const [count, setCount] = useState(0);

  const startHandler = () => {
    if (timerIdRef.current) { return; }
    timerIdRef.current = setInterval(() => setCount(c => c+1), 1000);
  };

  const stopHandler = () => {
    clearInterval(timerIdRef.current);
    timerIdRef.current = 0;
  };

  useEffect(() => {
    return () => clearInterval(timerIdRef.current);
  }, []);

  return (
    <div>
      <div>Timer: {count}s</div>
      <div>
        <button onClick={startHandler}>Start</button>
        <button onClick={stopHandler}>Stop</button>
      </div>
    </div>
  );
}
```
startHandler() function, which is invoked when the Start button is clicked, starts the timer and saves the timer id in the reference timerIdRef.current = setInterval(...).

To stop the stopwatch user clicks Stop button. The Stop button handler stopHandler accesses the timer id from the reference and stops the timer clearInterval(timerIdRef.current).

Additionally, if the component unmounts while the stopwatch is active, the cleanup function of useEffect() is going to stop the timer too.

In the stopwatch example, the reference was used to store the infrastructure data — the active timer id.

Side challenge: can you improve the stopwatch by adding a Reset button? Share your solution in a comment below!

### 2. Accessing DOM elements
	
Another useful application of the useRef() hook is to access DOM elements directly. This is performed in 3 steps:

Define the reference to access the element const elementRef = useRef();
Assign the reference to ref attribute of the element: <div ref={elementRef}></div>;
After mounting, elementRef.current points to the DOM element.

```javascript 
	import { useRef, useEffect } from 'react';

function AccessingElement() {
  const elementRef = useRef();

   useEffect(() => {
    const divElement = elementRef.current;
    console.log(divElement); // logs <div>I'm an element</div>
  }, []);

  return (
    <div ref={elementRef}>
      I'm an element
    </div>
  );
}
```
	
2.1 Use case: focusing on an input
You would need to access DOM elements, for example, to focus on the input field when the component mounts.

To make it work you'll need to create a reference to the input, assign the reference to ref attribute of the tag, and after mounting call the special method element.focus() on the element.

Here's a possible implementation of the <InputFocus> component:

```javascript
import { useRef, useEffect } from 'react';

function InputFocus() {
  const inputRef = useRef();

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return (
    <input 
      ref={inputRef} 
      type="text" 
    />
  );
}
```
	
const inputRef = useRef() creates a reference to hold the input element.

inputRef is then assigned to ref attribute of the input field: <input ref={inputRef} type="text" />.

React then, after mounting, sets inputRef.current to be the input element. Inside the callback of useEffect() you can set the focus to the input programmatically: inputRef.current.focus().

Tip: if you want to learn more about useEffect(), I highly recommend checking my post A Simple Explanation of React.useEffect().

### Ref is null on initial rendering
During initial rendering, the reference supposed to hold the DOM element is empty:
	
```javascript
import { useRef, useEffect } from 'react';

function InputFocus() {
  const inputRef = useRef();

  useEffect(() => {
    // Logs `HTMLInputElement` 
    console.log(inputRef.current);

    inputRef.current.focus();
  }, []);

  // Logs `undefined` during initial rendering
  console.log(inputRef.current);

  return <input ref={inputRef} type="text" />;
}
```
During initial rendering React still determines the output of the component, so there's no DOM structure created yet. That's why inputRef.current evaluates to undefined during initial rendering.

useEffect(callback, []) hook invokes the callback right after mounting when the input element has already been created in DOM.

callback function of the useEffect(callback, []) is the right place to access inputRef.current because it is guaranteed that the DOM is constructed.

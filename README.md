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


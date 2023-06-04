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

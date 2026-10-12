# useEffect in React Native

## 1. useEffect

Runs side effects, which are things outside of rendering: fetching data, timers, subscriptions, listeners, and storage reads.

```jsx
useEffect(() => {
    // runs AFTER the component renders
    return () => {
        // optional cleanup: runs before the next effect or on unmount
    };
}, [dependencies]);
```

## 2. The Dependency Array

The dependency array controls when the effect runs:

```jsx
useEffect(() => { /* ... */ });           // after EVERY render
useEffect(() => { /* ... */ }, []);       // once, after the first render (mount)
useEffect(() => { /* ... */ }, [userId]); // on mount and whenever userId changes
```

## 3. Common Mistakes

| Mistake                                                | Result                                                     | Fix                               |
| ------------------------------------------------------ | ---------------------------------------------------------- | --------------------------------- |
| Missing dependency                                     | Stale values (old props or state used inside the effect)   | List every value the effect reads |
| Setting state in an effect with no dependency array    | Infinite render loop                                       | Add a dependency array            |
| Forgetting cleanup                                     | Memory leaks, duplicate listeners                          | Return a cleanup function         |
| `useEffect(async () => ...)`                           | Warning, since it returns a Promise instead of a cleanup   | Define an inner async function    |
| Using an effect for derived data                       | Extra renders                                              | Compute it during render instead  |

# React.memo

React.memo is a wrapper for components that makes React skip re-rendering them when their props haven’t changed.

By default, when a parent re-renders, all its children re-render, even with identical props. React.memo opts a component out of that. Before re-rendering it, React does a shallow comparison of the old and new props. If every prop is equal, React reuses the last result.

`useMemo` keeps objects and arrays stable, `useCallback` keeps functions stable. `React.memo` is the component that actually benefits from those stable references, It only blocks re-renders caused by the parent.

```jsx
const Child = React.memo(function Child({ name }) {
    console.log('Child rendered');
    return <Text>{name}</Text>;
});

function Parent() {
    const [count, setCount] = useState(0);
    return (
        <View>
            <Button title={`Count ${count}`} onPress={() => setCount(count + 1)} />
            <Child name="Mohammad" />
        </View>
    );
}
```

Pressing the button re-renders `Parent`, but `Child` is skipped: `name` is the same string each time. Without `React.memo`, it would log on every press.

## The full picture

| Tool | Stabilizes | Used on |
|---|---|---|
| `React.memo` | Skips re-render if props are equal | A component |
| `useMemo` | Objects, arrays, expensive values | The parent side |
| `useCallback` | Functions | The parent side |

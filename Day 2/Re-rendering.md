# Re-rendering

A component is a function. A render means React calls that function again to get the new UI description. In React Native this does not redraw the whole screen. React compares the new output with the old one and only updates the native views that changed.

## What triggers a re-render

- Its own state changes (setState).
- Its parent re-renders, which re-renders all its children by default, even if their props didn’t change.
- Context it consumes changes.
- A hook it uses changes (a custom hook’s state, a store subscription, etc.).

## Example 1: state change

```jsx
export default function Counter() {
    const [count, setCount] = useState(0);
    console.log('Counter rendered');

    return (
        <View>
            <Text>{count}</Text>
            <Button title="+" onPress={() => setCount(count + 1)} />
        </View>
    );
}
```

Each press calls `setCount`, React re-runs `Counter`, and the log prints every time.

## Example 2: parent re-render drags the children along

```jsx
function Child() {
    console.log('Child rendered');
    return <Text>I have no props</Text>;
}

export default function Parent() {
    const [count, setCount] = useState(0);
    return (
            <View>
                <Button title={`Count: ${count}`} onPress={() => setCount(count + 1)} />
                <Child />
            </View>
    );
}
```

`Child` takes no props and uses no state, yet it logs on every press because `Parent` re-rendered.

## Example 3: changing a variable does NOT re-render

```jsx
export default function Broken() {
    let count = 0;
    return (
        <View>
            <Text>{count}</Text>
            <Button title="+" onPress={() => { count++; console.log(count); }} />
        </View>
    );
}
```

The variable changes (you see it in the console) but the screen stays at 0. React only re-renders when it is told to.

## Example 4: mutating state doesn’t work either

```jsx
const [items, setItems] = useState(['a']);

// Wrong: same array reference, React sees no change
items.push('b');
setItems(items);

// Right: new array
setItems([...items, 'b']);
```

React compares state by reference (`Object.is`). A new array or object means “changed”, while the same one mutated means “nothing happened”.

## Example 5: setting the same value

```jsx
setCount(5); // already 5, React usually bails out and skips the re-render
```

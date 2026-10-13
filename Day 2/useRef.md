# useRef

`useRef` gives you a mutable box that lives as long as the component does. It returns an object `{ current: ... }`, and React keeps the same object across renders.

The key difference from state: changing `ref.current` does **NOT** trigger a re-render.

```jsx
const ref = useRef(initialValue);
ref.current;          // read
ref.current = 5;      // write, no re-render
```

## The two main uses in React Native

- Holding a reference to a native component (to call its methods, like focusing a `TextInput`).
- Storing a value that must survive re-renders but shouldn’t cause one (timers, previous values, counters, flags).

## Example 1: focusing a TextInput

```jsx
export default function LoginForm() {
    const passwordRef = useRef(null);

    return (
        <View>
        <TextInput
            placeholder="Email"
            returnKeyType="next"
            onSubmitEditing={() => passwordRef.current?.focus()}
        />
        <TextInput
            ref={passwordRef}
            placeholder="Password"
            secureTextEntry
        />
        <Button title="Login" onPress={() => {}} />
        </View>
    );
}
```

Pressing “next” on the keyboard calls `.focus()` on the second input. `passwordRef.current` is the actual `TextInput` instance.

## Example 2: scrolling a list

```jsx
const listRef = useRef(null);

<FlatList ref={listRef} data={data} renderItem={renderItem} />
<Button title="Top" onPress={() => listRef.current?.scrollToOffset({ offset: 0 })} />
```

## Example 3: remembering the previous value

```jsx
function usePrevious(value) {
    const ref = useRef();
    useEffect(() => {
        ref.current = value;
    });
    return ref.current;
}

const prevCount = usePrevious(count);
```

## useRef vs useState

| | `useState` | `useRef` |
|---|---|---|
| Change triggers re-render | Yes | No |
| Value persists between renders | Yes | Yes |
| Use for | Anything shown on screen | Things the UI doesn’t display |
| Update style | `setX(new)` | `ref.current = new` |

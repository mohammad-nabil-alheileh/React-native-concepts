# Custom hooks

A custom hook is a regular JavaScript function whose name starts with `use` and that calls other hooks inside it (`useState`, `useEffect`, `useRef`, etc.). It lets you extract and reuse stateful logic across components.

## Key points

- It shares the logic, not the state. Each component that calls the hook gets its own separate copy of the state.
- The `use` prefix is required. It tells React (and the linter) to enforce the Rules of Hooks.
- It can return anything: a value, an array, an object, or functions.
- It re-runs on every render of the component that uses it, just like built-in hooks. If its state changes, that component re-renders.

## Example 1: useToggle

```jsx
function useToggle(initial = false) {
    const [on, setOn] = useState(initial);
    const toggle = useCallback(() => setOn(v => !v), []);
    return [on, toggle];
}

// Usage
function Settings() {
    const [notifications, toggleNotifications] = useToggle(true);
    const [darkMode, toggleDarkMode] = useToggle(false);

    return (
        <View>
            <Switch value={notifications} onValueChange={toggleNotifications} />
            <Switch value={darkMode} onValueChange={toggleDarkMode} />
        </View>
    );
}
```

Two calls, two independent states.

## Example 4: useKeyboardVisible

```jsx
function useKeyboardVisible() {
    const [visible, setVisible] = useState(false);

    useEffect(() => {
        const show = Keyboard.addListener('keyboardDidShow', () => setVisible(true));
        const hide = Keyboard.addListener('keyboardDidHide', () => setVisible(false));
        return () => { show.remove(); hide.remove(); };
    }, []);

    return visible;
}
```

## Custom hook vs plain function vs component

| | Can use hooks? | Returns | Use for |
|---|---|---|---|
| Plain function | No | Anything | Pure logic (formatting, math) |
| Custom hook | Yes | Anything | Reusable stateful logic |
| Component | Yes | JSX | Reusable UI |

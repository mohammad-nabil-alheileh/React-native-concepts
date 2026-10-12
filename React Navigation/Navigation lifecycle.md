# Navigation Lifecycle

In a web page, leaving a page unmounts it. In React Navigation, screens often **stay mounted** when you leave them:

- In a stack, the screens below the top one stay mounted.
- In tabs and drawers, visited screens stay mounted.

So `useEffect(() => {...}, [])` runs once (on mount) and does **not** run again when you come back to the screen. It also doesn't clean up when you navigate away.

## The Events

| Event                                | When it fires                                          |
| ------------------------------------ | ------------------------------------------------------ |
| `focus`                              | The screen becomes focused                             |
| `blur`                               | The screen loses focus                                 |
| `beforeRemove`                       | A screen is about to be removed (you can block it)     |
| `state`                              | The navigator's state changed                          |
| `transitionStart` / `transitionEnd`  | Stack transition animations (stack navigators)         |
| `tabPress`                           | A tab was pressed (tab navigators)                     |

## 1. Listening with navigation.addListener

```jsx
React.useEffect(() => {
    const unsubscribe = navigation.addListener('focus', () => {
        loadPlants();   // runs every time Home becomes active
    });
    return unsubscribe;
}, [navigation]);
```

- `addListener('focus', ...)` registers a function that runs every time the screen gets focus, not only the first time.
- It returns an `unsubscribe` function. Returning it from `useEffect` removes the listener when the screen unmounts, so it doesn't run after the screen is gone.

## 2. useFocusEffect

A hook that works like `useEffect`, but runs when the screen is **focused** and cleans up when it **blurs**.

```jsx
function HomeScreen() {
    useFocusEffect(
        React.useCallback(() => {
            // runs when the screen gains focus (including the first time)
            console.log('focused');

            return () => {
                // cleanup: runs when the screen loses focus
                console.log('blurred');
            };
        }, [])
    );

    return <Text>Home</Text>;
}
```

- The function body runs on every focus, and the returned function runs on every blur.
- This makes it good for things that should only run while the screen is visible: timers, polling, subscriptions.

The callback must be wrapped in `useCallback`. Without it, the function is a new one on every render, so the effect re-runs on every render.

## 3. useIsFocused

Use it to render conditionally.

```jsx
function CameraScreen() {
    const isFocused = useIsFocused();
    return isFocused ? <Camera /> : null;   // only mount the camera while visible
}
```

`isFocused` is `true` while the screen is visible and `false` otherwise. The screen **re-renders** each time it changes, so the camera is mounted when you arrive and unmounted when you leave.

## 4. Blocking Leave with beforeRemove

```jsx
function EditScreen({ navigation }) {
    const [hasUnsaved, setHasUnsaved] = useState(false);

    React.useEffect(() => {
        const unsubscribe = navigation.addListener('beforeRemove', (e) => {
            if (!hasUnsaved) return;          // nothing to protect, let it go

            e.preventDefault();               // stop the navigation

            Alert.alert('Discard changes?', 'You have unsaved changes.', [
                { text: 'Stay', style: 'cancel' },
                {
                    text: 'Discard',
                    style: 'destructive',
                    onPress: () => navigation.dispatch(e.data.action), // continue the original action
                },
            ]);
        });
        return unsubscribe;
    }, [navigation, hasUnsaved]);

    return <TextInput onChangeText={() => setHasUnsaved(true)} />;
}
```

- `beforeRemove` fires when the user tries to leave the screen (back button, header back, `goBack()`).
- If nothing is unsaved, the listener returns early and the user leaves normally.
- `e.preventDefault()` cancels the leave, and the alert asks the user what to do.
- `e.data.action` is the original action (like "go back"). Dispatching it on **Discard** finishes the leave the user started.
- `hasUnsaved` is in the dependency array, so the listener is re-created with the latest value.

> **Note:** React Navigation 7 also has a `usePreventRemove(hasUnsaved, callback)` hook that does the same thing in less code. With the native stack, it can't block the iOS swipe-back gesture, so set `gestureEnabled: false` on that screen.

## 5. Which One to Use

| Need                                                          | Use                                         |
| ------------------------------------------------------------- | ------------------------------------------- |
| Run code once when the component first mounts                 | `useEffect(..., [])`                        |
| Run code every time the screen gets focus, with cleanup on blur | `useFocusEffect`                          |
| Just react to focus/blur events directly                      | `navigation.addListener('focus' / 'blur')`  |
| Render something only while visible                           | `useIsFocused`                              |
| Confirm before leaving (unsaved changes)                      | `beforeRemove` listener                     |

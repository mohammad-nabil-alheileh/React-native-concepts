# useNavigation

`useNavigation` is a hook that gives you the `navigation` object from **any** component, not just screen components.

Screens registered in a navigator receive `navigation` as a prop automatically. But a component nested deeper (a button, a card, a header) does not. Without the hook, you would have to pass `navigation` down through props manually.

It returns the same object you'd get as a prop, so everything works the same: `navigate`, `push`, `goBack`, `replace`, `setOptions`, and so on.

## 1. Example: A Button Inside a Screen

```jsx
function GoToDetailsButton() {
    const navigation = useNavigation();

    return (
        <Button
            title="Open Details"
            onPress={() => navigation.navigate('Details')}
        />
    );
}

function HomeScreen() {
    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Home</Text>
            <GoToDetailsButton />
        </View>
    );
}
```

`HomeScreen` doesn't pass anything to `GoToDetailsButton`. The button gets `navigation` by itself through the hook.

## 2. Example: A Custom Header Button

```jsx
function HeaderBackButton() {
    const navigation = useNavigation();
    return <Button title="Back" onPress={() => navigation.goBack()} />;
}

<Stack.Screen
    name="Details"
    component={DetailsScreen}
    options={{ headerLeft: () => <HeaderBackButton /> }}
/>
```

Components rendered in the header aren't screens, so they don't get the `navigation` prop. The hook still works there.

> **Note:** `useNavigation` only works inside `NavigationContainer`. Calling it outside (for example, in a component rendered above the container) throws an error.

## 3. navigation Prop vs useNavigation

|                   | `navigation` prop                         | `useNavigation()`                    |
| ----------------- | ----------------------------------------- | ------------------------------------ |
| **Available in**  | Screen components only                    | Any component inside the container   |
| **Prop drilling** | Needed for children                       | Not needed                           |
| **Reusability**   | Component depends on parent passing it    | Self-contained                       |
| **Typical use**   | Simple screens                            | Buttons, cards, headers, shared UI   |

# Native Stack

A stack navigator keeps screens in a pile, like cards:

- Navigating to a screen **pushes** it on top.
- Going back **pops** it off.
- Only the top screen is visible, but the screens below stay alive, so their state is preserved.

## 1. Basic Example

```jsx
const Stack = createNativeStackNavigator();

function HomeScreen({ navigation }) {
    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Home</Text>
            <Button title="Go to Details" onPress={() => navigation.navigate('Details')} />
        </View>
    );
}

function DetailsScreen({ navigation }) {
    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Details</Text>
            <Button title="Go back" onPress={() => navigation.goBack()} />
        </View>
    );
}

export default function App() {
    return (
        <NavigationContainer>
            <Stack.Navigator initialRouteName="Home">
                <Stack.Screen name="Home" component={HomeScreen} />
                <Stack.Screen name="Details" component={DetailsScreen} />
            </Stack.Navigator>
        </NavigationContainer>
    );
}
```

- Every screen receives a `navigation` prop.
- A header with a back button appears automatically on Details.
- The Android hardware back button also pops the stack.

## 2. Navigation Methods

| Method            | What it does                                              |
| ----------------- | --------------------------------------------------------- |
| `navigate('Name')`| Go to a screen                                            |
| `push('Name')`    | Always add a new copy on top                              |
| `goBack()`        | Pop the current screen                                    |
| `pop(n)`          | Pop `n` screens                                           |
| `popTo('Name')`   | Go back to an existing screen in the stack                |
| `popToTop()`      | Go back to the first screen                               |
| `replace('Name')` | Swap the current screen (you can't go back to it)         |

## 3. Customizing the Header with Options

```jsx
<Stack.Screen
    name="Details"
    component={DetailsScreen}
    options={{
        title: 'Product Details',
        headerStyle: { backgroundColor: '#1b5e20' },
        headerTintColor: '#fff',
        headerRight: () => <Button title="Info" onPress={() => {}} />,
    }}
/>
```

To apply the same options to all screens, use `screenOptions` on the navigator:

```jsx
<Stack.Navigator screenOptions={{ headerTintColor: '#1b5e20' }}>
```

Per-screen `options` override `screenOptions`.

To change options from inside a screen:

```jsx
React.useLayoutEffect(() => {
    navigation.setOptions({ title: 'Updated title' });
}, [navigation]);
```

`useLayoutEffect` runs before the screen is painted, so the user never sees the old title flash.

## 4. Useful Options

| Option           | Example value                                       | What it does                                                         |
| ---------------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| `headerShown`    | `false`                                             | Hides the header (then handle the safe area yourself)                |
| `animation`      | `'slide_from_bottom'` \| `'fade'` \| `'none'`       | Changes the transition animation when the screen opens               |
| `presentation`   | `'modal'`                                           | Slides the screen up like a modal                                    |
| `gestureEnabled` | `false`                                             | Disables swipe-back (iOS)                                            |
| `contentStyle`   | `{ backgroundColor: '#fff' }`                       | Styles the screen background                                         |

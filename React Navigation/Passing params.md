# Passing Params

Params are data you send to a screen when you navigate to it. They live in that screen's `route.params`, which you read with the `route` prop or the `useRoute` hook.

## 1. Sending and Reading Params

```jsx
function HomeScreen({ navigation }) {
    return (
        <Button
            title="Open Rose"
            onPress={() => navigation.navigate('Details', { id: 42, title: 'Rose' })}
        />
    );
}

function DetailsScreen({ route }) {
    const { id, title } = route.params;
    return <Text>{title} (#{id})</Text>;
}
```

- `navigate('Details', { id: 42, title: 'Rose' })`: the second argument is the params object.
- `route.params` in `DetailsScreen` is that same object: `{ id: 42, title: 'Rose' }`.

## 2. Default Values

Params can have default values with `initialParams`:

```jsx
<Stack.Screen
    name="Details"
    component={DetailsScreen}
    initialParams={{ id: 0, title: 'Unknown' }}
/>
```

They are used only when the screen opens without params. Params passed by `navigate` are merged over them.

Also guard when reading, because `route.params` is `undefined` if the screen opens with no params and no `initialParams`:

```jsx
const { id, title } = route.params ?? {};
```

## 3. Updating Params After the Screen Is Open

```jsx
navigation.setParams({ title: 'Renamed' });
```

`setParams` merges the new values into the current params, so `id` stays the same and only `title` changes.

## 4. Navigating to a Screen That Is Already Open

When you call `navigate` to a screen that already exists, its params are **replaced**, not merged. To keep the old values, ask for a merge:

```jsx
navigation.navigate('Details', { title: 'New' }, { merge: true });
```

## 5. Passing Data Back to the Previous Screen

Params flow forward by default. To send a result back (for example, picking a plant from a list), navigate to the previous screen with params:

```jsx
function PickPlantScreen({ navigation }) {
    return (
        <Button
            title="Pick Monstera"
            onPress={() => navigation.popTo('Home', { selectedPlant: 'Monstera' }, { merge: true })}
        />
    );
}

function HomeScreen({ route }) {
    const selectedPlant = route.params?.selectedPlant;

    useEffect(() => {
        if (selectedPlant) {
            // use the result, e.g. save it or show it
        }
    }, [selectedPlant]);

    return <Text>Selected: {selectedPlant ?? 'none'}</Text>;
}
```

`popTo('Home', ...)` closes the picker and goes back to the existing Home screen, and `{ merge: true }` adds `selectedPlant` to Home's params without removing the others.

## 6. Summary

| Way                                        | When to use it                                         | Merge or replace?        |
| ------------------------------------------ | ------------------------------------------------------ | ------------------------ |
| `navigate('Screen', params)`               | Open a screen with data                                | Replace                  |
| `initialParams` on `<Stack.Screen>`        | Default values when no params are passed               | Passed params merged over|
| `setParams(params)`                        | Change params of the current screen                    | Merge                    |
| `navigate('Screen', params, { merge: true })` | Update a screen that is already open                | Merge                    |
| `popTo('Screen', params, { merge: true })` | Send a result back to a previous screen                | Merge                    |

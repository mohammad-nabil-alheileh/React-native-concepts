# Zustand

Zustand is a tiny state library. You create a store (an object holding state plus the functions that change it) and read it in components with a hook. There is no Provider, so no wrapping the app and no provider nesting.

## Three core ideas

- **Store:** one place that holds state.
- **Actions:** functions inside the store that update state.
- **Selectors:** functions that pick which slice of the store a component subscribes to.

Selectors fix the main weakness of Context. A component re-renders only when the selected value changes, not when anything in the store changes.

## Example 1: a basic store

```jsx
const useCounterStore = create((set) => ({
    // state
    count: 0,

    // actions
    increment: () => set((state) => ({ count: state.count + 1 })),
    reset: () => set({ count: 0 }),
}));
```

- `create` returns a hook (`useCounterStore`).
- `set` merges the new values into the state (a shallow merge, so you don’t need to spread the old state).
- Use the function form `set((state) => ...)` when the new value depends on the old one.

```jsx
function CounterText() {
    const count = useCounterStore((state) => state.count); // selector
    return <Text>{count}</Text>;
}

function CounterButtons() {
    const increment = useCounterStore((state) => state.increment);
    const reset = useCounterStore((state) => state.reset);
    return (
        <View>
            <Button title="+" onPress={increment} />
            <Button title="Reset" onPress={reset} />
        </View>
    );
}
```

Two separate components share the same `count` with no props and no Provider. Action functions never change, so `CounterButtons` never re-renders when `count` changes.

## Example 2: selectors and re-renders

```jsx
const useStore = create((set) => ({
    user: { name: 'Mohammad' },
    theme: 'light',
    cart: [],
    addToCart: (item) => set((s) => ({ cart: [...s.cart, item] })),
}));

function ThemeLabel() {
    const theme = useStore((s) => s.theme); // subscribes to theme only
    console.log('ThemeLabel rendered');
    return <Text>{theme}</Text>;
}
```

When `addToCart` runs, `cart` changes but `theme` does not, so `ThemeLabel` does not re-render.

## Example 3: the selector mistake (don’t select the whole store)

```jsx
// Bad: subscribes to EVERYTHING, so it re-renders on any change
const store = useStore();

// Good: pick only what you need
const theme = useStore((s) => s.theme);
```

## Example 4: selecting several values

```jsx
// Bad: returns a NEW object every time, so Object.is says "changed"
// and the component re-renders on every store update
const { user, theme } = useStore((s) => ({ user: s.user, theme: s.theme }));

// Good: useShallow compares the object's fields instead
import { useShallow } from 'zustand/react/shallow';

const { user, theme } = useStore(
    useShallow((s) => ({ user: s.user, theme: s.theme }))
);

// Also fine: one selector per value
const user = useStore((s) => s.user);
const theme = useStore((s) => s.theme);
```

Same reference-equality trap as before: a selector must not create a new object or array each time unless you wrap it in `useShallow`.

## Example 5: immutable updates (nested state)

```jsx
const usePlantStore = create((set) => ({
    plants: [],

    addPlant: (plant) =>
        set((s) => ({ plants: [...s.plants, plant] })),

    waterPlant: (id) =>
        set((s) => ({
            plants: s.plants.map((p) =>
                p.id === id ? { ...p, lastWatered: Date.now() } : p
            ),
        })),

    removePlant: (id) =>
        set((s) => ({ plants: s.plants.filter((p) => p.id !== id) })),
}));
```

`set` only merges the top level. For nested data you must create new arrays and objects yourself. Never mutate: `s.plants.push(plant)` would keep the same reference and nothing would update.

## Example 6: reading a single item efficiently

```jsx
const PlantRow = React.memo(function PlantRow({ id }) {
    // Only re-renders when THIS plant changes
    const plant = usePlantStore((s) => s.plants.find((p) => p.id === id));
    const waterPlant = usePlantStore((s) => s.waterPlant);

    return (
        <View>
            <Text>{plant.name}</Text>
            <Button title="Water" onPress={() => waterPlant(id)} />
        </View>
    );
});

function PlantList() {
    const ids = usePlantStore(useShallow((s) => s.plants.map((p) => p.id)));
    return (
        <FlatList
            data={ids}
            keyExtractor={(id) => id}
            renderItem={({ item }) => <PlantRow id={item} />}
        />
    );
}
```

Passing the id and letting each row select its own data keeps list updates small. `PlantRow` re-renders only when its own plant object changes, because the others keep their references.

## Zustand vs Redux Toolkit

| Concept | Zustand | Redux Toolkit |
|---|---|---|
| Store | `create(...)` | `configureStore(...)` |
| State + update logic | Actions inside the store | `createSlice` with reducers |
| Update call | `increment()` | `dispatch(increment())` |
| Selector | `useStore(s => s.count)` | `useSelector(s => s.counter.count)` |
| Provider | Not needed | `<Provider store={store}>` required |

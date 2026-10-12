# Keys

When you render a list, React needs a way to tell which item is which between renders. That is what the `key` prop is for.

## 1. The Basics

```jsx
const plants = [
    { id: 'p1', name: 'Monstera' },
    { id: 'p2', name: 'Cactus' },
    { id: 'p3', name: 'Fern' },
];

function PlantList() {
    return (
        <View>
            {plants.map(plant => (
                <Text key={plant.id}>{plant.name}</Text>
            ))}
        </View>
    );
}
```

Without a key, React logs a warning: *"Each child in a list should have a unique key prop."*

## 2. Why Keys Matter

On every render, React compares the new list with the previous one. Keys let it match items by identity instead of by position.

If a list changes from `[A, B, C]` to `[C, A, B]`:

- **With stable keys:** React sees that C moved to the front and reorders the existing components. Their state is preserved.
- **Without proper keys (or with index keys):** React matches by position, so it thinks item 0 changed from A to C, item 1 from B to A, and so on. It updates the wrong components, which can cause wasted renders and, worse, state attached to the wrong item.

## 3. What Makes a Good Key

- **Unique among siblings:** it only needs to be unique within that one list, not globally.
- **Stable:** the same item always gets the same key across renders.
- **Predictable:** it is not generated during render (for example, never use `Math.random()`).

## 4. Keys in FlatList

With `FlatList` you don't put a `key` prop on your items. It uses `keyExtractor` instead:

```jsx
<FlatList
    data={plants}
    keyExtractor={item => item.id}
    renderItem={({ item }) => <PlantRow plant={item} />}
/>
```

The key must be a string (convert numbers: `String(item.id)`).

If you leave out `keyExtractor`, FlatList looks for `item.key`, then `item.id`, and falls back to the index.

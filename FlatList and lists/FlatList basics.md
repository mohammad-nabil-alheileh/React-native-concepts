# FlatList Basics

`FlatList` is React Native's component for rendering a long list of items efficiently. Instead of rendering every item at once, it **virtualizes**: it renders only the items near the screen and recycles or unmounts the ones that scroll far away.

If we use `ScrollView` + `.map()`, all items render immediately, which is slow and memory-hungry for big lists.

## 1. The Core Props

| Prop                     | Purpose                                                                       |
| ------------------------ | ----------------------------------------------------------------------------- |
| `data`                   | The array of items to render                                                  |
| `renderItem`             | A function that returns the UI for one item                                   |
| `keyExtractor`           | A function that returns a unique string key for each item                     |
| `ItemSeparatorComponent` | A component rendered between items (not before the first or after the last)  |

```jsx
const PLANTS = [
    { id: '1', name: 'Monstera', room: 'Living room' },
    { id: '2', name: 'Cactus', room: 'Kitchen' },
    { id: '3', name: 'Basil', room: 'Balcony' },
];

const Separator = () => <View style={styles.separator} />;

export default function PlantList() {
    return (
        <FlatList
            data={PLANTS}
            keyExtractor={(item) => item.id}
            renderItem={({ item }) => <PlantRow plant={item} />}
            ItemSeparatorComponent={Separator}
            ListEmptyComponent={<Text>No plants yet</Text>}
            ListHeaderComponent={<Text style={styles.title}>My Plants</Text>}
        />
    );
}
```

## 2. Useful Extras

| Prop                  | Purpose                                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `ListEmptyComponent`  | Shown when `data` is empty                                                                                                      |
| `ListHeaderComponent` | Shown at the top of the list and scrolls with it                                                                                |
| `extraData`           | Pass this when the list depends on state outside `data`, because FlatList is a PureComponent and won't re-render otherwise |

# Pull-to-Refresh

Pull-to-refresh is the gesture where the user drags a list down from the top to reload its data. React Native supports it natively on both platforms. You don't build the gesture yourself; you just control when the spinner shows and what happens when the user pulls.

## 1. The Two Props

| Prop         | Purpose                                                                 |
| ------------ | ----------------------------------------------------------------------- |
| `refreshing` | A boolean you control. `true` shows the spinner, `false` hides it       |
| `onRefresh`  | A function called when the user pulls past the threshold                |

Both are required together. If you pass `onRefresh` without `refreshing`, React Native throws an error, because the spinner state belongs to you.

```jsx
export default function PlantList() {
    const [plants, setPlants] = useState([]);
    const [refreshing, setRefreshing] = useState(false);

    const onRefresh = useCallback(async () => {
        setRefreshing(true);
        try {
            const fresh = await fetchPlants();
            setPlants(fresh);
        } finally {
            setRefreshing(false); // always stop the spinner, even on error
        }
    }, []);

    return (
        <FlatList
            data={plants}
            keyExtractor={(item) => item.id}
            renderItem={({ item }) => (
                <View style={{ padding: 16 }}>
                    <Text>{item.name}</Text>
                    <Text style={{ color: 'gray' }}>Updated: {item.updated}</Text>
                </View>
            )}
            refreshing={refreshing}
            onRefresh={onRefresh}
        />
    );
}
```

## 2. Customizing with RefreshControl

For styling, or for a plain `ScrollView`, use the `RefreshControl` component. `FlatList` accepts it through the `refreshControl` prop too.

```jsx
<ScrollView
    refreshControl={
        <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            tintColor="#2e7d32"                 // iOS spinner color
            colors={['#2e7d32', '#81c784']}     // Android spinner colors
            progressBackgroundColor="#ffffff"   // Android circle background
            title="Updating plants..."          // iOS text under spinner
            progressViewOffset={60}             // push spinner down (e.g. below a header)
        />
    }
>
    {/* content */}
</ScrollView>
```

**RefreshControl customization props:**

| Prop                      | Platform | What it does                                          |
| ------------------------- | -------- | ----------------------------------------------------- |
| `tintColor`               | iOS      | Spinner color                                         |
| `title`                   | iOS      | Text shown under the spinner                          |
| `colors`                  | Android  | Spinner colors (it cycles through them)               |
| `progressBackgroundColor` | Android  | Background color of the spinner circle                |
| `progressViewOffset`      | Both     | Pushes the spinner down (e.g. below a header)         |

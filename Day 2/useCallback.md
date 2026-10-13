# useCallback

useCallback caches a function between renders. It returns the same function reference until a value in the dependency array changes.

```jsx
const handlePress = useCallback(() => {
    doSomething(a, b);
}, [a, b]);
```

## Why its needed

Every render creates a new function object, even if the code inside is identical:

```jsx
const a = () => {};
const b = () => {};
a === b; // false
```

## Example 1: FlatList

```jsx
const PlantItem = React.memo(function PlantItem({ plant, onWater }) {
    return (
        <View>
            <Text>{plant.name}</Text>
            <Button title="Water" onPress={() => onWater(plant.id)} />
        </View>
    );
});

function PlantList({ plants }) {
    const [watered, setWatered] = useState({});

    const handleWater = useCallback((id) => {
        setWatered(prev => ({ ...prev, [id]: Date.now() }));
    }, []);

    const renderItem = useCallback(
        ({ item }) => <PlantItem plant={item} onWater={handleWater} />,
        [handleWater]
    );

    const keyExtractor = useCallback((item) => item.id, []);

    return <FlatList data={plants} renderItem={renderItem} keyExtractor={keyExtractor} />;
}
```

Stable `renderItem` and handlers mean unchanged rows don’t re-render when one row updates.

## Example 2: as an effect dependency

```jsx
const fetchPlants = useCallback(async () => {
    const res = await fetch(`https://api.example.com/plants?q=${query}`);
    setPlants(await res.json());
}, [query]);

useEffect(() => {
    fetchPlants();
}, [fetchPlants]); // re-runs only when query changes
```

Without `useCallback`, `fetchPlants` is new every render and the effect would run on every render, possibly causing an infinite loop.

## useMemo vs useCallback

| | `useMemo` | `useCallback` |
|---|---|---|
| Caches | A value (result of a function) | The function itself |
| Calls your function? | Yes, to compute the value | No, just stores it |
| Typical use | Expensive calculation, stable object | Stable handler for `React.memo` or effects |

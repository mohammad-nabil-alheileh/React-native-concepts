# useMemo

useMemo caches the result of a calculation between renders. React re-runs the function only when one of it's values in the dependency array change. Otherwise it returns the stored result.

```jsx
const value = useMemo(() => computeSomething(a, b), [a, b]);
```

- The first argument is a function that returns the value.
- The second is the dependency array, compared with `Object.is` (by reference for objects and arrays).
- It’s a performance hint, not a guarantee. React may throw the cache away, so your code must still work correctly without it.

## Example 1: expensive filtering and sorting

```jsx
export default function PlantList({ plants }) {
    const [query, setQuery] = useState('');
    const [darkMode, setDarkMode] = useState(false);

    const visiblePlants = useMemo(() => {
        console.log('filtering...');
        return plants
            .filter(p => p.name.toLowerCase().includes(query.toLowerCase()))
            .sort((a, b) => a.name.localeCompare(b.name));
    }, [plants, query]);

    return (
        <View>
            <TextInput value={query} onChangeText={setQuery} placeholder="Search" />
            <FlatList
                data={visiblePlants}
                keyExtractor={p => p.id}
                renderItem={({ item }) => <Text>{item.name}</Text>}
            />
        </View>
    );
}
```

If `darkMode` changes and the component re-renders, the filter and sort do not run again, because `plants` and `query` are unchanged.

## Example 2: stabilizing a Context value

```jsx
function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');

    const value = useMemo(() => ({ theme, setTheme }), [theme]);

    return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}
```

Without `useMemo`, `{ theme, setTheme }` is a new object whenever `ThemeProvider` re-renders, so every consumer re-renders too.

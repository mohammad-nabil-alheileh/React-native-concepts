# Loading, error, and empty states in the UI

Every screen that fetches data is in one of several states at any moment, ou need a UI for each, so the user always knows what is happening:

| State | Meaning | What the user should see |
| --- | --- | --- |
| Loading | Request in flight, no data yet | Spinner or skeleton |
| Error | Request failed | Message + “Try again” button |
| Empty | Request succeeded but the list is `[]` | Friendly message + a call to action |
| Success | Data arrived | The actual content |

Empty is not the same as error. An empty list means everything worked and there is simply nothing there yet.

## Basic version: three separate booleans

```jsx
export default function PlantList() {
  const [plants, setPlants] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const load = async () => {
    try {
      setLoading(true);
      setError(null);
      setPlants(await getPlants());
    } catch (e) {
      setError(parseError(e));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => { load(); }, []);

  if (loading) return <ActivityIndicator size="large" />;

  if (error) {
    return (
      <View>
        <Text>{error.message}</Text>
        <Button title="Try again" onPress={load} />
      </View>
    );
  }

  if (plants.length === 0) {
    return (
      <View>
        <Text>No plants yet</Text>
        <Button title="Add your first plant" onPress={() => {}} />
      </View>
    );
  }

  return (
    <FlatList
      data={plants}
      keyExtractor={(p) => String(p.id)}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
}
```

The order of the checks matters: loading, then error, then empty, then success.

## Three details that make an app feel polished

- **First load vs refresh.** On the first load there is no data, so show a full-screen spinner. On a pull-to-refresh or a background refetch, keep the old data visible and show only a small indicator.
- **Errors with old data.** If a refresh fails but you already have data, don’t replace the whole screen with an error. Keep the data and show a small banner or toast.
- **Disable buttons while submitting,** so a double tap does not send two POSTs.

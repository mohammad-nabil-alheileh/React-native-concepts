# Server state vs UI state

There are two very different kinds of state in an app:

- **UI state** is owned by your app. It exists only on the device and only matters to the interface: a modal being open, the selected tab, text in an input, the current theme. You can change it instantly and it is always correct.
- **Server state** is owned by the server. Your app only holds a copy (a snapshot) of data that lives somewhere else: plants, orders, user profile, messages. It is asynchronous (you must wait for it), shared (other users and devices can change it), and can become stale at any moment.

## UI state vs server state

| | UI state | Server state |
|---|---|---|
| Owner | Your app | The server |
| Source of truth | The app | The server (you hold a copy) |
| Speed | Instant | Async (network delay) |
| Can go stale? | No | Yes |
| Can fail? | No | Yes (offline, 500, timeout) |
| Shared by other users? | No | Yes |
| Examples | Modal open, input text, selected tab | Plant list, user profile, orders |
| Tools | `useState`, Context, Zustand | TanStack Query, RTK Query, SWR |

## Example 1: handling server state by hand

```jsx
function PlantScreen() {
    const [plants, setPlants] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        let cancelled = false;
        fetch('https://api.example.com/plants')
            .then(r => r.json())
            .then(data => { if (!cancelled) setPlants(data); })
            .catch(e => { if (!cancelled) setError(e); })
            .finally(() => { if (!cancelled) setLoading(false); });
        return () => { cancelled = true; };
    }, []);

    // ...
}
```

This is `useFetch` without the hook. And it is missing most of what real server state needs:

- No caching: leaving and returning to the screen refetches everything and shows a spinner again.
- No deduplication: two components needing the same data fire two requests.
- No refetching when the app comes back to the foreground or the network reconnects.
- No retries after a failed request.
- No pagination or infinite scroll handling.
- No way to update the list after a change (add, edit, delete) except manual bookkeeping.

## Example 2: the same screen with TanStack Query

```jsx
const queryClient = new QueryClient();

export default function App() {
    return (
        <QueryClientProvider client={queryClient}>
            <PlantScreen />
        </QueryClientProvider>
    );
}

async function fetchPlants() {
    const res = await fetch('https://api.example.com/plants');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
}

function PlantScreen() {
    const { data, isPending, isError, error, refetch } = useQuery({
        queryKey: ['plants'],
        queryFn: fetchPlants,
    });

    if (isPending) return <ActivityIndicator />;
    if (isError) return <Button title={`Retry (${error.message})`} onPress={refetch} />;

    return (
        <FlatList
            data={data}
            keyExtractor={p => p.id}
            renderItem={({ item }) => <Text>{item.name}</Text>}
        />
    );
}
```

You get caching, deduplication, retries and background refetching with no extra code. Leave the screen and come back: the cached list appears instantly while a fresh copy loads in the background.

## Example 3: query keys

```jsx
// Different key = different cache entry
useQuery({ queryKey: ['plants'], queryFn: fetchPlants });
useQuery({ queryKey: ['plants', plantId], queryFn: () => fetchPlant(plantId) });
useQuery({ queryKey: ['plants', { search }], queryFn: () => fetchPlants(search) });
```

The key identifies the data. When something in the key changes (like `search`), Query fetches the new data automatically.

## Example 4: changing server data

```jsx
function AddPlantButton() {
    const queryClient = useQueryClient();

    const addPlant = useMutation({
        mutationFn: (newPlant) =>
            fetch('https://api.example.com/plants', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(newPlant),
            }).then(r => r.json()),

        onSuccess: () => {
            // Mark the cached list as stale, so it refetches
            queryClient.invalidateQueries({ queryKey: ['plants'] });
        },
    });

    return (
        <Button
            title={addPlant.isPending ? 'Saving...' : 'Add plant'}
            onPress={() => addPlant.mutate({ name: 'Monstera' })}
        />
    );
}
```

`useQuery` is for reading, `useMutation` is for writing. After a write, you invalidate the related queries so the UI shows the server’s real data. Never push the new item into a local array and hope it matches the server.

## Example 5: React Native specifics

On web, Query refetches when the browser tab regains focus. React Native has no tabs, so you connect it to the app state yourself:

```jsx
// Refetch when the app returns to the foreground
AppState.addEventListener('change', (status) => {
    focusManager.setFocused(status === 'active');
});

// Know when the device is online or offline
onlineManager.setEventListener((setOnline) =>
    NetInfo.addEventListener((state) => setOnline(!!state.isConnected))
);
```

Set this up once, and your screens refetch stale data when the user comes back to the app and pause requests while offline.

# Pagination

Pagination means loading data in small chunks (pages) instead of fetching everything at once.

## 1. Three Common Pagination Styles

| Style        | Request looks like       | Best for                       |
| ------------ | ------------------------ | ------------------------------ |
| Page-based   | `?page=2&limit=20`       | Simple lists, page numbers     |
| Offset-based | `?offset=40&limit=20`    | Same idea, row-based           |
| Cursor-based | `?after=abc123&limit=20` | Feeds where data changes often |

## 2. How You Know There Is More Data

The response usually includes one of these, so the app knows whether to ask for another page:

| Response field | Example                          | How to read it                                                                           |
| -------------- | -------------------------------- | ---------------------------------------------------------------------------------------- |
| `hasMore`      | `{ items, hasMore: true }`       | A boolean from the server: `true` = another page exists, `false` = this was the last one |
| `total`        | `{ items, total: 45 }`           | The total number of items; there is more while `loadedCount < total`                     |
| `nextCursor`   | `{ items, nextCursor: 'abc123' }`| The value to send in the next request (`?after=abc123`); `null` when finished            |

## 3. Example: "Load More" Button

```jsx
const PAGE_SIZE = 10;
const TOTAL = 45;
const wait = (ms) => new Promise((r) => setTimeout(r, ms));

// fake API: returns one page of plants and a hasMore flag
async function fetchPlants(page, limit = PAGE_SIZE) {
    await wait(800);
    const start = (page - 1) * limit;
    const items = Array.from(
        { length: Math.max(0, Math.min(limit, TOTAL - start)) },
        (_, i) => ({ id: String(start + i + 1), name: `Plant #${start + i + 1}` })
    );
    return { items, hasMore: start + limit < TOTAL };
}

export default function PaginatedList() {
    const [plants, setPlants] = useState([]);
    const [page, setPage] = useState(1);
    const [hasMore, setHasMore] = useState(true);
    const [loading, setLoading] = useState(false);

    const loadPage = useCallback(async (pageToLoad) => {
        setLoading(true);
        try {
            const res = await fetchPlants(pageToLoad);
            // page 1 replaces the list, later pages are appended to the old items
            setPlants((prev) => (pageToLoad === 1 ? res.items : [...prev, ...res.items]));
            setHasMore(res.hasMore);
            setPage(pageToLoad);
        } finally {
            setLoading(false);
        }
    }, []);

    useEffect(() => {
        loadPage(1);
    }, [loadPage]);

    const Footer = () => {
        if (loading) return <ActivityIndicator style={{ margin: 16 }} />;
        if (!hasMore) return <Text style={{ textAlign: 'center', padding: 16 }}>No more plants</Text>;
        return <Button title="Load more" onPress={() => loadPage(page + 1)} />;
    };

    return (
        <FlatList
            data={plants}
            keyExtractor={(item) => item.id}
            renderItem={({ item }) => (
                <View style={{ padding: 16 }}>
                    <Text>{item.name}</Text>
                </View>
            )}
            ListFooterComponent={Footer}
        />
    );
}
```

## 4. The State You Need to Track

| State     | Type    | Meaning                                                                                     |
| --------- | ------- | ------------------------------------------------------------------------------------------- |
| `plants`  | array   | All items loaded so far (every page joined together)                                        |
| `page`    | number  | The last page that was loaded, so the next request asks for `page + 1`                      |
| `hasMore` | boolean | Whether another request makes sense. Starts `true`, set from the server's response         |
| `loading` | boolean | Whether a request is running right now. `true` while fetching, `false` when it finishes     |

## 5. The Flags Explained

The two boolean flags, `hasMore` and `loading`, decide what the footer shows:

| `loading` | `hasMore` | Footer shows            | Why                                                                 |
| --------- | --------- | ----------------------- | ------------------------------------------------------------------- |
| `true`    | any       | Spinner                 | A page is being fetched, so wait for it                             |
| `false`   | `true`    | "Load more" button      | Nothing is loading and the server says more pages exist             |
| `false`   | `false`   | "No more plants" text   | The last page was reached, so there is nothing left to ask for      |

- **`hasMore`** stops useless requests. Once the server says `hasMore: false`, the app never asks for another page, so it doesn't send requests that would come back empty.
- **`loading`** stops duplicate requests. While it is `true`, the button is replaced by a spinner, so the user can't tap "Load more" twice and load the same page two times. It is reset in `finally`, so the spinner stops even if the request fails.

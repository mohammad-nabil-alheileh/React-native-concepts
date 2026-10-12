# Infinite Scrolling

Infinite scrolling loads the next page automatically when the user nears the bottom of the list, with no buttons. It is **pagination plus a scroll trigger**.

## 1. The Two Key Props

| Prop                    | Purpose                                                                                                                                                  |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `onEndReached`          | Called when the user scrolls within the threshold distance of the end of the list                                                                        |
| `onEndReachedThreshold` | How close to the end, measured in multiples of the visible list height (not pixels). `0.5` means "when the bottom is half a screen away". Default is `2` |

## 2. Example

```jsx
const PAGE_SIZE = 15;
const TOTAL = 60;
const wait = (ms) => new Promise((r) => setTimeout(r, ms));

// Mock paginated API
async function fetchPlants(page) {
  await wait(900);
  const start = (page - 1) * PAGE_SIZE;
  const count = Math.max(0, Math.min(PAGE_SIZE, TOTAL - start));
  const items = Array.from({ length: count }, (_, i) => ({
    id: String(start + i + 1),
    name: `Plant #${start + i + 1}`,
  }));
  return { items, hasMore: start + PAGE_SIZE < TOTAL };
}

export default function InfiniteList() {
  const [plants, setPlants] = useState([]);
  const [hasMore, setHasMore] = useState(true);
  const [loadingMore, setLoadingMore] = useState(false);

  const pageRef = useRef(1);        // next page to load
  const loadingRef = useRef(false); // synchronous guard

  const loadMore = useCallback(async () => {
    if (loadingRef.current || !hasMore) return;
    loadingRef.current = true;
    setLoadingMore(true);
    try {
      const res = await fetchPlants(pageRef.current);
      setPlants((prev) => [...prev, ...res.items]);
      setHasMore(res.hasMore);
      pageRef.current += 1;
    } finally {
      loadingRef.current = false;
      setLoadingMore(false);
    }
  }, [hasMore]);

  // first page
  useEffect(() => {
    loadMore();
  }, []); // eslint-disable-line react-hooks/exhaustive-deps

  return (
    <FlatList
      data={plants}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <View style={{ padding: 20 }}>
          <Text>{item.name}</Text>
        </View>
      )}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={
        loadingMore ? (
          <ActivityIndicator style={{ margin: 16 }} />
        ) : !hasMore ? (
          <Text style={{ textAlign: 'center', padding: 16 }}>You reached the end</Text>
        ) : null
      }
    />
  );
}
```

## 3. How the Infinite Scroll Code Works

### The flow

1. The screen opens, and the `useEffect` calls `loadMore()` once to load **page 1**.
2. The user scrolls down. When the bottom of the list is half a screen away (`onEndReachedThreshold={0.5}`), FlatList calls `onEndReached`, which is `loadMore`.
3. `loadMore` fetches the next page and **appends** it to `plants`, so the list grows.
4. Steps 2 and 3 repeat until the server returns `hasMore: false`. After that, `loadMore` does nothing and the footer says "You reached the end".

### The state and refs

| Name          | Kind  | What it does                                                                                       |
| ------------- | ----- | -------------------------------------------------------------------------------------------------- |
| `plants`      | state | All items loaded so far. Each new page is added to the end                                         |
| `hasMore`     | state | `false` once the server says there are no more pages, so no more requests are sent                 |
| `loadingMore` | state | `true` while a page is loading. Used only for the UI (to show the spinner in the footer)           |
| `pageRef`     | ref   | The number of the **next** page to load. Goes up by 1 after each successful fetch                  |
| `loadingRef`  | ref   | A guard that blocks a second `loadMore` call while one is already running                          |

### Why refs instead of state?

`onEndReached` can fire **several times in a row** while the user scrolls, often before React has re-rendered. A state update (`setLoadingMore(true)`) isn't visible until the next render, so a second call would still see `false` and load the same page again.

A ref changes **immediately** (synchronously). When the first call sets `loadingRef.current = true`, every call after it sees `true` and stops at the first line:

```jsx
if (loadingRef.current || !hasMore) return;
```

That's why the code tracks loading twice: `loadingRef` blocks duplicate calls, and `loadingMore` state triggers a re-render to show the spinner.

The page number is a ref for a similar reason. It's only needed inside `loadMore` and never shown on screen, so changing it shouldn't cause a re-render.

### The footer

| Condition                 | Footer shows                  |
| ------------------------- | ----------------------------- |
| `loadingMore` is `true`   | A spinner                     |
| `hasMore` is `false`      | "You reached the end"         |
| Otherwise                 | Nothing (`null`)              |
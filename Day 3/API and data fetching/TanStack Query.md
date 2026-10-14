# TanStack Query

TanStack Query is a library for server state: data that lives on the server and that your app only borrows. It handles fetching, caching, refetching, and updating that data, so you stop writing the same useState + useEffect + try/catch pattern on every screen.

## What it replaces

| Manual way | With TanStack Query |
| --- | --- |
| loading, error, data with useState | `isPending`, `isError`, `data` |
| try/catch + setError | `error` is provided |
| AbortController in useEffect | You pass the `signal`, and it cancels for you |
| Retry interceptor with backoff | `retry` option, built in |
| Reload when back online | `refetchOnReconnect`, once you connect NetInfo |
| Remembering data between screens | Cache |

## Set up

```js
// App.jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000, // data counts as fresh for 30 seconds
    },
  },
});

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <RootNavigator />
    </QueryClientProvider>
  );
}
```

## Two React Native specific steps

React Query was built for browsers. In React Native you connect it to the phone yourself:

```js
// src/api/queryManagers.js
import { onlineManager, focusManager } from "@tanstack/react-query";
import NetInfo from "@react-native-community/netinfo";
import { AppState } from "react-native";

// Topic 11: tell React Query when the phone goes offline or online
onlineManager.setEventListener((setOnline) =>
  NetInfo.addEventListener((state) => {
    setOnline(state.isConnected !== false && state.isInternetReachable !== false);
  })
);

// "Window focus" in a browser = "app returns to the foreground" on a phone
export function startFocusManager() {
  const sub = AppState.addEventListener("change", (status) => {
    focusManager.setFocused(status === "active");
  });
  return () => sub.remove();
}
```

## Part 1: Queries (reading data)

```jsx
import { useQuery } from "@tanstack/react-query";
import { View, Text, FlatList } from "react-native";
import { getPlants } from "../api/plants";
import { parseError } from "../api/getErrorMessage";
import { Loading, ErrorView, EmptyView } from "../components/StateViews";

export default function PlantList() {
  const { data, isPending, isError, error, refetch, isRefetching } = useQuery({
    queryKey: ["plants"],
    queryFn: ({ signal }) => getPlants({ signal }),
  });

  if (isPending) return <Loading />;

  if (isError) {
    return <ErrorView message={parseError(error).message} onRetry={refetch} />;
  }

  return (
    <FlatList
      data={data}
      keyExtractor={(p) => String(p.id)}
      renderItem={({ item }) => <Text>{item.name}</Text>}
      ListEmptyComponent={<EmptyView message="No plants yet" />}
      refreshing={isRefetching}
      onRefresh={refetch}
    />
  );
}
```

### The two required options

- **queryKey:** the cache address for this data. It is an array.
- **queryFn:** a function that returns a Promise with the data. It must throw on failure. Axios already does that for non-2xx responses.

### Query keys

The key identifies the data. Anything the request depends on goes inside it, and when the key changes, React Query fetches again:

```js
["plants"]                          // the whole list
["plants", id]                      // one plant
["plants", { search: "aloe" }]      // a filtered list
```

### Search as you type

```js
const { data } = useQuery({
  queryKey: ["plants", "search", search],
  queryFn: ({ signal }) => searchPlants(search, { signal }),
  enabled: search.length > 1, // do not run until there is something to search
});
```

Every time `search` changes, the key changes, a new request starts, and the old one is cancelled because the signal was passed to Axios.

### Status fields

| Field | Meaning |
| --- | --- |
| `isPending` | There is no data yet. Show the full-screen spinner |
| `isFetching` | A request is running right now, even if cached data is on screen |
| `isRefetching` | Fetching again while data already exists |
| `isError` | The query failed |
| `isLoading` | `isPending` and `isFetching` together |

## Part 2: Caching

React Query keeps results in memory and follows the stale-while-revalidate idea: show what you have immediately, then update it in the background.

Two timers control it:

| Setting | Default | Meaning |
| --- | --- | --- |
| `staleTime` | `0` | How long data counts as fresh. Fresh data is never refetched. Stale data is shown instantly, then refetched in the background |
| `gcTime` | 5 minutes | How long unused data stays in memory before it is deleted |

The default `staleTime` of `0` means data is stale immediately, so it refetches often. For most mobile apps, set a sensible `staleTime` (30 seconds to a few minutes) and use `Infinity` for data that almost never changes (like a list of countries).

By default, stale queries refetch when:

- a component using them mounts (`refetchOnMount`)
- the app returns to the foreground (`refetchOnWindowFocus`, needs the `focusManager`)
- the phone reconnects (`refetchOnReconnect`, needs the `onlineManager`)

One React Native detail: React Navigation keeps earlier screens mounted in a stack. So going back to a screen does not trigger a mount refetch. The usual fix is to invalidate after changes, and to call `refetch` in `useFocusEffect` only for data that changes elsewhere.

### Retry and offline behavior

- Queries retry 3 times with exponential backoff by default
- Mutations do not retry by default, which is the safe choice for POST
- When the phone is offline, queries pause instead of failing, then resume on reconnect

Do not retry client errors. Customize it once in the `QueryClient`:

```js
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,
      retry: (failureCount, error) => {
        const status = error?.response?.status;
        if (status && status >= 400 && status < 500) return false; // 4xx will not fix itself
        return failureCount < 2;
      },
    },
  },
});
```

## Part 3: Mutations

Queries read data. Mutations create, update, or delete it. They run only when you call them, not automatically.

```jsx
export default function AddPlantForm() {
  const queryClient = useQueryClient();

  const addPlant = useMutation({
    mutationFn: createPlant,
    onSuccess: () => {
      // The list is now outdated, so mark it stale and refetch it
      queryClient.invalidateQueries({ queryKey: ["plants"] });
    },
  });

  return (
    <View>
      <Button
        title={addPlant.isPending ? "Saving..." : "Save"}
        disabled={addPlant.isPending}
        onPress={() => addPlant.mutate({ name: "Aloe", waterEveryDays: 7 })}
      />
      {addPlant.isError && <Text>{parseError(addPlant.error).message}</Text>}
    </View>
  );
}
```

`mutate(variables)` fires and forgets. Use `mutateAsync` if you want a Promise to await.

`invalidateQueries` is the key idea: it tells the cache “this data is out of date”. Any screen showing `["plants"]` refetches. It also matches by prefix, so `["plants"]` invalidates `["plants", 5]` and the others too.

## Optimistic updates (instant UI)

Update the screen before the server answers, and undo it if the request fails. This makes taps feel instant.

## Part 4: useInfiniteQuery

For long lists loaded page by page as the user scrolls.

```js
// src/api/plants.js
export const getPlantsPage = async ({ page, signal }) => {
  const { data } = await api.get("/plants", { params: { page, limit: 20 }, signal });
  return data; // for example { items: [...], hasMore: true }
};
```

```jsx
export default function PlantFeed() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isPending,
    isError,
    error,
    refetch,
    isRefetching,
  } = useInfiniteQuery({
    queryKey: ["plants", "feed"],
    queryFn: ({ pageParam, signal }) => getPlantsPage({ page: pageParam, signal }),
    initialPageParam: 1,
    getNextPageParam: (lastPage, allPages) =>
      lastPage.hasMore ? allPages.length + 1 : undefined, // undefined means "no more pages"
  });

  if (isPending) return <Loading />;
  if (isError) return <ErrorView message={parseError(error).message} onRetry={refetch} />;

  const plants = data.pages.flatMap((page) => page.items); // merge all pages into one list

  return (
    <FlatList
      data={plants}
      keyExtractor={(p) => String(p.id)}
      renderItem={({ item }) => <Text>{item.name}</Text>}
      onEndReached={() => {
        if (hasNextPage && !isFetchingNextPage) fetchNextPage();
      }}
      onEndReachedThreshold={0.5}
      ListFooterComponent={isFetchingNextPage ? <ActivityIndicator /> : null}
      ListEmptyComponent={<EmptyView message="No plants yet" />}
      refreshing={isRefetching && !isFetchingNextPage}
      onRefresh={refetch}
    />
  );
}
```

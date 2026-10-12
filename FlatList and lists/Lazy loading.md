# Lazy Loading

Lazy loading means only doing the work (rendering, loading, fetching) when it's actually needed, instead of upfront.

In React Native lists, it shows up in two ways:

- **Lazy rendering:** FlatList mounts only the rows near the viewport. This is built in, and you tune it with props.
- **Lazy loading of content:** heavy things inside rows (images, details) or whole screens load only when they come into view or are opened.

## 1. Lazy Rendering: The Windowing Props

FlatList keeps a "window" of rendered items around the visible area. It can be controlled with these props:

| Prop                        | What it does                                                                                      | Default |
| --------------------------- | ------------------------------------------------------------------------------------------------- | ------- |
| `initialNumToRender`        | How many items render on the first pass                                                           | `10`    |
| `maxToRenderPerBatch`       | How many items render per scroll batch                                                            | `10`    |
| `windowSize`                | How many "screens" of content stay mounted (1 = visible only, 21 = 10 above + 10 below + visible) | `21`    |
| `updateCellsBatchingPeriod` | Delay (ms) between batches                                                                        | `50`    |

## 2. Lazy Loading Images in Rows

Don't load every image at once. Rows only mount when near the screen, so images load naturally as you scroll.

## 3. Lazy Loading Screens and Components

Screens can also be loaded on demand. For example, bottom tabs can skip rendering a tab until the user first opens it (`lazy: true` is already the default for bottom tabs):

```jsx
<Tab.Navigator screenOptions={{ lazy: true }}>
    <Tab.Screen name="Home" component={HomeScreen} />
    <Tab.Screen name="Settings" component={SettingsScreen} />
</Tab.Navigator>
```

And for components, use `React.lazy` + `Suspense`:

```jsx
const HeavyChart = React.lazy(() => import('./HeavyChart'));

function Stats() {
    return (
        <React.Suspense fallback={<Text>Loading chart...</Text>}>
            <HeavyChart />
        </React.Suspense>
    );
}
```

> **Caveat:** Metro bundles everything into one file by default, so `React.lazy` delays execution, not the download. It doesn't shrink your bundle the way it does on the web.

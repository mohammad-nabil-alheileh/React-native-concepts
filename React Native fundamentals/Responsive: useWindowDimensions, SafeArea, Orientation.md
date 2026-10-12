# Responsive: useWindowDimensions, SafeArea, Orientation

## 1. useWindowDimensions

This hook gives you the current size of the app window, and it updates automatically when the size changes (rotation, split-screen, foldables).

```jsx
function Screen() {
    const { width, height, scale, fontScale } = useWindowDimensions();

    return (
        <View>
            <Text>{width} x {height}</Text>
            <Text>Pixel density: {scale}, font scale: {fontScale}</Text>
        </View>
    );
}
```

React Native has no media queries, so you build your own from `width`:

```jsx
const columns = width >= 900 ? 4 : width >= 600 ? 3 : 2;
const gap = 12;
const itemWidth = (width - gap * (columns + 1)) / columns;

return (
    <View style={{ flexDirection: 'row', flexWrap: 'wrap', padding: gap, gap }}>
        {products.map(p => (
            <View key={p.id} style={{ width: itemWidth, height: itemWidth }}>
                <Text>{p.name}</Text>
            </View>
        ))}
    </View>
);
```

### Dimensions.get() vs the Hook

`Dimensions.get()` reads the size once and doesn't update. If you call it at the top of a file, the value is frozen at load time and will be wrong after rotation. Prefer the hook inside components.

- `'window'` is the visible app area. Use this one.
- `'screen'` is the full physical screen, which on Android includes areas like the system bars.

## 2. SafeArea

Modern devices have areas your content must avoid: the notch or Dynamic Island, the status bar, the rounded corners, and the home indicator. The "safe area" is the part of the screen free of those. Without handling it, content can be hidden under the notch or the home bar.

Use **`react-native-safe-area-context`**. The built-in `SafeAreaView` from `react-native` only works on iOS and is deprecated in recent versions, so the community library is the standard. Wrap your app once in `<SafeAreaProvider>` so the components and hook below can read the insets.

### Option A: SafeAreaView Component

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';

function Home() {
    return (
        <SafeAreaView style={{ flex: 1, backgroundColor: '#fff' }}>
            <Text>Safe from the notch and home indicator</Text>
        </SafeAreaView>
    );
}
```

You can limit which edges it pads:

```jsx
<SafeAreaView edges={['top', 'left', 'right']} style={{ flex: 1 }}>
```

### Option B: useSafeAreaInsets Hook (More Control)

```jsx
function Footer() {
    const insets = useSafeAreaInsets(); // { top, bottom, left, right }

    return (
        <View style={{ paddingBottom: insets.bottom + 8, paddingHorizontal: 16 }}>
            <Pressable><Text>Save</Text></Pressable>
        </View>
    );
}
```

## 3. Orientation

There is no dedicated "orientation" value. You compare the dimensions:

```jsx
function Gallery() {
    const { width, height } = useWindowDimensions();
    const isLandscape = width > height;

    return (
        <View style={{ flexDirection: isLandscape ? 'row' : 'column', flex: 1 }}>
            <View style={{ flex: 1 }}><Text>Photo</Text></View>
            <View style={{ flex: 1 }}><Text>Details</Text></View>
        </View>
    );
}
```

## 4. Common Pitfalls

| Pitfall                                  | Why it hurts                             | Fix                                                          |
| ---------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| `Dimensions.get()` at file top level     | Stale after rotation                     | Use `useWindowDimensions()` inside the component             |
| Hard-coded pixel widths (`width: 375`)   | Breaks on other devices                  | Use `flex`, `%`, `aspectRatio`                               |
| Ignoring `fontScale`                     | Large system fonts break layouts         | Avoid fixed heights on text containers, test with big fonts  |
| Forgetting the bottom inset              | Buttons hidden under the home indicator  | `useSafeAreaInsets().bottom`                                 |
| Testing on one device only               | Layout bugs on tablets and small phones  | Test small phone, large phone, tablet, and landscape         |
| Fixed-width content on tablets           | Stretched, hard-to-read UI               | `maxWidth` plus `alignSelf: 'center'`                        |

# Reanimated

React Native Reanimated is an animation library that runs animations on the UI thread instead of the JS thread. Animations stay smooth (60/120 fps) even when your JavaScript is busy fetching data, rendering lists, or running logic.

## Core concepts

| Concept | What it does |
| --- | --- |
| Shared value (`useSharedValue`) | A value that lives on the UI thread. Changing it doesn’t re-render the component. |
| Animated style (`useAnimatedStyle`) | Maps shared values to style properties, re-evaluated on the UI thread. |
| Animation functions | `withTiming`, `withSpring`, `withDelay`, `withRepeat`, `withSequence`, `withDecay` |
| Worklet | A JS function that runs on the UI thread. Functions inside `useAnimatedStyle` and gesture callbacks are automatically worklets. |
| Animated components | `Animated.View`, `Animated.Text`, `Animated.ScrollView`, etc. |

## Examples

### 1. Basic animation

```jsx
import Animated, {
  useSharedValue, useAnimatedStyle, withTiming,
} from 'react-native-reanimated';

function Box() {
  const offset = useSharedValue(0);

  const style = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }],
  }));

  return (
    <>
      <Animated.View style={[styles.box, style]} />
      <Button title="Move" onPress={() => {
        offset.value = withTiming(offset.value + 100, { duration: 400 });
      }} />
    </>
  );
}
```

### 2. Spring and repeat

```jsx
const scale = useSharedValue(1);

useEffect(() => {
  scale.value = withRepeat(withTiming(1.2, { duration: 600 }), -1, true);
  // -1 = infinite, true = reverse each cycle
}, []);

const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));
```

### 3. Layout animations

```jsx
import Animated, { FadeIn, FadeOut, LinearTransition } from 'react-native-reanimated';

{items.map(item => (
  <Animated.View
    key={item.id}
    entering={FadeIn}
    exiting={FadeOut}
    layout={LinearTransition}
  />
))}
```

## When to choose which

| Need | Use |
| --- | --- |
| Simple fade or slide on a state change | Built-in `Animated` or Reanimated layout animations |
| Animate list items being added or removed | Reanimated layout animations |
| Drag, swipe, pinch | Gesture Handler + Reanimated |
| Scroll-linked headers or parallax | Reanimated |
| Anything while the JS thread is busy | Reanimated |

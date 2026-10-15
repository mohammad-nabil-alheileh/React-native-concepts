# Gesture Handler basics

React Native Gesture Handler (RNGH) is a library that handles touch gestures natively (on the iOS/Android UI thread) instead of through React Native’s JavaScript-based responder system.

It lets you detect gestures such as:

- Tap and double tap
- Long press
- Pan (drag)
- Swipe / Fling
- Pinch and Rotation

React Native’s built-in `Pressable` and `ScrollView` cover simple taps and scrolling. RNGH is for everything beyond that.

## Why it exists

The built-in system sends touch events from native to JS, and JS decides what to do. That is slow for continuous gestures like dragging, because every move event crosses the bridge. RNGH recognizes gestures natively, so they stay smooth even if the JS thread is busy. It also handles conflicts between gestures (for example a swipe inside a scroll view) much better.

## Setup

```bash
npm install react-native-gesture-handler
```

Wrap your app root in `GestureHandlerRootView`. Without it, gestures don’t work on Android.

```jsx
import { GestureHandlerRootView } from 'react-native-gesture-handler';

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <MyApp />
    </GestureHandlerRootView>
  );
}
```

## The core pattern

Two parts: create a gesture, then attach it with `GestureDetector`.

```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const tap = Gesture.Tap()
  .onEnd(() => console.log('tapped'));

<GestureDetector gesture={tap}>
  <View style={styles.box} />
</GestureDetector>
```

Each gesture has lifecycle callbacks: `onBegin`, `onStart`, `onUpdate`, `onEnd`, `onFinalize`.

## Examples

### 1. Double tap vs single tap

```jsx
const doubleTap = Gesture.Tap()
  .numberOfTaps(2)
  .onEnd(() => console.log('double tap'));

const singleTap = Gesture.Tap()
  .onEnd(() => console.log('single tap'));

// Exclusive: single tap only fires if double tap fails
const taps = Gesture.Exclusive(doubleTap, singleTap);
```

### 2. Draggable box (Pan)

```jsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

function DraggableBox() {
  const x = useSharedValue(0);
  const y = useSharedValue(0);

  const pan = Gesture.Pan()
    .onUpdate((e) => {
      x.value = e.translationX;
      y.value = e.translationY;
    })
    .onEnd(() => {
      x.value = withSpring(0);   // snap back
      y.value = withSpring(0);
    });

  const style = useAnimatedStyle(() => ({
    transform: [{ translateX: x.value }, { translateY: y.value }],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.box, style]} />
    </GestureDetector>
  );
}
```

## Combining gestures

| Composer | Meaning |
| --- | --- |
| `Gesture.Simultaneous(a, b)` | Both can be active at once (for example pinch + rotate an image) |
| `Gesture.Race(a, b)` | First one to activate wins, the other is cancelled |
| `Gesture.Exclusive(a, b)` | `a` gets priority, `b` only fires if `a` fails |

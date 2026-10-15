# Unnecessary re-renders and large lists

Re-rendering is not always bad, because React is fast at it. It becomes a problem when the work is expensive or happens very often. In React Native, every re-render runs on the JS thread, so wasted renders eat into your 16.6 ms frame budget.

## Large lists

Never render a big list with `ScrollView` and `.map()`. It creates every item at once, which costs memory, startup time, and JS thread time.

- Use virtualized lists, which render only what is on screen (`FlatList`, `FlashList`).
  - `FlashList` is a drop-in alternative that recycles row components instead of creating and destroying them.

# Cold start, background app, and already-open app

A deep link can arrive in three different situations, and the app receives the URL differently in each.

## The three states

| State | Is the app process alive? | Is your JS state kept? | How the URL arrives |
| --- | --- | --- | --- |
| Cold start (killed or never opened) | No, it starts from scratch | No, everything is fresh | `Linking.getInitialURL()` |
| Background (user pressed Home) | Yes, suspended | Yes, the same screens and state | `Linking` `'url'` event |
| Already open (foreground) | Yes, running | Yes | `Linking` `'url'` event |

Background and already-open behave the same way in code. The difference is that in the background case the app comes back to the front first.

## Manual handling

```js
import { useEffect } from 'react';
import { Linking } from 'react-native';

function useDeepLinks(handle) {
  useEffect(() => {
    // Cold start: app was launched by the link
    Linking.getInitialURL().then((url) => {
      if (url) handle(url);
    });

    // Background / already open: link arrives while app is alive
    const sub = Linking.addEventListener('url', ({ url }) => handle(url));
    return () => sub.remove();
  }, []);
}
```

## With React Navigation

When you pass `linking` to `NavigationContainer`, it does both paths for you:

- On cold start, it calls `getInitialURL()` and builds the initial navigation state from it.
- While running, it listens for `'url'` events and navigates to the matching screen.

You only need to customize this when links come from somewhere other than `Linking`, such as a push notification

```js
const linking = {
  prefixes: ['myapp://', 'https://example.com'],
  config,

  // Cold start
  async getInitialURL() {
    const url = await Linking.getInitialURL();
    if (url != null) return url;
    // you could check other sources here
  },

  // Already running
  subscribe(listener) {
    const sub = Linking.addEventListener('url', ({ url }) => listener(url));
    return () => sub.remove();
  },
};
```

# Handling authentication links

Authentication creates two different deep-link problems:

- **Links to protected screens.** The user taps `myapp://order/99` but is logged out. Where do they land, and do they reach the order after logging in?
- **Links that perform authentication.** Email verification, magic links, password resets, and OAuth redirects, like `myapp://auth/verify?token=abc123`.

## Problem 1: Protected screens

Most apps render screens conditionally based on auth state:

When logged out, `Order` doesn’t exist in the navigator, so a link to `order/99` can’t reach it. It’s ignored or falls through to your `NotFound` screen, and the user ends up on `Login` with their destination lost.

### The fix: remember the link, replay it after login

```js
import { Linking } from 'react-native';
import { createContext, useContext, useRef } from 'react';

const PendingLinkContext = createContext(null);
```

Capture the URL in a custom `getInitialURL` and subscribe when the user is logged out:

```js
const pendingUrl = useRef(null);

const linking = {
  prefixes: ['myapp://', 'https://example.com'],
  config,

  async getInitialURL() {
    const url = await Linking.getInitialURL();
    if (url && !isLoggedIn()) {
      pendingUrl.current = url;   // save it
      return null;                // don't try to navigate yet
    }
    return url;
  },

  subscribe(listener) {
    const sub = Linking.addEventListener('url', ({ url }) => {
      if (!isLoggedIn()) {
        pendingUrl.current = url; // save it
        return;
      }
      listener(url);
    });
    return () => sub.remove();
  },
};
```

After a successful login, replay it:

```js
async function onLoginSuccess() {
  setUser(await login());
  if (pendingUrl.current) {
    const url = pendingUrl.current;
    pendingUrl.current = null;
    // wait a tick so the logged-in screens have mounted
    setTimeout(() => Linking.openURL(url), 0);
  }
}
```

## Problem 2: Links that authenticate

Use a short-lived, one-time token. The app sends it to your backend, which exchanges it for a real session.

### Password reset

`myapp://reset-password?token=xyz` opens a `ResetPassword` screen in the logged-out group. The user types a new password, and the token goes to the backend with it.

### OAuth (Google, GitHub, and so on)

The flow is: your app opens the system browser, the user logs in, and the provider redirects to `myapp://oauth/callback?code=...`. The app receives that URL and exchanges the code for tokens.

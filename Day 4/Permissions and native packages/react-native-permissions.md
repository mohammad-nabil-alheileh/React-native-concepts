# react-native-permissions

Each platform has its own permission names, states, and rules. `react-native-permissions` is a library that gives you one JavaScript API for both. You write `check` and `request` once, and the library handles the Android and iOS differences.

You still need the native setup: manifest entries on Android, Info.plist keys on iOS. The library does not replace them.

## The main API

| Function | What it does |
| --- | --- |
| `check(permission)` | Reads the current state. Never shows a popup. |
| `request(permission)` | Shows the system popup if allowed, and returns the new state. |
| `checkMultiple([...])` / `requestMultiple([...])` | Same, for several permissions at once. |
| `openSettings()` | Opens the app’s page in system Settings. |

Permission names live in `PERMISSIONS.ANDROID.*` and `PERMISSIONS.IOS.*`.

## Example: one helper for both platforms

```js
import { Platform } from 'react-native';
import {
  check, request, openSettings, PERMISSIONS, RESULTS,
} from 'react-native-permissions';

const CAMERA = Platform.select({
  android: PERMISSIONS.ANDROID.CAMERA,
  ios: PERMISSIONS.IOS.CAMERA,
});

export async function ensureCamera() {
  let status = await check(CAMERA);

  if (status === RESULTS.DENIED) {
    status = await request(CAMERA);   // shows the system popup
  }

  if (status === RESULTS.BLOCKED) {
    await openSettings();             // only Settings can fix this
  }

  return status === RESULTS.GRANTED || status === RESULTS.LIMITED;
}
```

## A small hook

```js
import { useEffect, useState } from 'react';
import { AppState } from 'react-native';
import { check, RESULTS } from 'react-native-permissions';

export function usePermission(permission) {
  const [status, setStatus] = useState(null);

  useEffect(() => {
    const read = () => check(permission).then(setStatus);
    read();
    // re-check when the user returns from Settings
    const sub = AppState.addEventListener('change', s => s === 'active' && read());
    return () => sub.remove();
  }, [permission]);

  return status;
}
```

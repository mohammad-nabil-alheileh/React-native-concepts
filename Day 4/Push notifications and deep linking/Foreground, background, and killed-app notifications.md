# Foreground, background, and killed-app notifications

push notification can arrive when your app is in one of three states. The system and your code behave differently in each.

## The three states

| State | What it means |
| --- | --- |
| Foreground | The app is open and visible on screen |
| Background | The app was opened, then the user pressed Home or switched apps. The process is alive but not visible |
| Killed (quit) | The user swiped it away from recents, or the OS ended it. No process is running |

## Two kinds of messages

- **Notification message:** contains `notification: {title, body}`. The system can display it for you.
- **Data message:** contains only `data: {...}`. Nothing is displayed unless your code does it.

## What happens in each state

| State | Notification message | Data-only message |
| --- | --- | --- |
| Foreground | Not shown automatically. Your `onMessage` handler runs, and you decide whether to show anything | `onMessage` runs |
| Background | The system tray shows it automatically. Your JS does not run until the user taps it | Your background handler runs (Android: needs high priority. iOS: needs `content-available`) |
| Killed | The system tray shows it automatically | Android: background handler can run, if the app was not force-stopped. iOS: silent pushes generally do not wake a swiped-away app |

The rule to remember: in the foreground, you are always responsible for displaying it. In the background and killed states, the OS displays notification messages for you.

## Foreground: `onMessage`

```js
import { useEffect } from 'react';
import { getApp } from '@react-native-firebase/app';
import { getMessaging, onMessage } from '@react-native-firebase/messaging';
import notifee from '@notifee/react-native';

const messaging = getMessaging(getApp());

export function useForegroundPush() {
  useEffect(() => {
    const unsubscribe = onMessage(messaging, async (remoteMessage) => {
      await notifee.displayNotification({
        title: remoteMessage.notification?.title,
        body: remoteMessage.notification?.body,
        data: remoteMessage.data,
        android: { channelId: 'orders' },   // channel must exist first
      });
    });
    return unsubscribe;
  }, []);
}
```

Here are your options in the foreground:

- Show a system-style notification (Notifee).
- Show an in-app banner or toast instead.
- Update state silently, for example by refreshing a badge or reloading the orders list.

## Background and killed: `setBackgroundMessageHandler`

```js
// index.js (the ROOT file, outside any component)
import { AppRegistry } from 'react-native';
import { getApp } from '@react-native-firebase/app';
import { getMessaging, setBackgroundMessageHandler } from '@react-native-firebase/messaging';
import notifee from '@notifee/react-native';
import App from './App';
import { name as appName } from './app.json';

setBackgroundMessageHandler(getMessaging(getApp()), async (remoteMessage) => {
  // Runs with NO UI mounted: no hooks, no navigation, no React state
  await notifee.displayNotification({
    title: remoteMessage.data?.title,
    body: remoteMessage.data?.body,
    data: remoteMessage.data,
    android: { channelId: 'orders' },
  });
});

AppRegistry.registerComponent(appName, () => App);
```

# Notification tap handling

Showing a notification is half the job. When the user taps it, your app should open and know which notification was tapped, so it can react to the data you sent. Detecting the tap depends on the app state and on who displayed the notification. This topic covers detecting the tap and reading its data.

## Who handles the tap?

| Notification was displayed by | App state when tapped | API that fires |
| --- | --- | --- |
| The system (a message with a `notification` field) | Background | `onNotificationOpenedApp` |
| The system | Killed | `getInitialNotification` |
| Your code (Notifee, from `onMessage` or a background handler) | Foreground | `notifee.onForegroundEvent` |
| Your code | Background | `notifee.onBackgroundEvent` |
| Your code | Killed | `notifee.getInitialNotification` |

## Case 1: System-displayed notifications

```js
import { useEffect } from 'react';
import { getApp } from '@react-native-firebase/app';
import {
  getMessaging,
  onNotificationOpenedApp,
  getInitialNotification,
} from '@react-native-firebase/messaging';

const messaging = getMessaging(getApp());

export function useNotificationTaps(handleTap) {
  useEffect(() => {
    // App was in the background, user tapped the notification
    const unsubscribe = onNotificationOpenedApp(messaging, (remoteMessage) => {
      handleTap(remoteMessage);
    });

    // App was killed, and the tap launched it
    getInitialNotification(messaging).then((remoteMessage) => {
      if (remoteMessage) handleTap(remoteMessage);
    });

    return unsubscribe;
  }, []);
}
```

- `getInitialNotification` returns the message once. After you read it, it returns `null`.
- `onNotificationOpenedApp` fires only when the app was in the background. It does not fire for foreground taps or cold starts.
- Neither one fires for notifications you displayed yourself with Notifee, because those aren’t remote messages.

## Case 2: Notifications you display yourself (Notifee)

When you display from `onMessage` (foreground) or `setBackgroundMessageHandler` (data-only messages), Notifee owns the tap.

First, give the notification a press action:

```js
await notifee.displayNotification({
  title: 'Order shipped',
  body: 'Order #99 is on its way',
  data: remoteMessage.data,                       // keep your routing data here
  android: {
    channelId: 'orders',
    pressAction: { id: 'default' },               // required to open the app on tap
  },
});
```

Then listen for presses:

```js
import notifee, { EventType } from '@notifee/react-native';

// Foreground (inside a component or hook)
useEffect(() => {
  return notifee.onForegroundEvent(({ type, detail }) => {
    if (type === EventType.PRESS) handleTap({ data: detail.notification?.data });
  });
}, []);

// Background (top level of index.js, like the message handler from topic 10)
notifee.onBackgroundEvent(async ({ type, detail }) => {
  if (type === EventType.PRESS) handleTap({ data: detail.notification?.data });
});

// Killed (read once at startup)
const initial = await notifee.getInitialNotification();
if (initial) handleTap({ data: initial.notification?.data });
```

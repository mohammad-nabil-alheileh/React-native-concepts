# Deep linking from a notification

The simplest design is to put a deep link URL in the notification’s `data`, then treat a tap as if the user had opened that URL.

```text
Backend: data: { url: "myapp://order/99" }
    │
    ▼
Notification shown (system or Notifee)
    │ user taps
    ▼
Tap handler reads data.url
    │
    ▼
React Navigation linking config  maps "order/99" → Order screen, id = '99'
```

## Step 1: Send the URL in the payload

Keep the URL in `data`, and keep it as a full URL.

## Step 2: Feed notification taps into React Navigation’s linking

```js
import { Linking } from 'react-native';
import { getApp } from '@react-native-firebase/app';
import {
  getMessaging,
  getInitialNotification,
  onNotificationOpenedApp,
} from '@react-native-firebase/messaging';
import notifee, { EventType } from '@notifee/react-native';

const messaging = getMessaging(getApp());

const linking = {
  prefixes: ['myapp://', 'https://example.com'],
  config,

  // COLD START: where did the app launch from?
  async getInitialURL() {
    // 1. A normal deep link
    const url = await Linking.getInitialURL();
    if (url != null) return url;

    // 2. A system-displayed notification tap
    const message = await getInitialNotification(messaging);
    if (message?.data?.url) return message.data.url;

    // 3. A Notifee notification tap
    const initial = await notifee.getInitialNotification();
    return initial?.notification?.data?.url;
  },

  // RUNNING APP: background or foreground
  subscribe(listener) {
    // Normal deep links
    const linkSub = Linking.addEventListener('url', ({ url }) => listener(url));

    // System-displayed notification tapped while the app was in the background
    const unsubscribeFcm = onNotificationOpenedApp(messaging, (message) => {
      const url = message?.data?.url;
      if (url) listener(url);
    });

    // Notifee notification tapped while the app is in the foreground
    const unsubscribeNotifee = notifee.onForegroundEvent(({ type, detail }) => {
      if (type === EventType.PRESS) {
        const url = detail.notification?.data?.url;
        if (url) listener(url);
      }
    });

    return () => {
      linkSub.remove();
      unsubscribeFcm();
      unsubscribeNotifee();
    };
  },
};
```

# FCM and the device token

Your server needs a way to say “send this notification to that phone.” FCM solves this with a **device token**, a unique string that identifies one app installation on one device. Your app gets the token from FCM, you store it on your backend, and your backend uses it to target that device.

## The flow

```text
1. App starts      ──asks──>  FCM       "who am I?"
2. FCM             ──gives──> App       device token: "fGx3...long string"
3. App             ──sends──> Your backend (saves token + user ID)
4. Your backend    ──sends──> FCM       "deliver this message to token fGx3..."
5. FCM             ──pushes─> Device    notification arrives
```

Your server never talks to the phone directly. FCM keeps a persistent connection to the device and relays the message.

## Getting the token in React Native

```js
import { getApp } from '@react-native-firebase/app';
import { getMessaging, getToken } from '@react-native-firebase/messaging';

const messaging = getMessaging(getApp());

async function registerDevice() {
  const token = await getToken(messaging);
  console.log('FCM token:', token);
  await api.post('/devices', { token, platform: 'android' });
}
```

## Tokens change, so listen for refreshes

The token is not permanent. FCM can issue a new one, and your stored copy then becomes useless.

A token can change when:

- The app is reinstalled
- The user clears app data
- The user restores the app on a new device
- FCM rotates it (for security, or after long inactivity)

Request the token on every app start and compare it with what the server has.

## Notification messages vs data messages

| | Notification message | Data message |
| --- | --- | --- |
| Fields | `notification: {title, body}` | `data: {...}` only |
| When app is in background or killed | The system tray shows it automatically | Nothing is shown. Your code must handle it and display something |
| When app is in foreground | Your code receives it (system tray doesn’t show it) | Your code receives it |
| Control over display | Limited | Full |

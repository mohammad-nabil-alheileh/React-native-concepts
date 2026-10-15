# APNs

APNs (Apple Push Notification service) is Apple’s delivery service for push notifications. On iOS, every push goes through APNs, with no exceptions. Android has FCM as its own delivery service, and iOS has APNs.

On Android, FCM delivers straight to the device. On iOS, FCM is a middleman and APNs does the last hop.

## Two tokens on iOS

- **APNs device token:** Apple issues this to your app installation.
- **FCM registration token:** FCM maps itself to that APNs token. This is the token you store on your backend.

```js
import { getMessaging, getAPNSToken, getToken } from '@react-native-firebase/messaging';

const apns = await getAPNSToken(messaging);   // iOS only, null on Android
const fcm = await getToken(messaging);        // the one you send to your server
```

## Common errors

| Error | Usual cause |
| --- | --- |
| `BadDeviceToken` | Token from the wrong environment (sandbox vs production) |
| `Unregistered` | App uninstalled or the token is dead |
| `InvalidProviderToken` | Wrong Key ID, Team ID, or `.p8` file in Firebase |
| `PayloadTooLarge` | Over 4 KB |

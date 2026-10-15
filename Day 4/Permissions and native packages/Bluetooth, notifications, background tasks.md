# Bluetooth, notifications, background tasks

They are less about reading one resource and more about talking to hardware, reaching the user, and running when the app is closed.

| Feature | Android | iOS |
| --- | --- | --- |
| Bluetooth | `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT` (Android 12+) | `NSBluetoothAlwaysUsageDescription` |
| Notifications | `POST_NOTIFICATIONS` (Android 13+), runtime request | Runtime request, no Info.plist key |
| Background tasks | Foreground service, WorkManager | Background Modes capability, system-scheduled tasks |

## Bluetooth

**Main idea:** connecting to nearby devices such as headphones, fitness bands, sensors, and printers.

```js
const result = await PermissionsAndroid.requestMultiple([
  PermissionsAndroid.PERMISSIONS.BLUETOOTH_SCAN,
  PermissionsAndroid.PERMISSIONS.BLUETOOTH_CONNECT,
]);
```

## Notifications

**Main idea:** showing messages outside your app. There are two kinds:

- Local notifications are scheduled by the app itself (a reminder like “water your plant”).
- Push notifications are sent from a server through FCM (Android) or APNs (iOS).

## Background tasks

**Main idea:** running code when the app is not on screen, for example syncing data, uploading a file, or tracking a run. This is less a “permission” and more a negotiation with the OS, because both platforms limit background work to save battery.

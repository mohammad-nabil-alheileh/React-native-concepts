# Notification permission on Android 13+

Before Android 13, every app could show notifications with no questions asked. Android 13 (API 33) added a runtime permission, `POST_NOTIFICATIONS`, so users now have to opt in, as they already did on iOS.

## How it behaves

| Situation | What happens |
| --- | --- |
| App targets API 33+ on Android 13+ | You declare the permission and request it at runtime |
| Android 12 or lower | No permission exists, and notifications are always allowed |
| App installed before the phone upgraded to Android 13 | Usually granted automatically, but the user can still turn it off in settings |
| User taps “Don’t allow” twice | Android stops showing the dialog. It is permanently blocked until the user goes to Settings |

## Step 1: Declare it in the manifest

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

## Step 2: Request it at runtime

### Option A: React Native’s built-in `PermissionsAndroid`

```js
import { PermissionsAndroid, Platform } from 'react-native';

async function requestNotificationPermission() {
  if (Platform.OS !== 'android' || Platform.Version < 33) {
    return true; // nothing to ask on older Android
  }

  const result = await PermissionsAndroid.request(
    PermissionsAndroid.PERMISSIONS.POST_NOTIFICATIONS
  );
  return result === PermissionsAndroid.RESULTS.GRANTED;
}
```

### Option B: React Native Firebase

```js
import { getApp } from '@react-native-firebase/app';
import { getMessaging, requestPermission, AuthorizationStatus } from '@react-native-firebase/messaging';

const messaging = getMessaging(getApp());

async function askForPush() {
  const status = await requestPermission(messaging);
  return (
    status === AuthorizationStatus.AUTHORIZED ||
    status === AuthorizationStatus.PROVISIONAL
  );
}
```

## Step 3: Check the current status

Check before asking, so you don’t re-prompt a user who already agreed:

## Step 4: Handle “blocked” by sending users to Settings

# Android runtime permissions

On Android, a permission has two parts:

1. **Declare it in `AndroidManifest.xml`.** This tells the OS and Google Play what your app might use.
2. **Request it at runtime (Android 6+) for “dangerous” permissions.** This is the popup the user sees.

If you skip step 1, the request silently fails. If you skip step 2, the feature is denied. You need both.

## Normal vs dangerous permissions

| Type | Examples | What you need to do |
| --- | --- | --- |
| Normal | internet, vibrate | Declaring them in the manifest is enough. They are granted automatically at install. |
| Dangerous | camera, location, microphone, contacts | Declared in the manifest and requested at runtime. |

## Requesting at runtime

React Native ships a built-in `PermissionsAndroid` API.

```js
import { PermissionsAndroid, Platform } from 'react-native';

async function requestCamera() {
  if (Platform.OS !== 'android') return true;

  const result = await PermissionsAndroid.request(
    PermissionsAndroid.PERMISSIONS.CAMERA,
    {
      title: 'Camera access',
      message: 'We use the camera to scan plant tags.',
      buttonPositive: 'Allow',
      buttonNegative: 'Not now',
    }
  );

  // result is 'granted' | 'denied' | 'never_ask_again'
  return result === PermissionsAndroid.RESULTS.GRANTED;
}
```

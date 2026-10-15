# Permission states

Before an app touches a sensitive resource (camera, location, and so on), the OS makes the user approve it. At any moment, each permission is in one state, and your code must handle every state.

| State | Meaning | Can you ask again? |
| --- | --- | --- |
| `granted` | The user said yes | Not needed |
| `denied` | Not granted yet, or the user said no once | Yes (Android). On iOS, a “no” is final after the first prompt |
| `blocked` | The user chose “Don’t ask again”, or the OS forbids asking | No. Only the Settings app can change it |
| `limited` | Partial access (iOS “Selected Photos”, approximate location; Android 14 partial photo access) | Can be upgraded, but you must work with what you have |

Libraries such as `react-native-permissions` also add `unavailable`, meaning the device has no such feature (for example, no camera).

## Example

```js
import { Alert, Linking } from 'react-native';
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

async function ensureCamera() {
  const status = await check(PERMISSIONS.ANDROID.CAMERA);

  switch (status) {
    case RESULTS.GRANTED:
      return true;                       // use the camera
    case RESULTS.DENIED: {
      const next = await request(PERMISSIONS.ANDROID.CAMERA);
      return next === RESULTS.GRANTED;   // show the system prompt
    }
    case RESULTS.BLOCKED:
      Alert.alert('Camera needed', 'Enable it in Settings.', [
        { text: 'Open Settings', onPress: () => Linking.openSettings() },
      ]);
      return false;
    case RESULTS.LIMITED:
      return true;                       // works, but with partial access
    case RESULTS.UNAVAILABLE:
      return false;                      // no camera on this device
  }
}
```

The key point is that `blocked` can’t be fixed from code. The only thing you can do is explain why you need the permission and send the user to Settings with `Linking.openSettings()`.

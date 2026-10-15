# Camera, location, photos, microphone

| Resource | Android permission | iOS Info.plist key |
| --- | --- | --- |
| Camera | `CAMERA` | `NSCameraUsageDescription` |
| Location | `ACCESS_COARSE_LOCATION`, `ACCESS_FINE_LOCATION` | `NSLocationWhenInUseUsageDescription` |
| Photos | `READ_MEDIA_IMAGES` (Android 13+), `READ_EXTERNAL_STORAGE` (older) | `NSPhotoLibraryUsageDescription` |
| Microphone | `RECORD_AUDIO` | `NSMicrophoneUsageDescription` |

```js
import { Platform } from 'react-native';
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

const P = {
  camera: Platform.select({
    android: PERMISSIONS.ANDROID.CAMERA,
    ios: PERMISSIONS.IOS.CAMERA,
  }),
  location: Platform.select({
    android: PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION,
    ios: PERMISSIONS.IOS.LOCATION_WHEN_IN_USE,
  }),
  microphone: Platform.select({
    android: PERMISSIONS.ANDROID.RECORD_AUDIO,
    ios: PERMISSIONS.IOS.MICROPHONE,
  }),
  photos: Platform.select({
    android:
      Platform.Version >= 33
        ? PERMISSIONS.ANDROID.READ_MEDIA_IMAGES
        : PERMISSIONS.ANDROID.READ_EXTERNAL_STORAGE,
    ios: PERMISSIONS.IOS.PHOTO_LIBRARY,
  }),
};

export async function ensure(kind) {
  let status = await check(P[kind]);
  if (status === RESULTS.DENIED) status = await request(P[kind]);
  return status; // let the caller decide what to do with BLOCKED / LIMITED
}
```

## Camera

**Main idea:** access to take photos, record video, or scan codes.

- Camera permission alone does not let you save to the gallery. That is a separate photo permission.
- Recording video with sound needs camera and microphone.

## Location

**Main idea:** the device’s position.

- Coarse (approximate) gives a city-level area. Fine (precise) gives an exact spot

## Photos

**Main idea:** reading from, or saving to, the user’s photo library.

## Microphone

**Main idea:** recording audio, for voice notes, calls, or video with sound.

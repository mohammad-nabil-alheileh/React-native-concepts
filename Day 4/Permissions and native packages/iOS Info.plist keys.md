# iOS Info.plist keys

iOS has no manifest where you list permissions. Instead, `Info.plist` holds a usage description for each protected resource: a short sentence the OS shows inside the permission popup.

The rule is strict: if your code touches the camera and the matching key is missing, iOS crashes the app immediately. This is different from Android, where a missing declaration fails silently.

## Where the file lives

```xml
<key>NSCameraUsageDescription</key>
<string>We use the camera to scan plant tags.</string>

<key>NSLocationWhenInUseUsageDescription</key>
<string>We use your location to show nearby plant shops.</string>

<key>NSPhotoLibraryUsageDescription</key>
<string>Choose a photo of your plant from your library.</string>
```

Each `<key>` is followed by a `<string>` holding the text the user will see.

## Keys you should recognize

| Key | Resource |
| --- | --- |
| `NSCameraUsageDescription` | Camera |
| `NSMicrophoneUsageDescription` | Microphone |
| `NSPhotoLibraryUsageDescription` | Read the photo library |
| `NSPhotoLibraryAddUsageDescription` | Only save photos to the library |
| `NSLocationWhenInUseUsageDescription` | Location while the app is open |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Location in the background too |
| `NSBluetoothAlwaysUsageDescription` | Bluetooth |
| `NSContactsUsageDescription` | Contacts |

Notifications are the exception. There is no Info.plist key. You request them at runtime and enable the Push Notifications capability in Xcode.

# APK vs AAB

Both are file formats that package the Android app.

- **APK (Android Package)** is a ready-to-install file. You can send it to a phone, tap it, and the app installs. It contains everything for every device type.
- **AAB (Android App Bundle)** is a publishing file. You cannot install it directly. You upload it to the Play Store, and Google builds optimized APKs for each user’s device from it.

## Why AAB exists

A universal APK has to carry resources for every phone:

- All CPU architectures
- All screen densities
- All languages

Your phone needs only one of each. With an AAB, Google Play splits the app and delivers just the parts that your device needs.

## Side-by-side

| | APK | AAB |
| --- | --- | --- |
| Installable directly | Yes | No |
| Play Store upload | Not accepted for new apps | Required for new apps |
| Size for the user | Larger (universal) | Smaller (split per device) |
| Who builds the final installable | You | Google Play |
| Signing | You sign it | You sign with the upload key, Google re-signs |

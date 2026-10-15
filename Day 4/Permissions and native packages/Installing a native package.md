# Installing a native package

React Native packages come in two kinds:

| Kind | Examples | Details |
| --- | --- | --- |
| JS-only | lodash, date-fns, axios | Pure JavaScript. Install them and Metro picks them up instantly. |
| Native | react-native-permissions, camera, Bluetooth libraries | Contain Kotlin/Java code for Android and Swift/Obj-C code for iOS, plus JavaScript that talks to it. |

Native code has to be compiled into the app binary. That’s why you can’t just install it and hit Fast Refresh.

## The three steps

1. **Install the package.**
2. **Autolinking connects the native code for you.** Autolinking is built into the React Native CLI. When you build, it scans `node_modules` and automatically wires each native package into the Android Gradle project and the iOS project.
3. **Rebuild the app.**

   ```bash
   # Android
   npx react-native run-android

   # iOS (macOS only)
   cd ios && pod install && cd ..
   npx react-native run-ios
   ```

## Things to know

- **Reloading is not rebuilding.** Metro reload and Fast Refresh only update JavaScript. A new native package needs a full rebuild.
- **Stale builds cause strange bugs.** If a rebuild still fails, try a clean build

Some packages need manual setup even with autolinking, such as adding permissions to the manifest

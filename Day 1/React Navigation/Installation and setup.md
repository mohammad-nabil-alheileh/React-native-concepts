# Installation and Setup

## 1. Create the Project

```bash
npx @react-native-community/cli init MyApp
cd MyApp
```

This generates a bare project with `android/` and `ios/` folders.

## 2. Install the Core and Native Dependencies

```bash
npm install @react-navigation/native react-native-screens react-native-safe-area-context react-native-gesture-handler
```

| Package                          | Role                                   | Needed for                          |
| -------------------------------- | -------------------------------------- | ----------------------------------- |
| `@react-navigation/native`       | The core of React Navigation           | Everything                          |
| `react-native-screens`           | Real native screen containers          | All navigators                      |
| `react-native-safe-area-context` | Gives you the safe area insets         | Headers, screens without a header   |
| `react-native-gesture-handler`   | Handles gestures on the native side    | The Drawer          |

**What each one does and why it exists:**

- **`@react-navigation/native`:** the core. It provides `NavigationContainer`, hooks like `useNavigation`, and the linking and back-button logic.

- **`react-native-screens`:** by default, React Native would render each screen as a plain `View`, and every screen you've visited stays mounted in the view hierarchy. This library maps each screen to a real native screen container (a `Fragment` on Android, a `UIViewController` on iOS). Inactive screens can then be detached, which saves memory and makes transitions use the platform's native animations.

- **`react-native-safe-area-context`:** phones have notches, rounded corners, status bars, and gesture bars. This library tells you the exact insets so content doesn't draw underneath them. Navigator headers use it internally, and you use it in screens without a header.

- **`react-native-gesture-handler`** (extra): React Native's built-in touch system runs on the JS thread, so gestures can feel laggy. This library handles gestures on the native side. The Drawer (swipe from the edge) needs it. The native stack alone does not.

## 3. Link Native Code and Rebuild

**Android:** autolinking detects native modules in `node_modules` automatically. You don't edit Gradle files. You just rebuild:

```bash
npx react-native run-android
```

**Why a rebuild matters:** Metro only bundles JavaScript. Native libraries are compiled into the APK. If you install a native package and only reload, the JS calls native code that doesn't exist, and you get errors like:

```
Invariant Violation: requireNativeComponent: "RNSScreen" was not found
```

> **Rule:** new native dependency means rebuild.

**iOS:** needs `cd ios && pod install`, but that requires macOS.

| Platform | What to run after installing a native package |
| -------- | --------------------------------------------- |
| Android  | `npx react-native run-android`                |
| iOS      | `cd ios && pod install`, then rebuild         |

## 4. Two Small Native Config Steps

## Gesture Handler Import

Put this as the **first line** of `index.js`:

```js
import 'react-native-gesture-handler';
import { AppRegistry } from 'react-native';
import App from './App';
import { name as appName } from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

**Why first:** the library must initialize before any other code runs. Otherwise gesture-based features can crash in production builds even if they work in development.

## 5. Install a Navigator

```bash
npm install @react-navigation/native-stack
```
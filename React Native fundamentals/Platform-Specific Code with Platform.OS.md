# Platform-Specific Code with Platform.OS

## 1. Platform.OS

It returns a string: `'ios'`, `'android'` (and `'web'` if you use React Native Web).

```jsx
function Hello() {
    return (
        <Text>
            {Platform.OS === 'ios' ? 'Hello from iPhone' : 'Hello from Android'}
        </Text>
    );
}
```

## 2. Platform.select

It takes an object and returns the value for the current platform.

```jsx
const styles = StyleSheet.create({
    card: {
        padding: 16,
        borderRadius: 12,
        ...Platform.select({
            ios: {
                shadowColor: '#000',
                shadowOffset: { width: 0, height: 2 },
                shadowOpacity: 0.2,
                shadowRadius: 4,
            },
            android: {
                elevation: 4,
            },
        }),
    },
    title: {
        fontFamily: Platform.select({
            ios: 'Helvetica Neue',
            android: 'Roboto',
            default: 'System',
        }),
    },
});
```

**Useful keys:**

- `ios` and `android`: one for each platform.
- `native`: matches both iOS and Android (as opposed to web).
- `default`: the fallback for anything not listed.

If no key matches and there is no `default`, it returns `undefined`.

## 3. Selecting Whole Components

```jsx
const Button = Platform.select({
    ios: () => require('./IOSButton').default,
    android: () => require('./AndroidButton').default,
})();
```

## 4. Platform-Specific Files

```
components/
    DatePicker.ios.js
    DatePicker.android.js
    Header.native.js   // iOS and Android
    Header.web.js      // web only
```

```jsx
import DatePicker from './components/DatePicker';
// iOS loads DatePicker.ios.js, Android loads DatePicker.android.js
```

## 5. Other Platform Values

```jsx
Platform.Version;
// iOS: a string like '17.2'
// Android: a number, the API level (like 34)

if (Platform.OS === 'android' && Platform.Version >= 33) {
    // Android 13 or newer
}

Platform.isPad; // true on iPad (iOS only)
Platform.isTV;  // true on tvOS/Android TV
```

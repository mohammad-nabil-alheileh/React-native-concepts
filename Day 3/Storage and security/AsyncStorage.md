# AsyncStorage

AsyncStorage is a simple, asynchronous, persistent key-value store for React Native. It’s the mobile equivalent of localStorage in the browser. Data survives app restarts but is removed when the app is uninstalled.

- Keys and values are strings only.
- Every call is async (it returns a Promise).
- It is not encrypted.

## Examples

### Store and read a string

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

await AsyncStorage.setItem('language', 'ar');
const lang = await AsyncStorage.getItem('language'); // 'ar' (or null if missing)
```

### Store an object (it must be serialized)

```javascript
const settings = { theme: 'dark', fontSize: 16 };
await AsyncStorage.setItem('settings', JSON.stringify(settings));

const raw = await AsyncStorage.getItem('settings');
const parsed = raw ? JSON.parse(raw) : null;
```

### Remove, and batch operations

```javascript
await AsyncStorage.removeItem('language');

await AsyncStorage.multiSet([['a', '1'], ['b', '2']]);
const pairs = await AsyncStorage.multiGet(['a', 'b']); // [['a','1'], ['b','2']]

await AsyncStorage.clear(); // wipes EVERYTHING the app stored here
```

## Good use cases

- Theme
- Language preference
- Cached non-sensitive data such as UI settings or a last-viewed tab

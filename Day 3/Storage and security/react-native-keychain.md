# react-native-keychain

react-native-keychain is the library that connects your JavaScript to the secure storage from keychain or keystore. You write one API, and it picks the right backend:

## The API

### Generic password (use this for tokens)

```javascript
// Save
await Keychain.setGenericPassword('user', refreshToken, { service: 'refresh-token' });

// Read -> { username, password, service, storage } or false
const creds = await Keychain.getGenericPassword({ service: 'refresh-token' });
if (creds) {
  console.log(creds.password);
}

// Delete
await Keychain.resetGenericPassword({ service: 'refresh-token' });
```

### Internet credentials (tied to a server)

```javascript
await Keychain.setInternetCredentials('api.myapp.com', 'user', 'pass');
const c = await Keychain.getInternetCredentials('api.myapp.com');
await Keychain.resetInternetCredentials({ server: 'api.myapp.com' });
```

### Other helpers

```javascript
await Keychain.hasGenericPassword({ service: 'refresh-token' }); // boolean
await Keychain.getAllGenericPasswordServices();                  // list of service names
await Keychain.getSupportedBiometryType();                       // 'FaceID', 'TouchID', 'Fingerprint', or null
```

| Helper | Returns |
| --- | --- |
| `hasGenericPassword({ service })` | `boolean` |
| `getAllGenericPasswordServices()` | List of service names |
| `getSupportedBiometryType()` | `'FaceID'`, `'TouchID'`, `'Fingerprint'`, or `null` |

## Storing more than one value

Each service holds one username/password pair. Use separate services, or store a JSON string:

```javascript
await Keychain.setGenericPassword('session', JSON.stringify({ refresh, deviceId }), {
  service: 'session',
});
const data = JSON.parse(creds.password);
```

## Complete example: a small token helper

```javascript
// secureTokens.js
import * as Keychain from 'react-native-keychain';

const SERVICE = 'refresh-token';

export async function saveRefreshToken(token) {
  try {
    await Keychain.setGenericPassword('refresh', token, {
      service: SERVICE,
      accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
    });
    return true;
  } catch (e) {
    console.warn('Secure save failed'); // never log the token itself
    return false;
  }
}

export async function loadRefreshToken() {
  try {
    const creds = await Keychain.getGenericPassword({ service: SERVICE });
    return creds ? creds.password : null;
  } catch (e) {
    return null; // treat any failure as "not logged in"
  }
}

export async function deleteRefreshToken() {
  await Keychain.resetGenericPassword({ service: SERVICE });
}
```

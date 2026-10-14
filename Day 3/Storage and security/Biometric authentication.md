# Biometric authentication

Biometric authentication lets the user prove who they are with a fingerprint, Face ID, or face unlock instead of typing a password. Your app never sees the biometric data. The OS does the matching and only tells you, or the secure store, whether it succeeded.

## Two approaches

| | A. UI-only check | B. Biometrics bound to the secret |
| --- | --- | --- |
| How | A library shows a prompt and returns true or false | The token is stored with a biometric requirement, and the OS refuses to release it until the user authenticates |
| Security | Weak. On a rooted or hooked device, the true can be faked | Strong. No real biometric means no token |
| Use for | Hiding a screen | Protecting tokens |

## Steps

### 1. Check what the device supports

```javascript
const type = await Keychain.getSupportedBiometryType();
// 'FaceID' | 'TouchID' | 'OpticID' | 'Fingerprint' | 'Face' | 'Iris' | null
if (!type) {
  // no biometrics available or enrolled: offer password/PIN login
}
```

### 2. Store the refresh token so a biometric is required to read it

```javascript
await Keychain.setGenericPassword('refresh', refreshToken, {
  service: 'refresh-token',
  accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_CURRENT_SET,
  accessible: Keychain.ACCESSIBLE.WHEN_PASSCODE_SET_THIS_DEVICE_ONLY,
});
```

### 3. Save the token after a normal login, with a fallback if biometrics are unavailable

```javascript
async function saveToken(token) {
  const biometry = await Keychain.getSupportedBiometryType();
  await Keychain.setGenericPassword('refresh', token, {
    service: 'refresh-token',
    accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
    ...(biometry && {
      accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_ANY_OR_DEVICE_PASSCODE,
    }),
  });
}
```

## Access control options

| Option | Meaning |
| --- | --- |
| `BIOMETRY_ANY` | Any enrolled biometric. Survives new fingerprints being added. |
| `BIOMETRY_CURRENT_SET` | Only the biometrics enrolled now. Adding a new fingerprint makes the item unreadable. Safer. |
| `BIOMETRY_ANY_OR_DEVICE_PASSCODE` | Biometric, or the device passcode as a fallback |
| `DEVICE_PASSCODE` | Device passcode only |

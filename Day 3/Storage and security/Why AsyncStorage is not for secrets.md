# Why AsyncStorage is not for secrets

AsyncStorage writes your data to the device as plain text, with no encryption. Anyone who can read the app’s storage can read your values directly, with no key or password needed.

## Where the data actually lives

| Platform | Location |
| --- | --- |
| Android | A SQLite database file at `/data/data/<your.package>/databases/RKStorage` |
| iOS | Plain files in the app’s Documents directory (`RCTAsyncLocalStorage_V1`) |

## How an attacker can read it

1. **Rooted or jailbroken device.** The OS sandbox is bypassed, so any app or user can open your app’s files.
2. **Backups.** On Android, if `android:allowBackup="true"`, data can be pulled with `adb backup`. iOS backups that are not encrypted can also be extracted.
3. **Debuggable builds.** On a debug build, this reads everything with no root
4. **Malicious JavaScript.** Any code running in your JS bundle can call `AsyncStorage.getItem('authToken')`. That includes a compromised npm package, which is a real supply-chain risk.
5. **Physical access.** Someone with the unlocked phone and a computer can copy the app data.

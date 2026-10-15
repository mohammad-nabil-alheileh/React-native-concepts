# URL schemes and Android App Links

A deep link is a URL that opens your app on a specific screen instead of just launching it, There are two ways to do this on Android.

## 1. Custom URL scheme

You invent your own protocol (`myapp://`) and tell the OS that your app handles it.

**Android** (`AndroidManifest.xml`, inside your `<activity>`):

```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" android:host="product" />
</intent-filter>
```

**iOS** (`Info.plist`):

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array><string>myapp</string></array>
    </dict>
</array>
```

## 2. Android App Links (verified HTTPS links)

Instead of a made-up scheme, your app claims a real web URL like `https://example.com/product/42`. Android verifies that you own the domain, so the link opens your app directly with no chooser dialog.

### Step 1: Add an intent filter with `autoVerify`

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https"
            android:host="example.com"
            android:pathPrefix="/product" />
</intent-filter>
```

### Step 2: Host a file at `https://example.com/.well-known/assetlinks.json` proving you own the app

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.myapp",
    "sha256_cert_fingerprints": ["AA:BB:CC:...your signing cert..."]
  }
}]
```

## Reading the URL in JavaScript

```js
import { Linking } from 'react-native';

// App was launched by the link (cold start)
const url = await Linking.getInitialURL();

// App was already running and received a link
const sub = Linking.addEventListener('url', ({ url }) => {
  console.log('Opened with:', url);
});
// later: sub.remove();

// Open a link yourself
Linking.openURL('myapp://product/42');
```

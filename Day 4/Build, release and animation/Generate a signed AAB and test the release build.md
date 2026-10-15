# Generate a signed AAB and test the release build

AAB (Android App Bundle) is the publishing format Google Play requires for new apps. You upload one file, and Google generates optimized APKs for each device (screen density, CPU architecture, language).

Signed means the bundle is cryptographically stamped with a key you own. Android refuses to install or update unsigned apps, and Play uses the signature to verify updates come from you.

Release build is the production version: JavaScript is bundled and minified into the app, Hermes compiles it to bytecode, and there is no Metro dev server or dev menu.

## Steps (bare React Native CLI)

### 1. Create an upload keystore (once per app)

```bash
keytool -genkeypair -v -storetype PKCS12 \
  -keystore my-upload-key.keystore \
  -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

Move the file to `android/app/`.

### 2. Store credentials outside the repo, in `~/.gradle/gradle.properties`

```properties
MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=*****
MYAPP_UPLOAD_KEY_PASSWORD=*****
```

### 3. Wire it into `android/app/build.gradle`

```groovy
android {
  signingConfigs {
    release {
      storeFile file(MYAPP_UPLOAD_STORE_FILE)
      storePassword MYAPP_UPLOAD_STORE_PASSWORD
      keyAlias MYAPP_UPLOAD_KEY_ALIAS
      keyPassword MYAPP_UPLOAD_KEY_PASSWORD
    }
  }
  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled true   // shrink/obfuscate native code
    }
  }
}
```

### 4. Build the bundle

```bash
cd android && ./gradlew bundleRelease
# output: android/app/build/outputs/bundle/release/app-release.aab
```

### 5. Test the release build

You can’t install an AAB directly, so either:

**Quick: run a release APK on a connected device**

```bash
npx react-native run-android --mode release
```

**Accurate: test the actual AAB with bundletool**

```bash
bundletool build-apks --bundle=app-release.aab --output=app.apks \
  --ks=my-upload-key.keystore --ks-key-alias=my-key-alias --mode=universal
bundletool install-apks --apks=app.apks
```

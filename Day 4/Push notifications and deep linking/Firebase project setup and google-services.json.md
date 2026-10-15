# Firebase project setup and google-services.json

Push notifications on Android go through Firebase Cloud Messaging (FCM), Google’s delivery service. Before your app can receive anything, Firebase has to know which app you are. `google-services.json` is the file that connects your Android app to your Firebase project.

## Step 1: Create the Firebase project

Go to the Firebase console and click **Add project**.

Name it. Google Analytics is optional and not needed for notifications.

## Step 2: Register your Android app

In **Project settings**, click **Add app → Android** and fill in:

| Field | Required? | Notes |
| --- | --- | --- |
| Package name | Yes | Must match your app’s `applicationId` exactly (in `android/app/build.gradle`). A mismatch is the most common setup failure. |
| App nickname | No | Optional. |
| SHA-1 fingerprint | No | Optional for plain FCM. It is required later for Google Sign-In and phone auth. |

```groovy
// android/app/build.gradle
android {
    defaultConfig {
        applicationId "com.myapp"   // <- this is the package name you register
    }
}
```

## Step 3: Download and place the file

Download `google-services.json` and put it here:

```text
android/
└── app/
    └── google-services.json
```

The file looks like this (shortened):

```json
{
  "project_info": {
    "project_number": "123456789012",
    "project_id": "my-app-1234"
  },
  "client": [{
    "client_info": {
      "mobilesdk_app_id": "1:123456789012:android:abc123",
      "android_client_info": { "package_name": "com.myapp" }
    },
    "api_key": [{ "current_key": "AIza..." }]
  }]
}
```

It holds your project ID, app ID, and API key, which is how the SDK finds your Firebase project.

## Step 4: Add the Gradle plugin (bare React Native)

```groovy
// android/build.gradle
buildscript {
    dependencies {
        classpath('com.google.gms:google-services:4.4.2')  // check Firebase docs for the current version
    }
}
```

```groovy
// android/app/build.gradle  (at the bottom)
apply plugin: 'com.google.gms.google-services'
```

## Step 5: Install the JS libraries

```bash
npm install @react-native-firebase/app @react-native-firebase/messaging
cd android && ./gradlew clean && cd ..
npx react-native run-android
```

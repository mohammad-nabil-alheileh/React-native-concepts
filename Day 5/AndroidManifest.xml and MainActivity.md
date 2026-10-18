# AndroidManifest.xml and MainActivity

`AndroidManifest.xml` Android reads it before running anything. It declares:

- The app’s package name
- The permissions the app needs (camera, internet, location)
- The activities (screens) the app contains
- Which activity is the launcher (the one opened from the home screen)
- The app name, icon, and theme

`MainActivity` is the single native Android screen that hosts your whole React Native app. React Native is not many native screens. It is one native activity, with your JavaScript UI rendered inside it.

## The manifest

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />

    <application
        android:name=".MainApplication"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/AppTheme">

        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:windowSoftInputMode="adjustResize">

        </activity>
    </application>
</manifest>
```

## Common things to edit

| Goal | Where |
| --- | --- |
| Add a permission | Manifest, `<uses-permission>` |
| Lock to portrait | Manifest, `android:screenOrientation="portrait"` |
| Fix the keyboard covering inputs | Manifest, `windowSoftInputMode` |

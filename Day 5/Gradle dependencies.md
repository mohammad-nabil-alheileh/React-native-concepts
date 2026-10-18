# Gradle dependencies

Gradle is the build tool that turns your Android project (native code, resources, and the JS bundle) into an installable app.

Dependencies are the libraries Gradle downloads and includes in that build, such as React Native itself, Hermes, Firebase, or an image library.

## Adding native libraries to Gradle

Native libraries are added to Gradle in two ways:

| Way | How it works |
| --- | --- |
| Autolinking (automatic) | When you `npm install react-native-camera`, the React Native Gradle plugin finds it in `node_modules` and adds it to the build. |
| Manually | For native Android libraries that are not npm packages, you add a line yourself. |

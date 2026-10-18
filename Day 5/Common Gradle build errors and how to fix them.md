# Common Gradle build errors and how to fix them

## The most common errors

| # | Error | Cause | Fix |
| --- | --- | --- | --- |
| 1 | SDK location not found | Gradle doesn’t know where the Android SDK is. | Create `android/local.properties` |
| 2 | Java version problems | The JDK on your machine doesn’t match what the Gradle and Android Gradle Plugin versions need. Recent React Native versions need JDK 17. | Install JDK 17 and point `JAVA_HOME` to it. |
| 3 | `Could not resolve ...` / `Could not find com.x:y:1.0` | A typo in the dependency, a missing repository, or no internet or proxy access. | Check the coordinates, confirm repositories |
| 4 | `Duplicate class ... found in modules ...` | Two libraries bring different versions of the same dependency. | Find the conflict and align versions |

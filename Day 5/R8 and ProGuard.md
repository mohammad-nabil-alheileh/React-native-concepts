# R8 and ProGuard

Both are tools that run on native Android code, when you build a release. They do three jobs:

| Job | What it does |
| --- | --- |
| Shrink | Removes code and classes that are never used |
| Obfuscate | Renames classes and methods to short meaningless names (`UserRepository` becomes `a.b`) |
| Optimize | Simplifies and inlines code to make it a bit faster |

ProGuard is the older tool. R8 is Google’s replacement.

R8 decides what is “unused” by tracing calls from the app’s entry points. It cannot see code reached in indirect ways, R8 may delete or rename some code, and the app then crashes at runtime in release only. You prevent that with keep rules.

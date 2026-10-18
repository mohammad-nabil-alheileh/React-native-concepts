# Build variants and product flavors

Three terms that fit together:

| Term | Meaning |
| --- | --- |
| Build type | How the app is built. By default you have `debug` and `release`. |
| Product flavor | Which version of the app you build, such as `dev`, `stage`, or `prod`. |
| Build variant | The combination of one flavor and one build type: flavor × build type. |

The point is to build one codebase into several apps: a dev app pointing at a test server, a stage app for QA, and a prod app for real users. Each can have its own name, ID, icon, and API URL, and all can be installed on the same phone at once.

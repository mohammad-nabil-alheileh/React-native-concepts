# Debug vs release

These are the two default build types. Same code, built with different rules.

- **debug:** a development build, made for you to run, inspect, and fix.
- **release:** a production build, made to be small, fast, signed, and shipped to users.

## Side-by-side

| | debug | release |
| --- | --- | --- |
| JavaScript | Loaded live from Metro on your computer | Bundled inside the app (Hermes bytecode) |
| Metro needed? | Yes | No |
| Fast refresh / dev menu | Yes | No |
| Error display | Red box and yellow warnings | Silent crash or nothing |
| Code shrinking | Off | Usually on |
| App size | Larger | Smaller |
| Speed | Slower (extra checks) | Fast |
| Play Store | Rejected | Accepted |

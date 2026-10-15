# Hermes

Your JavaScript needs an engine to run, just like Chrome has V8. Hermes is the JavaScript engine Meta built specifically for React Native. It is the default engine in current React Native versions, replacing JavaScriptCore (JSC). It runs on the JS thread, so it decides how fast your JS starts up and how much memory it uses.

## What makes Hermes different

1. **Ahead-of-time (AOT) bytecode compilation:** your JS is compiled to bytecode at build time. The app ships a `.hbc` file, and at launch Hermes loads the bytecode and starts running right away.
2. **No JIT compiler:** Hermes does not use just-in-time compilation. It trades peak speed on long-running heavy computation for faster startup and lower memory.
3. **Small memory footprint and efficient garbage collection:** Hermes uses a garbage collector called Hades, which does most of its work concurrently.

## Where you feel the difference

| Area | What changes |
| --- | --- |
| Cold start time (time to interactive) | Improves, because there is no parsing or compiling at launch. |
| App size on Android | Often drops, since bytecode is more compact than source plus engine overhead. |
| Memory use | Lower, which matters on low-end Android devices. |

Where it does not help much: heavy CPU work like large data crunching inside JS. With no JIT, a tight loop that runs for a long time can be slower than on a JIT engine. The fix is to move that work to native code, not to bend JS.

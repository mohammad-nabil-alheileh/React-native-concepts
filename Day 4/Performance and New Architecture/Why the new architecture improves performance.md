# Why the new architecture improves performance

The old architecture had three costs: serialization, asynchronous waiting, and loading everything up front. The new architecture removes or reduces each of them. Here are the improvements one by one.

1. **No JSON serialization (JSI):** In the old architecture every JS-to-native call was converted to JSON, sent, and parsed again. With JSI, JS calls native functions directly through C++ references, so the per-call overhead drops. This matters most for apps that make many calls, such as gestures, animations, and sensors.

2. **Synchronous communication when you need it:** The bridge was asynchronous only. In the new architecture, layout can be read synchronously.

3. **Faster startup (TurboModules):**
   - Old: every native module was initialized at launch, even ones you never used.
   - New: modules load lazily on first use.

4. **Better rendering (Fabric):**
   - Shared C++ shadow tree: JS and native read the same layout data instead of sending messages about it.
   - Rendering can happen on different threads, so urgent updates (like a touch) don’t have to wait behind heavy ones.

5. **Fewer runtime errors (Codegen):** Type mismatches between JS and native are caught at build time, not at runtime.

## Old vs new: where each gain comes from

| Problem in old architecture | New architecture fix | Result |
| --- | --- | --- |
| JSON serialization on every call | JSI direct calls | Lower call overhead |
| All modules loaded at startup | Lazy TurboModules | Faster cold start |
| One render priority | Concurrent rendering | Responsive input during heavy updates |
| Runtime type errors | Codegen | Fewer crashes, less validation |

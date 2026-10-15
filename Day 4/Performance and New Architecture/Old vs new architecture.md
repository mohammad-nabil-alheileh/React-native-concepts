# Old vs new architecture

## The old architecture (the Bridge)

JS and native talked through a single Bridge.

```text
JS thread  --[JSON message]-->  Bridge (async queue)  --[JSON message]-->  Native
```

**How it worked:**

- Every call from JS to native (and back) was serialized to JSON, put in a queue, sent asynchronously, then parsed on the other side.
- Messages were batched, so there was a delay between “JS asked” and “native did it”.
- All native modules were loaded at app startup, even ones you never used.
- Layout was calculated on the shadow thread and sent to the UI thread through the bridge.

**Example problem:** reading a scroll position or measuring a view from JS is never instant. JS asks, the message crosses the bridge, native answers, and the answer crosses back. By then the value may be stale.

## The new architecture

It replaces the bridge with direct communication, built from these parts:

### JSI (JavaScript Interface)

A C++ layer that lets JS hold direct references to native (C++) objects and call their methods, with no JSON serialization. Calls can be synchronous. Everything else in the new architecture is built on JSI.

### Fabric (the new renderer)

Replaces the old UI manager. The shadow tree is in C++ and shared between JS and native, so layout and view updates are no longer bridge messages. It supports synchronous layout reads and works with modern React features (Suspense, transitions, concurrent rendering).

### TurboModules (the new native modules)

- Lazy loaded: a module loads only when you first use it, which improves startup.
- Accessed through JSI, so calls are faster, and they can be sync or async.
- Typed through Codegen.

### Codegen

You write a typed spec, and at build time Codegen generates the C++ and native glue code. This gives type safety and catches mismatches between JS and native at build time instead of at runtime.

## Old vs new at a glance

| | Old | New |
| --- | --- | --- |
| Communication | Bridge (JSON, async, batched) | JSI (direct C++ references, sync or async) |
| Renderer | Old UI Manager | Fabric (shared C++ shadow tree) |
| Native modules | All loaded at startup | TurboModules, lazy loaded |
| Type safety | Manual, found at runtime | Codegen, found at build time |

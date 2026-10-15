# Memory leaks

JavaScript has a garbage collector (Hermes uses Hades). It frees objects that nothing can reach anymore. A memory leak happens when something you no longer need is still reachable, so the GC cannot free it.

## The common leaks

### 1. Timers that are never cleared

```js
// Leak: the interval keeps running after the screen is gone
useEffect(() => {
  setInterval(() => refresh(), 5000);
}, []);

// Fix: return a cleanup function
useEffect(() => {
  const id = setInterval(() => refresh(), 5000);
  return () => clearInterval(id);
}, []);
```

### 2. Event listeners that are never removed

```js
// Leak: a new listener is added every time the screen mounts, never removed
useEffect(() => {
  Dimensions.addEventListener('change', onChange);
}, []);

// Fix: modern RN APIs return a subscription, so remove it in cleanup
useEffect(() => {
  const sub = Dimensions.addEventListener('change', onChange);
  return () => sub.remove();
}, []);
```

### 3. Open subscriptions and connections

```js
useEffect(() => {
  const ws = new WebSocket(url);
  ws.onmessage = (e) => setMessages((m) => [...m, e.data]);
  return () => ws.close();           // without this, the socket lives on
}, [url]);
```

### 4. Async work that finishes after unmount

```js
useEffect(() => {
  const controller = new AbortController();

  fetch(url, { signal: controller.signal })
    .then(r => r.json())
    .then(setData)
    .catch(e => { if (e.name !== 'AbortError') console.warn(e); });

  return () => controller.abort();   // cancels the request on unmount
}, [url]);
```

### 5. Screens that stay mounted in navigation

```js
// Runs only while the screen is focused, and cleans up when it loses focus
useFocusEffect(
  useCallback(() => {
    const id = setInterval(poll, 3000);
    return () => clearInterval(id);
  }, [])
);
```

### 6. Things that grow forever

Global arrays, objects, and Maps, log buffers, and in-memory caches keep everything. Fix them with a size limit (for example an LRU cache), or clear them when no longer needed.

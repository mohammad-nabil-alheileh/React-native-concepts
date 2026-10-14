# AbortController

Cancellation means telling a request to stop. Timeout stops a request after a fixed time. Cancellation stops it when something in your app changes, such as the user leaving the screen or typing a new search.

AbortController is a built-in JavaScript object, available in React Native, that works with both `fetch` and Axios

## The basic API

```js
const controller = new AbortController();

api.get("/plants", { signal: controller.signal });

// Later, when you no longer need the result:
controller.abort();
```

- `controller.signal` is handed to the request
- `controller.abort()` cancels it
- One controller can cancel several requests that share the same signal
- A controller is single use: once aborted, you need a new one

## Detecting a cancelled request

An aborted request throws an error, so handle it.

```js
try {
  const { data } = await api.get("/plants", { signal: controller.signal });
  setPlants(data);
} catch (error) {
  if (axios.isCancel(error)) {
    return; // cancelled on purpose, ignore it
  }
  setError(parseError(error));
}
```

`axios.isCancel(error)` returns `true` for cancelled requests (the error code is `ERR_CANCELED`).

## Use case 1: leaving the screen

Without cancellation, a request keeps running after the user leaves the screen. When it finishes, it tries to update state of a component that is gone, which wastes data and battery.

```js
useEffect(() => {
  const controller = new AbortController();

  const load = async () => {
    try {
      setLoading(true);
      const { data } = await api.get("/plants", { signal: controller.signal });
      setPlants(data);
    } catch (error) {
      if (axios.isCancel(error)) return;
      setError(parseError(error));
    } finally {
      if (!controller.signal.aborted) setLoading(false);
    }
  };

  load();

  return () => controller.abort(); // runs when the component unmounts
}, []);
```

The `finally` check matters: after cancelling, you must not call `setLoading(false)` on a screen that is gone.

## Use case 2: search as you type

The user types “aloe”. Without cancellation, four requests fly out (“a”, “al”, “alo”, “aloe”). They can arrive out of order, so the results for “al” might land last and overwrite the correct results for “aloe”. Cancelling the previous request fixes that

```js
const [query, setQuery] = useState("");
const [results, setResults] = useState([]);

useEffect(() => {
  if (!query) {
    setResults([]);
    return;
  }

  const controller = new AbortController();

  api
    .get("/plants/search", { params: { q: query }, signal: controller.signal })
    .then(({ data }) => setResults(data))
    .catch((error) => {
      if (!axios.isCancel(error)) console.log(parseError(error).message);
    });

  return () => controller.abort(); // cancels the previous search
}, [query]);
```

Each time `query` changes, React runs the cleanup of the previous effect first, which cancels the old request.

## Use case 3: a manual “Cancel” button

For long operations like an upload:

```js
const controllerRef = useRef(null);

const upload = async () => {
  controllerRef.current = new AbortController();
  try {
    await api.post("/plants/photo", formData, {
      signal: controllerRef.current.signal,
      onUploadProgress: (e) => setProgress(e.loaded / e.total),
    });
  } catch (error) {
    if (axios.isCancel(error)) setProgress(0);
  }
};

const cancelUpload = () => controllerRef.current?.abort();
```

## Important limitation: the server may still run it

Cancelling only stops your side. The request may already have reached the server, and the server can still finish the work. Cancelling a `POST /orders` does not guarantee that no order was created. Cancellation is mainly for saving resources and avoiding stale UI, not for undoing actions.

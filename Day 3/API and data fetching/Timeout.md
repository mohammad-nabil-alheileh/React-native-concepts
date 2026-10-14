# Timeout

A timeout is a limit on how long you are willing to wait for a response. Without one, a request on a bad connection can hang for a very long time (or forever),

Axios has no timeout by default. The default is `0`, which means “wait forever”. You have to set one yourself.

## Setting a timeout in Axios

```js
// 1. Default for every request, set once on the instance
export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 10000, // 10 seconds, in milliseconds
});

// 2. Override for one specific request
await api.get("/plants", { timeout: 5000 });

// 3. A longer limit for slow operations like uploads
await api.post("/plants/photo", formData, { timeout: 60000 });
```

## What happens when it triggers

Axios aborts the request and throws an error. There is no response, so `error.response` is `undefined`.

## Timeout vs offline

Both fail without a server response, but they mean different things, so show different messages:

| Situation | What the user should see |
| --- | --- |
| Offline | “No internet connection” |
| Timeout | “The server is taking too long. Try again.” |

Mobile networks are slower and less stable than Wi-Fi, so don’t set it too aggressively. 2 or 3 seconds will cause false failures on a weak connection.

## fetch has no timeout option

If you ever use `fetch` instead of Axios, you have to build the timeout yourself with `AbortController`.

## A trap: retrying after a timeout

A timeout does not mean the server did nothing. For a `POST /orders`, the server may have created the order and only the response was slow. If you blindly retry, you can create duplicates.

- **GET** requests are safe to retry.
- **POST** requests need care.

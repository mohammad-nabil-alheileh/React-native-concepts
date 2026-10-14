# Retry failed requests

Some failures are temporary: a weak mobile signal, a server that is restarting, a rate limit that clears in a few seconds. Retrying means sending the same request again, automatically, before showing the user an error.

## Rule 1: retry only what can succeed later

| Failure | Retry? | Why |
| --- | --- | --- |
| No response (offline, network drop) | Yes | Often temporary |
| Timeout (`ECONNABORTED`) | Yes, with care | The server may just be slow |
| 500, 502, 503, 504 | Yes | Server trouble, often brief |
| 429 Too Many Requests | Yes, after waiting | Rate limit clears with time |
| 400, 404, 409, 422 | No | The same request fails again |
| 401 | Not here | Handled by token refresh (topic 9) |
| 403 | No | Permissions will not change |
| Cancelled (`axios.isCancel`) | No | You cancelled it on purpose |

## Rule 2: retry only requests that are safe to repeat

An operation is idempotent if running it twice has the same effect as running it once.

| Method | Safe to retry? |
| --- | --- |
| GET | Yes, it only reads |
| PUT, DELETE | Usually yes (setting or deleting twice leaves the same result) |
| POST | No by default. It may create a duplicate (two orders, two payments) |

## Rule 3: wait between attempts

Retrying instantly hammers a server that is already struggling. Wait longer after each failur

## Rule 4: always have a limit

Retry 2 or 3 times, then show the error. Endless retries drain battery and data, and leave the user watching a spinner.

## A simple helper (retry one call)

```js
const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

export async function withRetry(fn, { retries = 3, baseDelay = 1000 } = {}) {
  for (let attempt = 0; ; attempt++) {
    try {
      return await fn();
    } catch (error) {
      const canRetry = attempt < retries && shouldRetry(error);
      if (!canRetry) throw error;

      const backoff = baseDelay * 2 ** attempt;
      await sleep(backoff / 2 + Math.random() * (backoff / 2)); // jitter
    }
  }
}

// Usage
const plants = await withRetry(() => getPlants());
```

## Use a library: axios-retry

```js
import axiosRetry from "axios-retry";

axiosRetry(api, {
  retries: 3,
  retryDelay: axiosRetry.exponentialDelay,
  retryCondition: (error) =>
    axiosRetry.isNetworkOrIdempotentRequestError(error) ||
    error.response?.status === 429,
});
```

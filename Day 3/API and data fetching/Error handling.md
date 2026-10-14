# Error handling

Error handling is deciding what your app does when a request fails. failures come in three kinds. Good error handling sorts them and gives each a sensible result:

| Kind | How to detect | Example cause |
| --- | --- | --- |
| Server responded with an error | `error.response` exists | 400, 401, 404, 500 |
| No response came back | `error.request` exists, no `error.response` | offline, timeout, server down |
| Request never left | neither exists | bug in your config or code |

## Level 1: basic try/catch

```js
const loadPlants = async () => {
  try {
    const { data } = await api.get("/plants");
    return data;
  } catch (error) {
    console.log(error.message);
    throw error; // let the caller decide what to show
  }
};
```

## Level 2: turn Axios errors into one friendly shape

```js
// src/api/getErrorMessage.js
import axios from "axios";

export function parseError(error) {
  // Not an Axios error (a normal JS bug)
  if (!axios.isAxiosError(error)) {
    return { type: "unknown", message: "Something went wrong." };
  }

  // Server answered with 4xx or 5xx
  if (error.response) {
    const { status, data } = error.response;

    // Prefer the backend's own message if it sends one
    const serverMessage = data?.message || data?.detail;

    if (status === 401) return { type: "auth", status, message: "Please log in again." };
    if (status === 403) return { type: "forbidden", status, message: "You don't have permission." };
    if (status === 404) return { type: "notFound", status, message: "Not found." };
    if (status === 422 || status === 400)
      return { type: "validation", status, message: serverMessage || "Invalid data." };
    if (status >= 500) return { type: "server", status, message: "Server error. Try again later." };

    return { type: "http", status, message: serverMessage || "Request failed." };
  }

  // Timeout
  if (error.code === "ECONNABORTED") {
    return { type: "timeout", message: "The request took too long." };
  }

  // Request sent, no answer (offline or server unreachable)
  if (error.request) {
    return { type: "network", message: "No internet connection." };
  }

  return { type: "unknown", message: error.message };
}
```

## Level 3: handle some errors globally

Some errors are the same on every screen, such as 401 (go to login) or 500 (show a toast). Handling them inside every component means repeating code. You will move those into interceptors.

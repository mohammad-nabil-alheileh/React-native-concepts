# Interceptors

An interceptor is a function that Axios runs automatically on every request or response that goes through your instance. It lets you write shared logic once instead of repeating it in every API call.

| Type | When it runs | What to use it for |
| --- | --- | --- |
| Request interceptor | Before the request is sent | Modify the request (add headers, log it) |
| Response interceptor | After the response arrives, before your `await` receives it | Transform responses or handle errors globally |

## You attach them to the instance you created

```js
// src/api/client.js
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 10000,
});
```

## Request interceptor: attach the token automatically

Without interceptors, you would add the `Authorization` header in every call.

```js
api.interceptors.request.use(
  async (config) => {
    const token = await AsyncStorage.getItem("accessToken");
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config; // you MUST return config, or the request never goes out
  },
  (error) => Promise.reject(error)
);
```

The function receives `config` (the request settings), changes it, and must return it.

Now every call is authenticated with no extra code:

```js
const { data } = await api.get("/plants"); // token already attached
```

## Response interceptor: two handlers

`use()` takes two functions: one for success and one for errors.

```js
api.interceptors.response.use(
  (response) => response, // success: return it (or transform it)
  (error) => {
    // error: handle globally, then pass it on
    if (error.response?.status === 401) {
      console.log("Session expired");
    }
    return Promise.reject(error); // IMPORTANT: re-throw so callers still get the error
  }
);
```

1. **Unwrap the data**, so you write `res` instead of `res.data`

    ```js
    api.interceptors.response.use((response) => response.data);

    const plants = await api.get("/plants"); // already the body
    ```

2. **Send the app language on every request**

    ```js
    api.interceptors.request.use((config) => {
      config.headers["Accept-Language"] = currentLanguage; // "ar" or "en"
      return config;
    });
    ```

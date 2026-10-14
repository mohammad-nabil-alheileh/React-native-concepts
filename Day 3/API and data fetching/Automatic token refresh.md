# Automatic token refresh

the access token expires, the server answers 401, the app refreshes the token and retries. Automatic refresh means the app does all of that silently, so the user never sees an error or a login screen while their session is still valid.

## The flow

```text
GET /plants  →  401
      ↓
POST /auth/refresh  (with refresh token)  →  new access token
      ↓
save new tokens
      ↓
repeat GET /plants with the new token  →  200
      ↓
the caller's await returns data as if nothing happened
```

```js
api.interceptors.response.use(
  (res) => res,
  async (error) => {
    const original = error.config;

    if (error.response?.status === 401 && !original._retry) {
      original._retry = true; // mark it so we only retry once

      const refreshToken = await tokenStorage.getRefreshToken();
      const { data } = await axios.post(`${BASE_URL}/auth/refresh`, { refreshToken });
      await tokenStorage.save(data);

      original.headers.Authorization = `Bearer ${data.accessToken}`;
      return api(original); // repeat the original request
    }

    return Promise.reject(error);
  }
);
```

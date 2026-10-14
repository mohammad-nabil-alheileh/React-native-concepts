# Access and refresh tokens

Apps usually use two tokens instead of one. The access token is short-lived and goes with every request. The refresh token is long-lived and is only used to get a new access token.

## Access token vs refresh token

| | Access token | Refresh token |
| --- | --- | --- |
| Purpose | Sent with every API request to prove who you are | Used only to get a new access token |
| Lifetime | Short (5-15 minutes) | Long (days to weeks) |
| Sent to | Every API endpoint | Only the refresh endpoint |
| If stolen | Attacker has access until it expires, a few minutes | Attacker can keep generating new access tokens, which is much worse |

## The flow

1. **Login** (email + password) → server returns `accessToken` + `refreshToken`
2. **Call the API** with `Authorization: Bearer <accessToken>`
3. **Access token expires** → server answers `401`
4. **App sends `refreshToken` to `/auth/refresh`** → server returns a NEW `accessToken` (and often a new `refreshToken`)
5. **Retry the original request**
6. **Refresh token expires or is revoked** → user must log in again

## Where to store each one

| Token | Best place | Why |
| --- | --- | --- |
| Access token | In memory (a variable or state) | Short-lived, so losing it on app close is fine. Nothing is written to disk. |
| Refresh token | Secure storage (Keychain / Keystore) | Long-lived, must survive app restarts, so it must be encrypted at rest. |
| Either token | Never AsyncStorage | Unencrypted |

## Examples

### Keep the access token in memory

```javascript
let accessToken = null;

export const setAccessToken = (token) => { accessToken = token; };
export const getAccessToken = () => accessToken;
export const clearAccessToken = () => { accessToken = null; };
```

A plain module-level variable. It lives only while the JS runtime is running, so it is cleared automatically when the app closes. The three functions are the only way the rest of the app reads or writes it.

### Store the refresh token securely

```javascript
import * as Keychain from 'react-native-keychain';

await Keychain.setGenericPassword('refresh', refreshToken, { service: 'refresh-token' });

const creds = await Keychain.getGenericPassword({ service: 'refresh-token' });
const refreshToken = creds ? creds.password : null;
```

`react-native-keychain` saves the token in the iOS Keychain or Android Keystore, which are encrypted by the OS. The first argument (`'refresh'`) is a username and can be any string. The `service` name is the key used to find the item again. `getGenericPassword` returns `false` when nothing is stored, so `creds` is checked before reading `.password`.

### Login: save both

```javascript
async function login(email, password) {
  const { data } = await api.post('/auth/login', { email, password });
  setAccessToken(data.accessToken);                       // memory
  await Keychain.setGenericPassword('refresh', data.refreshToken, {
    service: 'refresh-token',                             // secure storage
  });
}
```

After a successful login, the access token goes into memory and the refresh token goes into the Keychain. The two are stored in different places because they have different lifetimes and risks.

### Auto-refresh with an Axios interceptor

```javascript
api.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (res) => res,
  async (error) => {
    const original = error.config;
    if (error.response?.status === 401 && !original._retry) {
      original._retry = true;                             // avoid infinite loops
      const creds = await Keychain.getGenericPassword({ service: 'refresh-token' });
      if (!creds) return Promise.reject(error);

      const { data } = await axios.post(`${BASE_URL}/auth/refresh`, {
        refreshToken: creds.password,
      });
      setAccessToken(data.accessToken);
      original.headers.Authorization = `Bearer ${data.accessToken}`;
      return api(original);                               // retry the original request
    }
    return Promise.reject(error);
  }
);
```

How this works:

- **Request interceptor:** runs before every request and adds the current access token to the `Authorization` header.
- **Response interceptor:** runs when a request fails. If the failure is a `401` and the request has not been retried yet, it refreshes the token and sends the original request again.
- **`_retry` flag:** marks the request so that if the refresh call also fails with `401`, the code does not loop forever.
- **`axios.post` instead of `api.post`:** the refresh call uses plain `axios`, so it does not go through these interceptors and cannot trigger itself.
- **`return api(original)`:** replays the failed request with the new token, so the caller gets the result as if nothing went wrong.

### App start: restore the session

```javascript
async function restoreSession() {
  const creds = await Keychain.getGenericPassword({ service: 'refresh-token' });
  if (!creds) return false;                               // go to the login screen
  const { data } = await axios.post(`${BASE_URL}/auth/refresh`, {
    refreshToken: creds.password,
  });
  setAccessToken(data.accessToken);
  return true;                                            // go to the home screen
}
```

Memory is empty on every app launch, so this function gets a new access token from the saved refresh token. It returns `true` if the user can go to the home screen and `false` if they need to log in.

### Logout: clear both

```javascript
async function logout() {
  clearAccessToken();
  await Keychain.resetGenericPassword({ service: 'refresh-token' });
}
```

Removes the access token from memory and deletes the refresh token from the Keychain. Both must be cleared, or the user would be logged back in on the next app start.

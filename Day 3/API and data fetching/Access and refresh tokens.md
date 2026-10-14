# Access and refresh tokens

After a user logs in, the server needs a way to recognize them on every later request. It doesn’t want to ask for the password each time, so it gives the app tokens. There are two kinds, and they do different jobs.

| | Access token | Refresh token |
| --- | --- | --- |
| Purpose | Sent with every API request to prove who you are | Used only to get a new access token |
| Lifetime | Short (about 5 to 30 minutes) | Long (days or weeks) |
| Sent to | Every endpoint | Only the refresh endpoint |
| If stolen | Attacker has limited time | More dangerous, so store it very carefully |

## Why two tokens instead of one?

If you had a single token that lasted 30 days and it leaked, the attacker would have access for 30 days. If you had a single token that lasted 10 minutes, the user would have to log in every 10 minutes.

Two tokens solve both problems:

- The access token is exposed constantly (sent on every request), so it expires quickly and a leak does little harm.
- The refresh token is exposed rarely (only to the refresh endpoint), so it can live long and keep the user logged in.

## What a token looks like

```text
eyJhbGciOi...  .  eyJzdWIiOiIxMjMiLCJleHAiOjE3...  .  SflKxwRJSM...
header              payload (claims)                  signature
```

The payload holds claims such as the user id and the expiry time.

Two important points:

- The payload is encoded, not encrypted. Anyone can read it, so never put secrets in them.
- The app can read a token but cannot trust it. Only the server can verify the signature.

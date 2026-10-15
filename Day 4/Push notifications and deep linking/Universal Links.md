# Universal Links

A normal `https://example.com/product/42` link opens your app on the right screen if it’s installed, and opens the website if it isn’t.

## How it works

There are two halves, and Apple checks that they match.

### Half 1: the website says “this app may open my links.”

Host a file called `apple-app-site-association` (AASA) at:

`https://example.com/.well-known/apple-app-site-association`

**Requirements:**

- No `.json` extension
- Served over HTTPS
- No redirects
- Content type `application/json`

```json
{
  "applinks": {
    "details": [
      {
        "appIDs": ["ABCDE12345.com.myapp"],
        "components": [
          { "/": "/admin/*", "exclude": true },
          { "/": "/product/*" }
        ]
      }
    ]
  }
}
```

- `ABCDE12345` is your Apple Team ID and `com.myapp` is your bundle ID.
- `components` lists which paths open the app. Rules are checked in order, so put `exclude` rules first.

### Half 2: the app says “I handle this domain.”

In Xcode, go to **Signing & Capabilities**, add **Associated Domains**, and add an entry:

```
applinks:example.com
```

## What happens on the device

- When the app is installed, iOS fetches the AASA file (through Apple’s CDN, not directly from your server) and caches it.
- When the user taps a matching link, iOS opens your app instead of Safari.
- Your JavaScript receives the URL through Linking API

## Android vs iOS at a glance

| | Android App Links | iOS Universal Links |
| --- | --- | --- |
| Server file | `assetlinks.json` | `apple-app-site-association` |
| App-side config | `intent-filter` with `autoVerify` | Associated Domains entitlement |
| Identifies app by | Package name + signing cert fingerprint | Team ID + bundle ID |
| Verified by | Android, at install time | Apple’s CDN, at install time |

# iOS (reading only): certificates, provisioning profiles, signing, entitlements, Archive and TestFlight

Android lets you install any signed APK. iOS only runs apps that Apple has approved in some way: installed on registered test devices, or distributed through TestFlight or the App Store.

| Piece | What it answers | Android equivalent |
| --- | --- | --- |
| Certificate | Who are you? (your or your team’s identity) | The keystore key |
| App ID (bundle ID) | Which app? (`com.app`, `com.app.dev`) | `applicationId` |
| Devices | Where may it run? (only for dev and ad hoc builds) | Nothing |

## 1. Certificates

A certificate proves your identity. It is created from a private key that lives in the Keychain of the Mac that requested it.

**Key facts:**

- Certificates expire after one year. An expired one breaks builds until you renew it
- The private key exists only on the Mac that created it. If that Mac is lost or wiped, you can’t sign with that certificate and need to create a new one.

## 2. Provisioning profiles

A provisioning profile is a file Apple generates that says: “this certificate may sign this App ID, to run on these devices, with these capabilities.” Xcode checks that your build matches it, and the phone checks it again at install time.

**Key facts:**

- Profiles also expire after a year, or sooner if the certificate inside is revoked
- A profile is tied to one App ID. With dev, stage, and prod bundle IDs

## 3. Entitlements

Entitlements are the special capabilities your app is allowed to use. Like Push notifications, Keychain sharing, iCloud.

## 4. Automatic vs manual signing

| | Automatic | Manual |
| --- | --- | --- |
| Who manages certs and profiles | Xcode, with your Apple account | You, by choosing the profile |
| Best for | Local development, small teams | strict control, many environments |
| Risk | Xcode may regenerate or revoke things unexpectedly | Easy to pick a wrong or expired profile |

## 5. Archive

An Archive is a Release build packaged for distribution.

## 6. TestFlight

TestFlight is Apple’s beta-testing system.

| | Internal testing | External testing |
| --- | --- | --- |
| Who | Members of your App Store Connect team (up to 100) | Anyone you invite by email or public link (up to 10,000) |
| Apple review | None | Required for the first build of each version (a lighter beta review) |
| Speed | Available once processing ends | Wait for review |

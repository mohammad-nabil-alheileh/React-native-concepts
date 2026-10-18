# Signing and keystore

Every Android app must be digitally signed before it can be installed or published. The signature proves two things:

- Who made the app
- The app wasn’t modified after you signed it

A keystore is the file that holds your private signing key.

## Key terms

| Term | Meaning |
| --- | --- |
| Keystore | The file that stores one or more keys |
| Alias | The name of one key inside the keystore |
| Store password | Protects the keystore file |
| Key password | Protects the individual key |

## Debug signing vs release signing

| | Debug | Release |
| --- | --- | --- |
| Keystore | Auto-generated | You create it |
| Password | Public: `android` | Secret, yours |
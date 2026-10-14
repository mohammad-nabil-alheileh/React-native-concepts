# Network state with NetInfo

`@react-native-community/netinfo` tells your app whether the phone is online or offline, and what kind of connection it has (Wi-Fi, cellular, and so on). It also notifies you when that changes.

NetInfo lets you know before, so you can:

- show an “You’re offline” banner
- skip requests that are certain to fail
- reload data automatically when the connection comes back

## The state object

```js
{
  type: "wifi",              // "wifi" | "cellular" | "none" | "unknown" | "ethernet" | "vpn" | ...
  isConnected: true,         // connected to a network?
  isInternetReachable: true, // can that network actually reach the internet?
  details: { ... },          // extra info that depends on the type
}
```

| Field | Meaning | Example where it differs |
| --- | --- | --- |
| `isConnected` | The phone is joined to a network | Wi-Fi connected, but the router has no internet |
| `isInternetReachable` | The internet really works | `false` on that same router |

Both can be `null`, which means “still checking” (this is common right after launch). Treat `null` as unknown, not as offline.

## 1. The hook: for UI

```jsx
export function OfflineBanner() {
  const { isConnected, isInternetReachable } = useNetInfo();
  const offline = isConnected === false || isInternetReachable === false;

  if (!offline) return null;

  return (
    <View style={{ backgroundColor: "#b00020", padding: 8 }}>
      <Text style={{ color: "white", textAlign: "center" }}>You're offline</Text>
    </View>
  );
}
```

## A one-time check: before an important action

```js
const save = async () => {
  const state = await NetInfo.fetch();
  if (state.isConnected === false) {
    Alert.alert("No connection", "Connect to the internet and try again.");
    return;
  }
  await createPlant(plant);
};
```

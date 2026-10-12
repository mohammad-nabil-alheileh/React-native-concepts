# Bottom Tabs

A bottom tab navigator shows a tab bar at the bottom of the screen. Each tab is its own screen (or its own stack of screens), and tapping a tab switches to it. Unlike a stack, tabs are **siblings**: switching doesn't push or pop anything, and each tab keeps its state when you leave it.

## 1. Common Options

| Option                                             | Purpose                                                                       |
| -------------------------------------------------- | ----------------------------------------------------------------------------- |
| `tabBarIcon`                                       | Function returning the icon component. Receives `{ focused, color, size }`    |
| `tabBarLabel`                                      | Text under the icon (or `tabBarShowLabel: false` to hide it)                  |
| `tabBarBadge`                                      | Small badge on the icon, such as a count                                      |
| `tabBarActiveTintColor` / `tabBarInactiveTintColor`| Colors for the selected and unselected tab                                    |
| `tabBarStyle`                                      | Style for the tab bar container                                               |
| `headerShown`                                      | Show or hide the header on tab screens                                        |
| `lazy`                                             | Render a tab only when first visited (default `true`)                         |
| `tabBarHideOnKeyboard`                             | Hide the bar when the keyboard opens (useful on Android)                      |

## 2. Tabs Plus Stacks

A tab usually contains its own stack, so you can open a details screen while keeping the tab bar visible.

```jsx
const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();

function HomeStack() {
    return (
        <Stack.Navigator>
            <Stack.Screen name="HomeMain" component={HomeScreen} />
            <Stack.Screen name="Details" component={DetailsScreen} />
        </Stack.Navigator>
    );
}

export default function App() {
    return (
        <NavigationContainer>
            <Tab.Navigator screenOptions={{ headerShown: false }}>
                <Tab.Screen name="Home" component={HomeStack} />
                <Tab.Screen name="Profile" component={ProfileScreen} />
            </Tab.Navigator>
        </NavigationContainer>
    );
}
```

- `headerShown: false` on the tabs avoids **two headers** on the Home tab: the stack inside it already shows its own header.
- The stack screen is named `HomeMain`, not `Home`, because the tab is already called `Home`. Different names make it clear which one you navigate to.

## 3. Tab State and Behavior

| Behavior                 | What happens                                                                                                                                         |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **State is preserved**   | Leave a tab and come back, and its scroll position and local state are still there, because the screens stay mounted after the first render.         |
| **Lazy by default**      | A tab isn't rendered until you first open it, which speeds up startup. Set `lazy: false` to render all tabs immediately.                            |
| **Re-tapping a tab**     | Tapping the active tab again pops its nested stack to the top. This happens automatically when the tab contains a stack.                            |
| **Back button**          | Tabs have no back history by default: going back from any tab returns to the **first** tab. Set `backBehavior: 'history'` on the navigator to go back to the previously visited tab instead. |

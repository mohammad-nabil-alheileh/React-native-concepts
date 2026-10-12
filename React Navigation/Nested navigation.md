# Nested Navigation

Nested navigation means rendering a navigator inside a screen of another navigator. The inner navigator is the `component` of a screen in the outer one.

**Use cases:**

- Each tab or drawer item needs its own history (a stack), so Details can open with a back button while the tab bar stays visible.
- You want some screens **above** the tabs, such as a modal or full-screen player, that hide the tab bar.
- You want to group flows (auth flow, onboarding, checkout) as their own navigator.

## 1. Example: Stack Inside Tabs Inside a Stack

```jsx
const RootStack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();
const HomeStack = createNativeStackNavigator();

function HomeStackScreen() {
    return (
        <HomeStack.Navigator>
            <HomeStack.Screen name="HomeMain" component={HomeScreen} />
            <HomeStack.Screen name="Details" component={DetailsScreen} />
        </HomeStack.Navigator>
    );
}

function MainTabs() {
    return (
        <Tab.Navigator screenOptions={{ headerShown: false }}>
            <Tab.Screen name="HomeTab" component={HomeStackScreen} />
            <Tab.Screen name="ProfileTab" component={ProfileScreen} />
        </Tab.Navigator>
    );
}

export default function App() {
    return (
        <NavigationContainer>
            <RootStack.Navigator screenOptions={{ headerShown: false }}>
                <RootStack.Screen name="Main" component={MainTabs} />
                <RootStack.Screen name="Settings" component={SettingsScreen} />
            </RootStack.Navigator>
        </NavigationContainer>
    );
}
```

The structure this creates:

```
NavigationContainer
└── RootStack
    ├── Main (MainTabs)
    │   ├── HomeTab (HomeStack)
    │   │   ├── HomeMain
    │   │   └── Details
    │   └── ProfileTab
    └── Settings          ← above the tabs, so the tab bar is hidden here
```

**Things to notice:**

- There is still **one** `NavigationContainer`, at the very top. Nested navigators never get their own.
- Each navigator is created once at module level, outside components.
- Headers are hidden on the parents (`headerShown: false`), so only the innermost stack draws one.
- Names are all different (`Main`, `HomeTab`, `HomeMain`), which avoids confusion.

## 2. How Navigation Actions Travel

Every navigator has its own navigation state, and `navigation` always refers to the **nearest** navigator. When you call an action:

1. The current navigator tries to handle it.
2. If it can't (the screen name isn't in it), the action **bubbles up** to the parent, then the grandparent, and so on.

**Example:** from `HomeMain`, calling `navigation.navigate('Settings')`:

| Step | Navigator  | Has `Settings`? | Result                         |
| ---- | ---------- | --------------- | ------------------------------ |
| 1    | HomeStack  | No              | Passes it up to the parent     |
| 2    | Tab        | No              | Passes it up to the parent     |
| 3    | RootStack  | Yes             | Opens Settings above the tabs  |

## 3. Navigating to a Screen Inside a Nested Navigator

To go to a screen deep inside another navigator, name the parent screen and use `screen` and `params` to point to the child:

```jsx
// from Settings, open Details inside the HomeTab stack, with an id
navigation.navigate('Main', {
    screen: 'HomeTab',
    params: {
        screen: 'Details',
        params: { id: 42 },
    },
});
```

Each level is `{ screen, params }`, and the last `params` is what `Details` receives in `route.params`.

## 4. Nested Navigator and Screen Lifecycle

- The inner navigator does not render until its parent screen renders. A lazy tab with a stack inside isn't created until first visited.
- Each navigator keeps its own state. Leaving a tab and returning restores the stack exactly where it was.
- Params you pass to the parent screen (`Main`) are not automatically visible to children. Pass them via `initialParams` or the nested `{ screen, params }` shape from section 3.

## 5. When to Nest, and When Not To

| Nest when                                   | Avoid                                                                                                         |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Each section needs its own back history     | Nesting the same type needlessly (a stack inside a stack gives double headers and confusing back behavior)    |
| Some screens must appear above the tabs     | Going deeper than 2 or 3 levels when a flatter structure works                                                |
|                                             | Duplicate names across navigators. React Navigation allows it, but `navigate('Details')` becomes ambiguous    |

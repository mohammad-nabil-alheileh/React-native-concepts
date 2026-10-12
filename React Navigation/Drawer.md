# Drawer

A drawer navigator gives you a side menu that slides in from the left edge. Each item in the menu opens a screen. It suits apps with many top-level sections (Home, Orders, Settings, Help, About).

## 1. Basic Example

```jsx
const Drawer = createDrawerNavigator();

function HomeScreen() {
    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Home</Text>
        </View>
    );
}

function SettingsScreen() {
    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Settings</Text>
        </View>
    );
}

export default function App() {
    return (
        <NavigationContainer>
            <Drawer.Navigator initialRouteName="Home">
                <Drawer.Screen name="Home" component={HomeScreen} />
                <Drawer.Screen name="Settings" component={SettingsScreen} />
            </Drawer.Navigator>
        </NavigationContainer>
    );
}
```

**What you get:**

- A header with a hamburger button on every screen
- Swipe from the left edge to open
- A menu listing each screen's name (or title)
- The Android back button closes the drawer first

## 2. Opening the Drawer from Code

The `navigation` object of a drawer screen has extra methods:

| Method                       | What it does                                   |
| ---------------------------- | ---------------------------------------------- |
| `navigation.openDrawer()`    | Opens the drawer                               |
| `navigation.closeDrawer()`   | Closes the drawer                              |
| `navigation.toggleDrawer()`  | Opens it if closed, closes it if open          |

**Example:** a custom button.

```jsx
function HomeScreen({ navigation }) {
    return <Button title="Menu" onPress={() => navigation.openDrawer()} />;
}
```

## 3. Customizing with Options

| Option                                             | Purpose                                                                                                                         |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `drawerType`                                       | `'front'` (overlays the screen), `'back'` (screen slides to reveal it), `'slide'` (both move), `'permanent'` (always visible, good for tablets) |
| `drawerPosition`                                   | `'left'` or `'right'`. For Arabic RTL layouts, test which side feels natural                                                    |
| `drawerStyle`                                      | Width, background color of the panel                                                                                            |
| `drawerLabel`                                      | Text shown in the menu                                                                                                          |
| `drawerIcon`                                       | Icon next to the label                                                                                                          |
| `drawerActiveTintColor` / `drawerInactiveTintColor`| Colors for the selected and unselected item                                                                                     |
| `swipeEnabled`                                     | Turn the swipe gesture on or off (useful on screens with horizontal scrolling)                                                  |
| `drawerItemStyle`                                  | Style for a single menu item                                                                                                    |
| `headerShown`                                      | Show or hide the header (and with it the hamburger button)                                                                      |

## 4. Custom Drawer Content

| Component                 | What it does                                                               |
| ------------------------- | -------------------------------------------------------------------------- |
| `DrawerContentScrollView` | Makes the menu scrollable                                                  |
| `DrawerItemList`          | Renders the registered screens as normal items                             |
| `DrawerItem`              | A single extra item that isn't a screen (logout, share, rate app)          |

```jsx
function CustomDrawerContent(props) {
    return (
        <DrawerContentScrollView {...props}>
            <DrawerItemList {...props} />
            <DrawerItem label="Logout" onPress={() => logout()} />
        </DrawerContentScrollView>
    );
}

<Drawer.Navigator drawerContent={(props) => <CustomDrawerContent {...props} />}>
    {/* screens */}
</Drawer.Navigator>
```

The `props` passed to `drawerContent` hold the navigation state, so `DrawerItemList` knows which screens to list and which one is active.

## 5. Drawer vs Bottom Tabs

|                        | Drawer                                       | Bottom tabs                    |
| ---------------------- | -------------------------------------------- | ------------------------------ |
| **Number of sections** | Many (5 to 15+)                              | Few (3 to 5)                   |
| **Visibility**         | Hidden until opened                          | Always visible                 |
| **Best for**           | Settings, secondary sections, admin panels   | Main frequently used sections  |

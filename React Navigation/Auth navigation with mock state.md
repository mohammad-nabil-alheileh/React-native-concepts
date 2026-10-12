# Auth Navigation with Mock State

Don't navigate to the Login or Home screen manually after signing in. Instead, **render different screens depending on the auth state**:

- **Logged out:** the navigator contains only Login and Register.
- **Logged in:** the navigator contains only Home, Profile, and so on.

## Why It's Better Than navigate('Home') After Login

|                       | Manual navigation                                          | Conditional screens                                  |
| --------------------- | ---------------------------------------------------------- | ---------------------------------------------------- |
| **Screens**           | Screens for both states exist at once                      | Protected screens don't exist while logged out       |
| **Back button/swipe** | Can return to Login after signing in                       | Impossible, since Login isn't registered anymore     |
| **History**           | You must remember to reset history on login and logout     | Handled automatically                                |
| **Deep links**        | Can open protected screens                                 | Protected screens are unreachable until signed in    |

## 1. The Auth State (Context)

```jsx
const AuthContext = createContext(null);

function AuthProvider({ children }) {
    const [user, setUser] = useState(null); // null = logged out

    const signIn = async (email, password) => {
        await new Promise(r => setTimeout(r, 500)); // fake network delay
        if (email === 'test@test.com' && password === '1234') {
            setUser({ name: 'Mohammad', email });
        } else {
            throw new Error('Wrong email or password');
        }
    };

    const signOut = () => setUser(null);

    return (
        <AuthContext.Provider value={{ user, signIn, signOut }}>
            {children}
        </AuthContext.Provider>
    );
}

const useAuth = () => useContext(AuthContext);
```

- `user` is the whole auth state: `null` means logged out, an object means logged in.
- `signIn` is a **mock**: it waits 500ms like a real request, then checks one hard-coded email and password. If they're wrong, it throws an error for the screen to show.
- `signOut` just sets `user` back to `null`.
- `useAuth()` is a shortcut so any component can read `user`, `signIn`, and `signOut` without passing props.

## 2. The Screens

Screens only change the state. They **never** call `navigate('Home')`.

### Login Screen

```jsx
function LoginScreen() {
    const { signIn } = useAuth();
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');

    const onLogin = async () => {
        try {
            setError('');
            await signIn(email, password);
        } catch (e) {
            setError(e.message);
        }
    };

    return (
        <View style={{ flex: 1, justifyContent: 'center', padding: 24 }}>
            <TextInput placeholder="Email" value={email} onChangeText={setEmail} autoCapitalize="none" />
            <TextInput placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
            {!!error && <Text style={{ color: 'red' }}>{error}</Text>}
            <Button title="Log in" onPress={onLogin} />
        </View>
    );
}
```

- `onLogin` calls `signIn`. On success, `user` changes and the navigator switches to Home by itself (section 3).
- On failure, the `catch` saves the error message, and it's shown in red.
- `!!error` turns the string into a boolean, so an empty string renders nothing instead of causing the "text must be inside `<Text>`" error.

### Home Screen

```jsx
function HomeScreen() {
    const { user, signOut } = useAuth();
    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Welcome, {user.name}</Text>
            <Button title="Log out" onPress={signOut} />
        </View>
    );
}
```

- `user.name` is safe to read here, because this screen only exists while `user` is set.
- **Log out** calls `signOut`, and the navigator switches back to Login.

## 3. The Navigator Renders Screens Based on the State

```jsx
const Stack = createNativeStackNavigator();

function RootNavigator() {
    const { user } = useAuth();

    return (
        <Stack.Navigator>
            {user ? (
                <Stack.Screen name="Home" component={HomeScreen} />
            ) : (
                <Stack.Screen name="Login" component={LoginScreen} options={{ headerShown: false }} />
            )}
        </Stack.Navigator>
    );
}
```

- This is the key part. When `user` is set, only Home is registered; when it's `null`, only Login is.
- When `user` changes, React Navigation sees that the screens changed and moves to the new screen automatically, with no `navigate` call.
- Because Login is removed after signing in, the back button can't return to it.

## 4. The Provider Goes Above the Navigator

```jsx
export default function App() {
    return (
        <AuthProvider>
            <NavigationContainer>
                <RootNavigator />
            </NavigationContainer>
        </AuthProvider>
    );
}
```

`AuthProvider` wraps everything, so both `RootNavigator` and all the screens can call `useAuth()`.

## How It Flows

| Action               | State change              | What the user sees            |
| -------------------- | ------------------------- | ----------------------------- |
| App starts           | `user` is `null`          | Login screen                  |
| Correct login        | `user` becomes an object  | Home screen (Login removed)   |
| Wrong login          | No change, error is set   | Login screen with red error   |
| Log out              | `user` becomes `null`     | Login screen (Home removed)   |

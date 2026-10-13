# Context API

Context lets a component provide a value to its entire subtree, and any descendant can read it directly without passing props through every level.

Three pieces:

- `createContext(defaultValue)` creates the context.
- `<Context.Provider value={...}>` supplies the value to everything inside it.
- `useContext(Context)` reads the nearest provider’s value.

```jsx
const ThemeContext = createContext('light');

<ThemeContext.Provider value="dark">
    <App />
</ThemeContext.Provider>

const theme = useContext(ThemeContext); // "dark"
```

Context is a delivery mechanism, not a state manager. The state itself still comes from `useState` or `useReducer`.

When the Provider’s `value` changes, every component that calls `useContext` for it re-renders, even if it only uses a small part of the value. `React.memo` does not block this.

## Example 1: theme context

```jsx
const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');

    const toggleTheme = useCallback(() => {
        setTheme(t => (t === 'light' ? 'dark' : 'light'));
    }, []);

    // Stable value, so consumers only re-render when theme actually changes
    const value = useMemo(() => ({ theme, toggleTheme }), [theme, toggleTheme]);

    return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

// Custom hook (Topic 6) with a safety check
export function useTheme() {
    const ctx = useContext(ThemeContext);
    if (!ctx) throw new Error('useTheme must be used inside <ThemeProvider>');
    return ctx;
}
```

Setup at the root:

```jsx
export default function App() {
    return (
        <ThemeProvider>
            <HomeScreen />
        </ThemeProvider>
    );
}
```

Use it anywhere below:

```jsx
function ThemeButton() {
    const { theme, toggleTheme } = useTheme();
    return <Button title={`Theme: ${theme}`} onPress={toggleTheme} />;
}

function Card({ children }) {
    const { theme } = useTheme();
    return (
        <View style={{ backgroundColor: theme === 'dark' ? '#111' : '#fff' }}>
            {children}
        </View>
    );
}
```

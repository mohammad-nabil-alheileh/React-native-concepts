# Global vs local state

Local state belongs to one component. Created with `useState` or `useReducer` inside the component.

Global state is shared by many components across the app, usually far apart in the tree. It lives outside any single component, in Context, or in a store like Zustand or Redux.

There is also a middle step, lifted state: when two nearby components need the same data, you move it up to their closest common parent and pass it down as props, Start local, and move up only when you must.

## Example 1: local state

```jsx
function PlantCard({ plant }) {
    const [expanded, setExpanded] = useState(false); // only this card cares

    return (
        <View>
            <Text onPress={() => setExpanded(e => !e)}>{plant.name}</Text>
            {expanded && <Text>{plant.description}</Text>}
        </View>
    );
}
```

Nobody else needs to know if this card is expanded, so it stays inside.

## Example 2: lifted state

```jsx
function SearchScreen() {
    const [query, setQuery] = useState(''); // lifted: two children need it

    return (
        <View>
            <SearchBar query={query} onChange={setQuery} />
            <ResultsList query={query} />
        </View>
    );
}
```

`SearchBar` writes it and `ResultsList` reads it, so the parent owns it.

## What belongs where

| Local | Global |
|---|---|
| Input text while typing | Logged-in user / auth token |
| Modal or dropdown open | Theme (dark/light) |
| Form field errors | Language and RTL setting |
| Selected tab in one screen | Shopping cart |
| Animation values | Notifications badge count |
| Toggle in one card | App-wide settings |

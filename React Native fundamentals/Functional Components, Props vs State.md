# Functional Components, Props vs State

## 1. Functional Components

A component is a JavaScript function that returns JSX. Modern React Native uses only functional components with hooks. Class components are legacy.

```jsx
function Greeting() {
    return (
        <View>
            <Text>Hello!</Text>
        </View>
    );
}

// Arrow function version (equally common)
const Greeting2 = () => <Text>Hello!</Text>;
```

**Rules:**

- The name must start with a capital letter (`Greeting`, not `greeting`).
- It must return a single root element (or a fragment `<>...</>`).
- You use it like a tag: `<Greeting />`.

## 2. Props

Props are inputs passed from a parent to a child. They work like function arguments and are **read-only**.

```jsx
function PlantCard({ name, waterEveryDays, onWater }) {
    return (
        <View>
            <Text>{name}</Text>
            <Text>Water every {waterEveryDays} days</Text>
            <Pressable onPress={onWater}>
                <Text>Water now</Text>
            </Pressable>
        </View>
    );
}

// Parent
<PlantCard
    name="Monstera"
    waterEveryDays={7}
    onWater={() => console.log('Watered!')}
/>
```

Props can be strings, numbers, booleans, objects, arrays, or functions (callbacks like `onWater`). Functions are how a child talks back to its parent.

Props can also have default values, set with destructuring:

```jsx
function PlantCard({ name, waterEveryDays = 7 }) { ... }
```

## 3. State

State is data owned by the component that can change over time. When state changes, the component re-renders.

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <View>
            <Text>Count: {count}</Text>
            <Pressable onPress={() => setCount(count + 1)}>
                <Text>Add</Text>
            </Pressable>
        </View>
    );
}
```

Updates are asynchronous and batched, so reading the value right after `setCount` gives the old value.

## 4. Props vs State

|                                     | Props                                 | State                    |
| ----------------------------------- | ------------------------------------- | ------------------------ |
| **Owned by**                        | Parent                                | The component itself     |
| **Can the component change it?**    | No (read-only)                        | Yes, via the setter      |
| **Triggers re-render when changed** | Yes (when the parent passes new ones) | Yes                      |
| **Purpose**                         | Configure the component               | Remember changing data   |
| **Web analogy**                     | HTML attributes                       | A form's current values  |

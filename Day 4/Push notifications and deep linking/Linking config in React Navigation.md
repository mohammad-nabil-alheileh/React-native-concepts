# Linking config in React Navigation

You give React nvigation a map from URL paths to screens, and it handles parsing, navigating, and building the back stack.

## Basic setup

Pass a `linking` object to `NavigationContainer`:

```js
import { NavigationContainer } from '@react-navigation/native';
import { ActivityIndicator } from 'react-native';

const linking = {
  prefixes: ['myapp://', 'https://example.com'],
  config: {
    screens: {
      Home: 'home',
      Product: 'product/:id',
      Settings: 'settings',
    },
  },
};

export default function App() {
  return (
    <NavigationContainer linking={linking} fallback={<ActivityIndicator />}>
      <RootStack />
    </NavigationContainer>
  );
}
```

| Option | What it does |
| --- | --- |
| `prefixes` | The URL beginnings your app accepts. Include your custom scheme and your https domain. React Navigation strips the prefix and matches the rest. |
| `config.screens` | Maps screen names to path patterns. |

So `myapp://product/42` opens the Product screen with `route.params.id === '42'`.

```js
function ProductScreen({ route }) {
  const { id } = route.params;   // '42' (a string)
  return <Text>Product {id}</Text>;
}
```

## Parsing params

URL params are always strings. Use `parse` to convert them, and `stringify` for the reverse direction

## Query strings

Query params are added to `route.params` automatically with no config needed

## Nested navigators

The config must mirror your navigator nesting.

## Back stack with `initialRouteName`

the stack has only one screen, so the back button has nowhere to go, Fix this by setting an initial route for that navigator

## Handling unknown URLs

Use a wildcard to catch anything that doesn’t match, so users don’t land on a blank screen

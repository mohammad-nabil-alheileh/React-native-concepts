# Typing Navigation with TypeScript

In plain JavaScript, nothing checks navigation calls. TypeScript can catch these at compile time: wrong screen names, missing or misspelled params, and wrong param types.

## 1. Define a Param List

One type that maps each screen name to its params:

```ts
export type RootStackParamList = {
    Home: undefined;                          // no params
    Details: { id: number; title?: string };  // id required, title optional
    Profile: { userId: string } | undefined;  // params optional
};
```

| Param type                    | Meaning                                          | Example screen |
| ----------------------------- | ------------------------------------------------ | -------------- |
| `undefined`                   | The screen takes no params                       | `Home`         |
| `{ ... }`                     | Params are required (fields with `?` are optional) | `Details`    |
| `{ ... } \| undefined`        | The whole params object is optional              | `Profile`      |

- Use a `type`, not an `interface`. Interfaces don't satisfy the constraint React Navigation expects, and you get confusing errors.
- Keys must match the `name` you register on each `Stack.Screen`.

## 2. Pass It to the Navigator

```tsx
const Stack = createNativeStackNavigator<RootStackParamList>();

<Stack.Navigator>
    <Stack.Screen name="Home" component={HomeScreen} />
    <Stack.Screen name="Details" component={DetailsScreen} />
</Stack.Navigator>
```

Now a misspelled name like `name="Detials"` is a compile error, and `Details` must be given a component compatible with its params.

## 3. Type the Screen Props

Each navigator package exports a `ScreenProps` helper that types both `navigation` and `route` at once:

| Navigator     | Helper type                  | Imported from                    |
| ------------- | ---------------------------- | -------------------------------- |
| Native stack  | `NativeStackScreenProps`     | `@react-navigation/native-stack` |
| Bottom tabs   | `BottomTabScreenProps`       | `@react-navigation/bottom-tabs`  |
| Drawer        | `DrawerScreenProps`          | `@react-navigation/drawer`       |

```tsx
type HomeProps = NativeStackScreenProps<RootStackParamList, 'Home'>;
type DetailsProps = NativeStackScreenProps<RootStackParamList, 'Details'>;

function HomeScreen({ navigation }: HomeProps) {
    return (
        <Button
            title="Open"
            onPress={() => navigation.navigate('Details', { id: 42 })}  // checked
        />
    );
}

function DetailsScreen({ route }: DetailsProps) {
    const { id, title } = route.params;   // id: number, title: string | undefined
    return <Text>{title ?? 'Untitled'} #{id}</Text>;
}
```

The helper takes two things: the param list, and the name of **this** screen. From those, TypeScript knows which screens `navigation` can go to and what is inside `route.params`.

## 4. What TypeScript Now Catches

```tsx
navigation.navigate('Details');                 // error: id is missing
navigation.navigate('Details', { id: '42' });   // error: string is not number
navigation.navigate('Detials', { id: 42 });     // error: unknown screen
navigation.navigate('Home', { id: 1 });         // error: Home takes no params
```

| Call                                          | Error                     |
| --------------------------------------------- | ------------------------- |
| `navigate('Details')`                         | `id` is missing           |
| `navigate('Details', { id: '42' })`           | `string` is not `number`  |
| `navigate('Detials', { id: 42 })`             | Unknown screen (typo)     |
| `navigate('Home', { id: 1 })`                 | `Home` takes no params    |

Because `Details` has required params, `route.params` is typed as non-undefined, so no guard is needed. For a screen with optional params (`Profile`), `route.params` can be `undefined`, and TypeScript forces you to handle it.

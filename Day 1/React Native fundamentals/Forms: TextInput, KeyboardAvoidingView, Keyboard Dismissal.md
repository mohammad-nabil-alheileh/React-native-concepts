# Forms: TextInput, KeyboardAvoidingView, Keyboard Dismissal

Forms are where React Native differs most from the web. There is no `<form>` tag and no Enter-to-submit. The on-screen keyboard also covers half the screen, and you have to manage it yourself.

## 1. Moving Focus Between Fields

Use refs and `onSubmitEditing` so the keyboard's action key jumps to the next field.

```jsx
function LoginForm() {
    const passwordRef = useRef(null);
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    return (
        <>
            <TextInput
                value={email}
                onChangeText={setEmail}
                keyboardType="email-address"
                autoCapitalize="none"
                returnKeyType="next"
                submitBehavior="submit"                  // keep the keyboard open when jumping to the next field
                onSubmitEditing={() => passwordRef.current?.focus()}
            />
            <TextInput
                ref={passwordRef}
                value={password}
                onChangeText={setPassword}
                secureTextEntry
                returnKeyType="done"
                onSubmitEditing={handleLogin}            // pressing "done" submits the form
            />
        </>
    );
}
```

## 2. The Keyboard Problem

When the keyboard opens, it can cover the input being typed into. React Native gives a few tools.

### KeyboardAvoidingView

It shrinks or shifts its content when the keyboard appears.

```jsx
function FormScreen() {
    return (
        <KeyboardAvoidingView
            style={{ flex: 1 }}
            behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
            keyboardVerticalOffset={Platform.OS === 'ios' ? 64 : 0}
        >
            <ScrollView
                contentContainerStyle={{ padding: 16, flexGrow: 1 }}
                keyboardShouldPersistTaps="handled"
            >
                {/* inputs */}
            </ScrollView>
        </KeyboardAvoidingView>
    );
}
```

**The `behavior` options:**

| Value      | What it does                                      | Typical use                          |
| ---------- | ------------------------------------------------- | ------------------------------------ |
| `padding`  | Adds bottom padding equal to the keyboard height  | iOS (most reliable)                  |
| `height`   | Reduces the view's height                         | Android, if needed                   |
| `position` | Moves the content up                              | Special cases, such as fixed layouts |

### keyboardVerticalOffset

Compensates for anything above the view (like a navigation header).

> **Platform note:** Android often handles the keyboard itself through `android:windowSoftInputMode="adjustResize"` in `AndroidManifest.xml` (the default in new projects). In that case you may not need `KeyboardAvoidingView` on Android at all.

## 3. Put the Form in a ScrollView

Long forms must be scrollable, otherwise the bottom fields can never be reached when the keyboard is open. Wrap the inputs in a `ScrollView`, inside the `KeyboardAvoidingView`.

## 4. Dismissing the Keyboard

The keyboard stays open until you close it. Users expect to close it by tapping outside, scrolling, or submitting.

### Keyboard.dismiss()

```jsx
const handleSubmit = () => {
    Keyboard.dismiss();   // close the keyboard first
    // ...submit
};
```

### Tap Outside to Dismiss

```jsx
<TouchableWithoutFeedback onPress={Keyboard.dismiss} accessible={false}>
    <View style={{ flex: 1 }}>
        {/* form */}
    </View>
</TouchableWithoutFeedback>
```

Do not use this trick if the content is inside a `ScrollView`, since it can block scrolling. Use the ScrollView props below instead.

### ScrollView Props

```jsx
<ScrollView
    keyboardShouldPersistTaps="handled"   // taps on buttons work while the keyboard is open
    keyboardDismissMode="on-drag"         // dragging the scroll view closes the keyboard
>
    {/* form */}
</ScrollView>
```

**`keyboardShouldPersistTaps` has three values:**

| Value               | Behavior                                                                                                      |
| ------------------- | ------------------------------------------------------------------------------------------------------------- |
| `never` (default)   | A tap outside the focused input only dismisses the keyboard, and the child doesn't receive the tap            |
| `always`            | The keyboard never dismisses on tap, and children always get the tap                                          |
| `handled`           | The keyboard stays open only if a child handles the tap (like a button), and it dismisses when tapping empty space |

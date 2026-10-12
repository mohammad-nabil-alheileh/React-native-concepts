# The Components That Build React Native Apps

## 1. View

The basic container, like `div`. Used for layout and grouping.

```jsx
<View>
    ...
</View>
```

## 2. Text

All text must be inside `<Text>`. Writing a string directly inside a `<View>` throws the error *"Text strings must be rendered within a `<Text>` component"*. You get a red error screen in development, and it can crash the app in production.

```jsx
<Text>Mohammad</Text>
```

Nested Text inherits styles from its parent Text only. A `<View>` does **not** pass styles (like `color` or `fontSize`) down to the Text inside it.

**Truncation props:**

- **`numberOfLines`**: the maximum number of lines to show; extra text is cut off.
- **`ellipsizeMode`**: where the `...` appears when text is cut off:
  - `head`: at the start → `...end of the text`
  - `middle`: in the middle → `start...end`
  - `tail`: at the end (default) → `start of the text...`
  - `clip`: no `...`, the text is just cut off at the edge

## 3. Image

Displays images in the app (`require(...)` is what imports the file).

```jsx
<Image source={require('./assets/logo.png')} />
```

Local images have their size detected automatically, but remote images need explicit dimensions (`width` and `height`).

**`resizeMode`** (how the image fits inside its box):

- `cover`: fills the whole box while keeping the aspect ratio; parts of the image may be cropped.
- `contain`: the whole image is visible while keeping the aspect ratio; empty space may appear.
- `stretch`: fills the box exactly, ignoring the aspect ratio, so the image may look distorted.
- `center`: centers the image without scaling it up; scales down only if it is bigger than the box.
- `repeat`: tiles the image to cover the box (like a pattern).

## 4. ScrollView

Renders all its children at once and makes them scrollable.

```jsx
<ScrollView>
    <Text>Item 1</Text>
    <Text>Item 2</Text>
    {/* ...more content */}
</ScrollView>
```

Everything is rendered up front, so it is bad for long lists because memory and performance suffer. For long lists use **`FlatList`**, which only renders the items visible on screen.

## 5. TextInput

Captures the user's input.

```jsx
<TextInput
    value={name}
    onChangeText={setName}
    placeholder="Your name"
    keyboardType="default"
    secureTextEntry={false}
/>
```

**Props:**

- `value`: the current text shown in the input (usually from state).
- `onChangeText`: called on every change and gives you the string directly (no `event.target.value`).
- `placeholder`: hint text shown when the input is empty.
- `keyboardType`: which keyboard opens (`default`, `numeric`, `email-address`, `phone-pad`, ...).
- `secureTextEntry`: hides the typed text (for passwords).
- `autoCapitalize`: controls automatic capital letters (`none`, `sentences`, `words`, `characters`).

Behavior differs slightly between iOS and Android. The keyboard can cover the input, which you handle with **`KeyboardAvoidingView`**.

## 6. Pressable

The way to handle touches.

```jsx
<Pressable
    onPress={() => console.log('pressed')}
    onLongPress={() => console.log('long press')}
>
    <Text>Press me</Text>
</Pressable>
```

**Events:**

- `onPress`: called on a normal tap (press and release).
- `onLongPress`: called when the user holds the press for a while (about 500ms by default).
- `onPressIn`: called the moment the finger touches the element.
- `onPressOut`: called the moment the finger is lifted.
- `onHoverIn` / `onHoverOut`: called when a mouse pointer enters / leaves the element (mainly web/desktop).

It works on any child. It has no default feedback, so you build it yourself, except on Android where you can add a native ripple effect with `android_ripple`.

## 7. Modal

A component that displays content on top of the current screen.

```jsx
<Modal
    visible={visible}
    animationType="slide"
    transparent
    onRequestClose={() => setVisible(false)}
>
    <View>
        <View>
            <Text>I'm a modal</Text>
            <Pressable onPress={() => setVisible(false)}>
                <Text>Close</Text>
            </Pressable>
        </View>
    </View>
</Modal>
```

It is native. **Props:**

- `visible`: shows the modal when `true`, hides it when `false`.
- `animationType`: how the modal appears:
  - `slide`: slides up from the bottom.
  - `fade`: fades in.
  - `none`: appears instantly with no animation.
- `transparent`: the modal has no background, so the screen behind it stays visible (you add the dark overlay yourself).
- `onRequestClose`: called when the user tries to close the modal, like pressing the Android back button.

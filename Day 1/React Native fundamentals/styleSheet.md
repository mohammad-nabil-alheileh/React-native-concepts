# StyleSheet

In React Native there is no CSS file. Styles are JavaScript objects, and `StyleSheet` is the helper for organizing them. Keep static styles in `StyleSheet.create`, and put only dynamic values inline.

## 1. The Basics

```jsx
function Card() {
    return (
        <View style={styles.card}>
            <Text style={styles.title}>Monstera</Text>
        </View>
    );
}

const styles = StyleSheet.create({
    card: {
        padding: 16,
        borderRadius: 12,
        backgroundColor: '#fff',
    },
    title: {
        fontSize: 20,
        fontWeight: '700',
        color: '#222',
    },
});
```

### Differences from Web CSS

| Web CSS                  | React Native                                                     |
| ------------------------ | ---------------------------------------------------------------- |
| `background-color`       | `backgroundColor` (camelCase)                                    |
| `padding: 16px`          | `padding: 16` (unitless, density-independent pixels)             |
| `width: 50%`             | `width: '50%'` (string)                                          |
| `font-weight: 700`       | `fontWeight: '700'` (string)                                     |
| `margin: 10px 20px`      | Not supported, use `marginVertical: 10, marginHorizontal: 20`    |
| `border: 1px solid red`  | `borderWidth: 1, borderColor: 'red'` (separate props)            |

There are no multi-value shorthands (like `margin: 10px 20px` or `border: 1px solid red`), no selectors, no media queries, and no pseudo-classes like `:hover`. Single-value shorthands such as `padding: 16` or `margin: 10` do work.

## 2. No Cascade

```jsx
// Does NOT work: color is not inherited by Text inside View
<View style={{ color: 'red' }}>
    <Text>Still black</Text>
</View>

// Works: nested Text inherits from parent Text
<Text style={{ color: 'red', fontSize: 18 }}>
    Red, <Text style={{ fontWeight: 'bold' }}>red and bold</Text>
</Text>
```

## 3. Combining Styles with Arrays

```jsx
<Text style={[styles.title, styles.error]}>Something went wrong</Text>

const styles = StyleSheet.create({
    title: { fontSize: 20, color: '#222' },
    error: { color: 'red' }, // overrides color
});
```

Later styles in the array override earlier ones.

## 4. Conditional Styles

```jsx
function Badge({ active }) {
    return (
        <View style={[styles.badge, active && styles.badgeActive]}>
            <Text style={styles.badgeText}>{active ? 'Active' : 'Inactive'}</Text>
        </View>
    );
}
```

`false`, `null`, and `undefined` in the array are ignored, so `condition && style` is safe.

## 5. Helpful Extras

- **`StyleSheet.absoluteFill`**: shortcut for `{ position: 'absolute', top: 0, left: 0, right: 0, bottom: 0 }`.
- **`StyleSheet.flatten`**: merges a style array into one object, useful when you need to read a value.
- **`StyleSheet.hairlineWidth`**: the thinnest line the device can draw, good for dividers.

## 6. Platform Differences to Know

Shadows are set differently on iOS and Android:

```jsx
card: {
    // iOS
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.2,
    shadowRadius: 4,

    // Android
    elevation: 4,
},
```

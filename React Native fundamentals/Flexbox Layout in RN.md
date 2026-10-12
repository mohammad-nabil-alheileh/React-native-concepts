# Flexbox Layout in React Native

Flexbox is the only layout system in React Native. There is no float, no grid, and no `display: block/inline`. Every `View` is already a flex container, so you never write `display: 'flex'`.

## 1. Differences from Web Flexbox

|                           | Web                            | React Native             |
| ------------------------- | ------------------------------ | ------------------------ |
| **Default flexDirection** | `row`                          | `column`                 |
| **Default alignItems**    | `stretch`                      | `stretch`                |
| **Default flexShrink**    | `1`                            | `0`                      |
| **flex shorthand**        | `flex: 1 1 0%` style strings   | A single number, `flex: 1` |
| **Enable it**             | `display: flex`                | Always on                |

## 2. The Two Axes

- **Main axis** is the direction children flow in (`flexDirection`). It is controlled by `justifyContent`.
- **Cross axis** is perpendicular to the main axis. It is controlled by `alignItems`.

In a `column` (the default), `justifyContent` works vertically and `alignItems` works horizontally. When you switch to `row`, the roles swap: `justifyContent` becomes horizontal and `alignItems` becomes vertical.

## 3. flexDirection

```jsx
flexDirection: 'column'          // default: top to bottom
flexDirection: 'row'             // left to right
flexDirection: 'row-reverse'     // right to left
flexDirection: 'column-reverse'  // bottom to top
```

## 4. justifyContent

```jsx
justifyContent: 'flex-start'    // default: pack at the start
justifyContent: 'center'        // center
justifyContent: 'flex-end'      // pack at the end
justifyContent: 'space-between' // first and last touch the edges, equal gaps between
justifyContent: 'space-around'  // equal space around each item
justifyContent: 'space-evenly'  // equal space between and at the edges
```

## 5. alignItems and alignSelf

```jsx
alignItems: 'stretch'    // default: fill the cross axis (if no fixed size)
alignItems: 'flex-start' // pack at the start
alignItems: 'center'     // center on the cross axis
alignItems: 'flex-end'   // pack at the end
alignItems: 'baseline'   // align text baselines
```

`alignSelf` overrides `alignItems` for one child:

```jsx
<View style={{ alignItems: 'flex-start' }}>
    <Text>Aligned left</Text>
    <Text style={{ alignSelf: 'flex-end' }}>I'm on the right</Text>
</View>
```

## 6. flex: Sharing the Space

`flex: 1` means "take all the remaining space". With several siblings, the space is split in proportion to their numbers.

```jsx
<View style={{ flex: 1 }}>             {/* fill the screen */}
    <View style={{ flex: 1 }} />       {/* 1 part */}
    <View style={{ flex: 2 }} />       {/* 2 parts */}
    <View style={{ height: 80 }} />    {/* fixed 80 */}
</View>
```

The fixed box takes 80, and the remaining space is divided 1:2 between the other two.

`flex: 1` only works if the parent has a size. A root container without `flex: 1` has no height to share.

```jsx
flexGrow: 1     // how much extra space to absorb
flexShrink: 1   // how much to shrink when space is short
flexBasis: 100  // starting size before growing or shrinking
```

## 7. flexWrap and gap

- `flexWrap: 'wrap'` moves items to the next line when they run out of room (the default is `nowrap`).
- `gap`, `rowGap`, and `columnGap` set the spacing between children.

## 8. Positioning

Every element is `position: 'relative'` by default, and `'absolute'` is the main other value.

- `absolute` positions relative to the parent, not the screen (there is no `fixed`).
- Use `zIndex` to control overlap.
- `top`, `left`, `right`, and `bottom` work as on the web.

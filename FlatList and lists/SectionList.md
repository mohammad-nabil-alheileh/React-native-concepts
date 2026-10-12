# SectionList

`SectionList` is React Native's virtualized list for **grouped data**: items organized under headers. It has the same performance benefits as `FlatList` (virtualization, pull-to-refresh, `onEndReached`), plus section headers and footers.

## 1. The Data Shape

Instead of a flat array, `sections` is an array of objects. Each has a `title` and a `data` array:

```jsx
const SECTIONS = [
    {
        title: 'Living room',
        data: [
            { id: '1', name: 'Monstera' },
            { id: '2', name: 'Fiddle leaf fig' },
        ],
    },
    {
        title: 'Kitchen',
        data: [
            { id: '3', name: 'Basil' },
            { id: '4', name: 'Mint' },
            { id: '5', name: 'Cactus' },
        ],
    },
];
```

- `data` is **required**, and the name must be exactly `data`. SectionList reads the items of each section from it.
- `title` is just a name we chose. You can add any other fields too (like `icon` or `count`), and they are available as `section.title`, `section.icon`, etc.

## 2. The Core Props

| Prop                          | Purpose                                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------------- |
| `sections`                    | The grouped data (array of `{ data: [...], ...anything }`)                                  |
| `renderItem`                  | Renders one item. Receives `{ item, index, section, separators }`                           |
| `renderSectionHeader`         | Renders a header. Receives `{ section }`                                                    |
| `renderSectionFooter`         | Optional footer for each section. Also receives `{ section }`                               |
| `keyExtractor`                | Unique string key per item: `(item, index) => string`                                       |
| `stickySectionHeadersEnabled` | Headers stick to the top while scrolling (default `true` on iOS, `false` on Android)        |
| `ItemSeparatorComponent`      | Separator between items inside a section                                                    |
| `SectionSeparatorComponent`   | Separator rendered at the top and bottom of each section                                    |

## 3. Example

```jsx
export default function PlantsByRoom() {
    return (
        <SectionList
            sections={SECTIONS}
            keyExtractor={(item) => item.id}
            renderItem={({ item }) => (
                <Text style={{ padding: 12 }}>{item.name}</Text>
            )}
            renderSectionHeader={({ section }) => (
                <Text style={{ fontWeight: 'bold', padding: 8, backgroundColor: '#eee' }}>
                    {section.title}
                </Text>
            )}
            renderSectionFooter={({ section }) => (
                <Text style={{ color: 'gray', padding: 8 }}>
                    {section.data.length} plants
                </Text>
            )}
            stickySectionHeadersEnabled
        />
    );
}
```

**How it works:**

- `sections={SECTIONS}` passes the grouped data. SectionList goes through each section, then through the `data` array inside it.
- `keyExtractor` uses each item's `id`. The ids must be unique **across all sections**, not just inside one section, because it's still one list.
- `renderItem` is called once per item. The `item` is one object from a section's `data` array, like `{ id: '1', name: 'Monstera' }`.
- `renderSectionHeader` is called once per section, **before** its items. `section` is the whole section object, so `section.title` gives "Living room" or "Kitchen".
- `renderSectionFooter` is called once per section, **after** its items. Here it uses `section.data.length` to show how many plants are in that room.
- `stickySectionHeadersEnabled` keeps the current section's header pinned at the top while you scroll through its items. It's set here so Android behaves the same as iOS.

The rendered order is:

```
Living room          ← header
  Monstera           ← item
  Fiddle leaf fig    ← item
2 plants             ← footer
Kitchen              ← header
  Basil
  Mint
  Cactus
3 plants             ← footer
```

## 4. Per-Section Rendering

Each section can override `renderItem`, `keyExtractor`, or `ItemSeparatorComponent`. This is useful when sections have different row types:

```jsx
const SECTIONS = [
    { title: 'Favorites', data: [...], renderItem: ({ item }) => <BigCard item={item} /> },
    { title: 'All plants', data: [...] }, // uses the default renderItem
];
```

The "Favorites" section has its own `renderItem`, so its items are shown as big cards. "All plants" doesn't have one, so it falls back to the `renderItem` prop passed to `<SectionList>`.

## 5. Grouping Flat Data into Sections

APIs usually return flat arrays, so you group them yourself:

```jsx
function groupByRoom(plants) {
    const map = {};
    plants.forEach((p) => {
        (map[p.room] ||= []).push(p);
    });
    return Object.entries(map).map(([title, data]) => ({ title, data }));
}
```

**How it works**, step by step:

1. `map` starts as an empty object. It will hold one array per room: `{ 'Kitchen': [...], 'Living room': [...] }`.
2. For each plant `p`, `map[p.room] ||= []` creates an empty array for that room **if it doesn't exist yet** (`||=` only assigns when the left side is empty/falsy). Then `.push(p)` adds the plant to its room's array.
3. `Object.entries(map)` turns the object into pairs: `[['Kitchen', [...]], ['Living room', [...]]]`.
4. `.map(([title, data]) => ({ title, data }))` turns each pair into the `{ title, data }` shape that SectionList expects.

**Example:**

```jsx
groupByRoom([
    { id: '1', name: 'Basil', room: 'Kitchen' },
    { id: '2', name: 'Monstera', room: 'Living room' },
    { id: '3', name: 'Mint', room: 'Kitchen' },
]);

// result:
// [
//     { title: 'Kitchen', data: [Basil, Mint] },
//     { title: 'Living room', data: [Monstera] },
// ]
```

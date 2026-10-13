# Localization

Localization (l10n) means making your app work in more than one language and region, in React Native it is controlled by a different system (`I18nManager`).

Three separate jobs are bundled in this topic:

- **Translation:** showing the right text for the user’s language (`i18next`).
- **Language switching:** changing the language at runtime and remembering the choice.
- **RTL layout:** mirroring the whole interface for right-to-left languages like Arabic.

## Packages

| Package | Job |
|---|---|
| `i18next` | The translation engine |
| `react-i18next` | The React binding (`useTranslation`) |
| `react-native-localize` | Reads the device language and region |
| `react-native-restart` | Restarts the app after a direction change |
| `@react-native-async-storage/async-storage` | Remembers the user’s chosen language |

## Example 1: translation files

```jsonc
// src/locales/en.json
{
    "home": {
        "title": "My Plants",
        "greeting": "Hello, {{name}}!",
        "water": "Water now"
    },
    "plants_count_one": "{{count}} plant",
    "plants_count_other": "{{count}} plants"
}

// src/locales/ar.json
{
    "home": {
        "title": "نباتاتي",
        "greeting": "مرحباً، {{name}}!",
        "water": "اسقِ الآن"
    },
    "plants_count_zero": "لا توجد نباتات",
    "plants_count_one": "نبتة واحدة",
    "plants_count_two": "نبتتان",
    "plants_count_few": "{{count}} نباتات",
    "plants_count_many": "{{count}} نبتة"
}
```

Keys are the same in every file, and only the values change.

## Example 2: initializing i18next

```js
// src/i18n.js
import 'intl-pluralrules';                 // must be imported first
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import * as RNLocalize from 'react-native-localize';
import AsyncStorage from '@react-native-async-storage/async-storage';
import en from './locales/en.json';
import ar from './locales/ar.json';

const SUPPORTED = ['en', 'ar'];

export async function initI18n() {
    const saved = await AsyncStorage.getItem('lang');

    // Best match between the device's language list and the languages we support
    const device = RNLocalize.findBestLanguageTag(SUPPORTED)?.languageTag ?? 'en';

    await i18n.use(initReactI18next).init({
        resources: {
            en: { translation: en },
            ar: { translation: ar },
        },
        lng: saved ?? device,        // the saved choice wins, then the device language
        fallbackLng: 'en',           // used when a key is missing
        interpolation: { escapeValue: false }, // React already escapes
    });
}

export default i18n;
```

## Example 3: using translations in components

```jsx
function HomeScreen({ user, plants }) {
    const { t } = useTranslation();

    return (
        <View>
            <Text>{t('home.title')}</Text>
            <Text>{t('home.greeting', { name: user.name })}</Text>
            <Text>{t('plants_count', { count: plants.length })}</Text>
            <Button title={t('home.water')} onPress={() => {}} />
        </View>
    );
}
```

- `t('home.title')` looks up the key in the current language.
- `{ name }` fills `{{name}}`. This is interpolation.
- `{ count }` picks the right plural form automatically.

When the language changes, every component that calls `useTranslation()` re-renders, It’s the same mechanism as Context.

## Example 4: never build sentences with string concatenation

```jsx
// Bad: word order is hardcoded
<Text>{t('hello') + ' ' + user.name + ', ' + t('youHave') + ' ' + count}</Text>

// Good: the whole sentence is one key, so each language controls its own word order
<Text>{t('home.summary', { name: user.name, count })}</Text>
```

## Example 5: language switching with RTL

```js
const RTL_LANGUAGES = ['ar', 'fa', 'ur'];

export async function setLanguage(lang) {
    const shouldBeRTL = RTL_LANGUAGES.includes(lang);

    await i18n.changeLanguage(lang);            // text updates immediately
    await AsyncStorage.setItem('lang', lang);   // remember the choice

    if (I18nManager.isRTL !== shouldBeRTL) {    // layout direction needs a restart
        I18nManager.allowRTL(shouldBeRTL);
        I18nManager.forceRTL(shouldBeRTL);
        RNRestart.restart();                      // restart so the new direction applies
    }
}
```

```jsx
function LanguageSwitcher() {
    return (
        <View>
            <Button title="English" onPress={() => setLanguage('en')} />
            <Button title="العربية" onPress={() => setLanguage('ar')} />
        </View>
    );
}
```

| | `i18n.changeLanguage` | `I18nManager.forceRTL` |
|---|---|---|
| Changes | Text | Layout direction |
| Takes effect | Immediately | After an app restart |
| Why | Just a React re-render | Direction is read by the native layout engine at startup |

## Example 6: RTL-safe styles

```js
// Bad: physical directions never flip
const styles = StyleSheet.create({
    card:  { marginLeft: 12, paddingRight: 8 },
    badge: { position: 'absolute', left: 0 },
    label: { textAlign: 'left' },
});

// Good: logical directions flip automatically
const styles = StyleSheet.create({
    card:  { marginStart: 12, paddingEnd: 8 },
    badge: { position: 'absolute', start: 0 },
    label: {},   // default alignment follows the text direction
});
```

| Physical (avoid) | Logical (use) |
|---|---|
| `marginLeft` / `marginRight` | `marginStart` / `marginEnd` |
| `paddingLeft` / `paddingRight` | `paddingStart` / `paddingEnd` |
| `left` / `right` | `start` / `end` |
| `borderLeftWidth` | `borderStartWidth` |
| `textAlign: 'left'` | leave unset (or `'auto'`) |

Icons that show direction (back arrows, chevrons, “next” buttons) must be mirrored manually:

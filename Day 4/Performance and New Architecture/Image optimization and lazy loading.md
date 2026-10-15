# Image optimization and lazy loading

Images are one of the biggest causes of slow screens, high memory use, and crashes in React Native apps. Two ideas fix most problems:

- **Image optimization:** make each image as small and cheap as possible for what you display.
- **Lazy loading:** load an image only when it is about to be needed.

## Part A: Image optimization

1. **Serve the right size.**

   Resize images on the server or CDN and request the size you need.
   Remember that phones have pixel density: a 100 dp avatar on a 3x screen needs roughly 300 px, not 1000.

2. **Choose the right format.**

   | Format | Best for | Notes |
   | --- | --- | --- |
   | JPEG | Photos | Small, no transparency |
   | PNG | Icons, graphics, transparency | Lossless, larger files |
   | WebP | Almost everything | Smaller than JPEG/PNG, supported on both platforms |
   | AVIF | Photos with the best compression | Even smaller, but support and decode cost vary by platform and version, so check before relying on it |
   | SVG | Simple icons and logos | Scales to any size, needs `react-native-svg` |

   For animated content, avoid large GIFs. Use animated WebP or a video instead.

3. **Always set dimensions for remote images.**

   React Native cannot know the size of a network image before it loads. Without `width` and `height`, it renders at 0 × 0. Setting dimensions also prevents layout jumps when the image arrives.

4. **Use caching.**

   The built-in `Image` has limited cache control. Dedicated libraries cache to memory and disk, so an image downloads once and then appears instantly.

5. **Android-specific help: `resizeMethod`.**

   On Android, the built-in `Image` has a `resizeMethod` prop, `resize` decodes the image at a smaller size instead of the full size, which can reduce memory use for large images.

## Part B: Lazy loading

Unlike the web, React Native has no `loading="lazy"` attribute. Lazy loading is achieved by controlling when the image component exists or renders.

1. **Virtualized lists do it for you.**

   `FlatList` and `FlashList` only mount rows near the screen. An image in a row starts loading only when the row is rendered, so an image far down the list loads when the user gets close.

2. **Do not render what is not visible.**

3. **Placeholders and progressive loading.**

   Show something light first, then swap in the full image.

4. **Prefetching (the opposite side of lazy loading).**

   When you know what is coming next, warm the cache ahead of time.

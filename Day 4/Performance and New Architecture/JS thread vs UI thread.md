# JS thread vs UI thread

A React Native app runs several threads at once. Two matter most:

| Thread | What it does |
| --- | --- |
| **JS thread** | Runs your JavaScript, including components, state updates, business logic, API response handling, and `onPress` handlers. |
| **UI thread** (main thread) | Runs the native side. It draws the real native views (`UIView` on iOS, `android.view.View` on Android) and handles touches and native scrolling. |

The target is 60 frames per second, so each frame has about 16.6 ms. If either thread takes longer than that, frames get dropped and the user sees lag or jank.

## How they work together

When you call `setState`, this happens:

1. The JS thread runs your component code and works out what changed.
2. It sends instructions to the native side (“create this view, change this text”).
3. The UI thread applies them and draws the pixels.

The two threads are separate, so one can be stuck while the other is fine.

# Debouncing and throttling

Some events fire very often: typing in a search box, scrolling, dragging, resizing, or sensor updates. If you run expensive work (an API call, a filter, a state update that re-renders a big tree) on every single event, the JS thread gets flooded and the app lags.

Debouncing and throttling both limit how often a function runs. They differ in when it runs.

| | Debounce | Throttle |
| --- | --- | --- |
| Rule | Run after the events stop for N ms | Run at most once every N ms while events continue |
| Analogy | Elevator door: it waits until nobody else is coming in, then closes | A tap that drips once per second no matter how hard you push |
| Fires during the burst? | No, only at the end | Yes, at regular intervals |
| Typical use | Search-as-you-type, form validation, autosave | Scroll position tracking, drag, resize, button spam |

## Debounce

If a new event arrives before the timer finishes, the timer resets. The function only runs once things go quiet.

## Throttle

The function runs immediately (or at the start), then ignores calls until N ms pass.

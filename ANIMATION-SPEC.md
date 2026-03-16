# Pull to Refresh - Bell Loader Animation Spec

## Overview

A pull-to-refresh interaction featuring a bell icon that fills with a liquid wave effect. Designed for the AJ Bell mobile app to provide visual feedback during manual content refresh.

**Live prototype:** https://pull-to-refresh-bell.netlify.app

---

## Animation Breakdown

### 1. Pull Phase (user is dragging)

| Property | Value |
|---|---|
| Trigger | Touch drag downward on content area |
| Bell entrance | Slides down from off-screen (`translateY(-70px)` to `translateY(0)`) |
| Bell entrance timing | `150ms ease-out` |
| Bell appears after | 10px of pull distance |
| Bell state during pull | Grey / unfilled (`#D3D3CA`) - no fill animation while finger is on screen |
| Content area movement | Translates down at 40% of pull distance |
| Max pull distance | 200px |
| Min pull to trigger refresh | 80px |

### 2. Fill Phase (after finger release)

| Property | Value |
|---|---|
| Trigger | User releases finger after pulling past 80px |
| Duration | **1200ms** |
| Easing | Ease-in-out quadratic (`t < 0.5 ? 2t^2 : 1 - (-2t+2)^2 / 2`) |
| Fill direction | Bottom to top |
| Fill colour | `#E81A41` |
| Fill technique | SVG path with dual sine waves clipped to bell shape via `clipPath` |
| Wave amplitude | Follows `sin(t * PI) * 1.8 + 0.3` - peaks at ~2.1px mid-animation, settles to 0.3px |
| Wave composition | Primary wave at 2x frequency + secondary wave at 3x frequency (35% amplitude of primary) |
| Wave speed | 0.065 radians per frame |
| Content area position | Held at `translateY(40px)` during fill (`300ms cubic-bezier(0.25, 0.46, 0.45, 0.94)` transition) |

### 3. Shake Phase (fill complete)

| Property | Value |
|---|---|
| Trigger | Fill animation reaches 100% |
| Duration | **500ms** |
| Type | CSS keyframe rotation (pendulum decay) |
| Easing | `ease-in-out` |
| Keyframes | `0%: 0deg` > `15%: 14deg` > `30%: -12deg` > `45%: 8deg` > `60%: -5deg` > `75%: 3deg` > `90%: -1deg` > `100%: 0deg` |

### 4. Skeleton Loading Phase

| Property | Value |
|---|---|
| Delay after shake | **400ms** |
| Bell exit | Slides back up off-screen (`translateY(-70px)`) via removing `.visible` class |
| Content area return | Springs back to `translateY(0)` in `400ms cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| Content swap | Default content hidden, skeleton shimmer shown |
| Shimmer cycle | **1400ms** per loop, infinite, `ease-in-out` |
| Shimmer gradient | `linear-gradient(90deg, #F0F0F0 25%, #E0E0E0 50%, #F0F0F0 75%)` at `200%` background-size |
| Skeleton visible for | **2000ms** |
| Content restore | Default content fades back in (opacity 0 to 1, `300ms ease`) |

---

## What Happens If the User Releases Early?

If the user releases their finger **before** reaching the 80px threshold:

- The content area springs back to its original position (`400ms cubic-bezier(0.25, 0.46, 0.45, 0.94)`)
- The bell slides back up off-screen
- **No fill animation plays**
- **No refresh is triggered**
- The interaction resets completely, ready for another attempt

---

## Bell Icon Specification

| Property | Value |
|---|---|
| SVG viewBox | `0 0 22 26` |
| Display size | 44px x 52px (2x scale) |
| Unfilled colour | `#D3D3CA` |
| Fill colour | `#E81A41` |
| Fill method | Animated SVG wave path clipped to bell shape using `clipPath` |

The bell SVG is the exact asset from the Figma design system file, containing the bell body and clapper as a single compound path.

---

## Complete Timeline

```
USER PULLS DOWN          RELEASES FINGER          FILL COMPLETE
     |                        |                        |
     |  Bell slides in (150ms)|  Liquid fills (1200ms) |  Shake (500ms)
     |  Content moves down    |  Content held at 40px  |
     |  No fill animation     |  Wave rises bottom>top |
     |                        |                        |
                                                       |
                                              400ms gap|
                                                       |
                                              SKELETON PHASE
                                                       |
                                              Bell exits (150ms)
                                              Content springs back (400ms)
                                              Skeleton shimmer (2000ms)
                                              Content fades in (300ms)
                                                       |
                                                     DONE

Total from finger release to content restored: ~4400ms
```

---

## CSS / Animation Tokens

```css
/* Timing tokens */
--ptr-bell-entrance: 150ms ease-out;
--ptr-fill-duration: 1200ms;
--ptr-fill-easing: ease-in-out (quadratic);
--ptr-shake-duration: 500ms;
--ptr-shake-easing: ease-in-out;
--ptr-post-shake-delay: 400ms;
--ptr-content-spring: 400ms cubic-bezier(0.25, 0.46, 0.45, 0.94);
--ptr-skeleton-shimmer-cycle: 1400ms ease-in-out infinite;
--ptr-skeleton-display: 2000ms;
--ptr-content-fade-in: 300ms ease;

/* Interaction thresholds */
--ptr-bell-appear-threshold: 10px;
--ptr-trigger-threshold: 80px;
--ptr-max-pull: 200px;
--ptr-content-pull-ratio: 0.4;
--ptr-content-hold-offset: 40px;

/* Colour tokens */
--ptr-bell-empty: #D3D3CA;
--ptr-bell-filled: #E81A41;
--ptr-background: #ECECEC;
--ptr-content-bg: #FFFFFF;
--ptr-skeleton-base: #F0F0F0;
--ptr-skeleton-highlight: #E0E0E0;
--ptr-handle: #D4D4D4;
```

---

## Usage Guidelines

- **When to use:** On screens that support manual content refresh (portfolio, transactions, notifications)
- **When not to use:** On static pages or settings screens where refresh has no meaning
- **Accessibility:** The animation is purely decorative. Screen readers should announce "Refreshing content" on trigger and "Content updated" on completion
- **Performance:** All animations use `transform` and `opacity` only (GPU-accelerated, no layout thrashing)
- **Touch handling:** Uses `touchstart`, `touchmove`, `touchend` events. Mouse fallback included for desktop testing

---

## Source Files

| Resource | Link |
|---|---|
| Live prototype | https://pull-to-refresh-bell.netlify.app |
| GitHub repo | https://github.com/alexanderuk82/pull-to-refresh |
| Figma designs | [Design System Mobile App v2.0](https://www.figma.com/design/8sTboPW0VLrUvP8yvGIGMG/manual-to-refresh?node-id=40003664-5716) |
| Jira ticket | [UXT-2124](https://ajbell.atlassian.net/browse/UXT-2124) |

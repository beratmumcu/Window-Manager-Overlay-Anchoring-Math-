# Android Overlay Position Anchoring Algorithm

A math-driven stabilization algorithm in Kotlin for Android `WindowManager` overlays. It anchors a specific child view (like a desktop screen pet or floating assistant widget) at a constant physical pixel coordinate on the screen while dynamically showing or hiding other views (like expanding speech bubbles or drop-down menus) inside a `WRAP_CONTENT` window.

## The Problem

When creating floating widgets in Android using `WindowManager.LayoutParams.WRAP_CONTENT`, the parent window automatically resizes whenever internal child views change visibility (e.g., from `GONE` to `VISIBLE`).

Because the window's top-left corner is bound to `params.x` and `params.y`, any change in view size causes:
1. **Horizontal Shifts:** Centered content shifts horizontally relative to the window bounds.
2. **Vertical Jumps:** Content below top-aligned headers is pushed down on the screen.
3. **Edge Collisions:** If the window is near the edge of the screen, expanding views will clip or push the main content completely off-screen.

## The Solution

This algorithm solves the shifting issue by capturing the absolute screen coordinates of the anchor view (the main character sprite) before changing visibilities, measuring the new bounds, and performing real-time coordinate translation:

1. **Window Re-positioning:** Move the parent window (`params.x`, `params.y`) to center around the target coordinates.
2. **Screen Boundary Clamping:** Safely clamp the window coordinates to keep the entire layout within screen limits.
3. **Translation Compensation:** Translate the child view inside the window (`translationX`, `translationY`) in the opposite direction to compensate for clamping or layout adjustments.

The visual result is a **seamless transition**—the character stays exactly where the user placed it, and speech bubbles/menus naturally expand outwards.

## Mathematical Model

Let:
- $X_{\text{screen}}$ and $Y_{\text{screen}}$ be the absolute pixel coordinates of the anchor view on the screen.
- $W_{\text{anchor}}$ and $H_{\text{anchor}}$ be the size of the anchor view.
- $W_{\text{new}}$ and $H_{\text{new}}$ be the newly measured dimensions of the parent window.
- $T_{\text{charTop}}$ be the height of all visible elements stacked above the anchor view.

### Horizontal Balancing:
$$\text{Target } X = X_{\text{screen}} + \frac{W_{\text{anchor}}}{2} - \frac{W_{\text{new}}}{2}$$
$$\text{Clamped } X = \text{coerceIn}(\text{Target } X, 0, \text{screenWidth} - W_{\text{new}})$$
$$\text{Translation } X = (X_{\text{screen}} - \text{Clamped } X) - \frac{W_{\text{new}} - W_{\text{anchor}}}{2}$$

### Vertical Balancing:
$$\text{Target } Y = Y_{\text{screen}} - T_{\text{charTop}}$$
$$\text{Clamped } Y = \text{coerceIn}(\text{Target } Y, 0, \text{screenHeight} - H_{\text{new}})$$
$$\text{Translation } Y = Y_{\text{screen}} - \text{Clamped } Y - T_{\text{charTop}}$$

## Implementation Code

See the [OverlayServiceSnippet.kt](OverlayServiceSnippet.kt) file for the clean Kotlin implementation of this algorithm.

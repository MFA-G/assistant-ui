---
"tw-shimmer": patch
---

fix: keep shimmer text readable below the browser floor

`shimmer` clipped the background to the text and set `-webkit-text-fill-color: transparent` unconditionally. Below the feature floor the gradient is invalid at computed value time and falls back to `none`, but the transparent fill stayed, so the label rendered as nothing.

Both declarations now sit behind `@supports (background-clip: text) and (color: oklch(from red l c h))`. Relative color syntax landed alongside `@property` and `tan()` in every engine, so it stands in for the gradient's requirements, and the unprefixed `background-clip: text` covers the clip itself. Failing either half also drops the gradient and the animation, so an element that happens to satisfy the gradient through explicit opt-ins (`shimmer-color`, `shimmer-angle-*`) cannot end up painting a band behind unclipped glyphs. Outside the gate the element keeps its normal fill and renders as plain static text, which is what the docs promise.

This gate only bounds the gradient path, and it does not move the package's support floor, which stays at Chrome 119+ / Firefox 128+ / Safari 16.4+. Chrome takes the `-webkit-mask-clip: text` compositor path instead, so unprefixed `background-clip: text` (Chrome 120) never applies to it; what sets Chrome's floor is the relative `oklch()` in `shimmer-bg`, which is 119. Firefox and Safari reach the new gate at the same versions the floor already names, because relative color syntax is the later of the two halves on both.

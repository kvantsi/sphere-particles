# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file WebGL/Canvas 2D animation: `sphere-particles.html`. Open it directly in a browser — no build step, no dependencies, no server needed.

```
open sphere-particles.html
```

## Git & GitHub

Remote: `https://github.com/kvantsi/sphere-particles`

Always commit **and push** after every change:
```
git add sphere-particles.html && git commit -m "..." && git push
```

## Architecture

Everything lives in one HTML file with an inline `<script>` block:

- **Particle data** — `thetas`, `phis`, `rnd` are pre-allocated `Float32Array`s (N=16000). `rnd` is fixed per-particle so particles don't flicker as they rotate.
- **Darkness formula** — the key visual driver. Two terms:
  - `edgeDark = sr^3.5` where `sr = sqrt(x²+y²)` is the screen-space radial distance (0=center, 1=silhouette). This creates the concentrated dark ring at the sphere edge.
  - `interiorDark = shadow * (1-sr)^1.5 * 0.65` — shadow-side shading in the interior.
- **Rendering** — Path2D bucket batching (8 buckets per hemisphere). All arcs for a given alpha level are batched into one `Path2D` and drawn with a single `ctx.fill()` call. Back-hemisphere paths are drawn first at low alpha (ghostly), then front-hemisphere at high alpha (solid black).
- **Rotation** — Y-axis only. `rotY` increments each frame; particle phi is offset by `rotY` at render time. The light direction is fixed in world space.

## Tuning parameters

| Variable | Effect |
|---|---|
| `N` | Total particle count (performance vs density) |
| `sr^3.5` exponent | Higher = sharper/thinner edge ring |
| `Math.pow(darkness, 0.55)` | Survival curve — lower exponent = more particles survive in bright areas |
| `size = 0.2 + darkness^0.85 * 9.0` | Dot size range |
| `light` direction | Moves the highlight/shadow |
| `rotY += 0.004` | Rotation speed |
| `R = Math.min(W,H) * 0.38` | Sphere radius as fraction of viewport |

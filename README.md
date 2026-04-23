
# OKLCH Gamut Mapping — Technical Deep Dive

👉 [Live Demo / Full Article](https://xcza.github.io/OKLCH-Gamut-Mapping-Chroma-Reduction-Binary-Search/)

---

## Overview

When working in perceptually uniform color spaces like **OKLab / OKLCH**, it's very easy to generate colors that fall **outside the displayable sRGB gamut**.

This project demonstrates:

- Why out-of-gamut colors happen
- Why naïve fixes (like RGB clipping) fail
- How **chroma reduction** solves the problem correctly
- A deep comparison of **four gamut mapping methods**

---

## Key Idea

Instead of distorting color channels (like clipping does), we:

> **Reduce chroma (C) while preserving lightness (L) and hue (H)**

In OKLCH, this means moving *radially inward* toward the neutral axis until the color fits inside sRGB.

---

## Features

- 🎨 Interactive OKLab / OKLCH visualization
- ⚡ High-performance **binary search chroma mapping**
- 🔬 Step-by-step worked example
- 📊 Side-by-side comparison of 4 methods:
  - Chroma Reduction (this project)
  - OKLCH + ΔEOK (CSS Color 4)
  - RGB Clipping
  - CIE LCH + ΔE2000
- 📈 Performance benchmarking
- 🧠 Deep math + annotated code

---

## The Algorithm (Chroma Reduction)

```js
function pickerMap(L, a, b) {
  if (L <= 0) return [0, 0, 0];
  if (L >= 1) return [1, 1, 1];

  const rgb = oklabToLinearSRGB(L, a, b);

  // Fast path
  if (inSRGBLinear(rgb)) {
    return rgb.map(linearToGamma);
  }

  const C = Math.sqrt(a * a + b * b);
  let H = Math.atan2(b, a) * 180 / Math.PI;
  if (H < 0) H += 360;

  const C_mapped = Math.min(C, maxChroma(L, H));
  return oklchToLinearSRGB(L, C_mapped, H).map(linearToGamma);
}
````

### Why it works

* Hue = angle → preserved
* Lightness = constant → preserved
* Only chroma is reduced → no distortion

---

## Comparison of Methods

| Method           | Speed     | Quality     | Hue Preserved |
| ---------------- | --------- | ----------- | ------------- |
| Clip             | ⭐ Fastest | ❌ Poor      | ❌ No          |
| Chroma Reduction | ⚡ Fast    | ✅ Excellent | ✅ Yes         |
| OKLCH + ΔEOK     | 🟡 Medium | ✅ Excellent | ✅ Yes         |
| CIE LCH + ΔE2000 | 🔴 Slow   | ✅ Excellent | ✅ Yes         |

---

## Performance

* **Clip:** ~4 µs per color
* **Chroma Reduction:** ~20–40 µs
* **OKLCH + ΔEOK:** ~75 µs
* **CIE LCH + ΔE2000:** ~230 µs

👉 Chroma Reduction gives **near-optimal visual quality** at a fraction of the cost.

---

## Math Summary

### OKLab → OKLCH

```
C = √(a² + b²)
H = atan2(b, a)
```

### Gamut Constraint

```
0 ≤ R, G, B ≤ 1   (in linear sRGB)
```

### Strategy

```
C_max = max chroma such that color is in gamut
C_final = min(C, C_max)
```

---

## Binary Search for Maximum Chroma

```js
function maxChroma(L, H) {
  let lo = 0, hi = 0.01;

  while (hi < 0.5 && inSRGBLinear(oklchToLinearSRGB(L, hi, H))) {
    hi *= 2;
  }

  for (let i = 0; i < 25; i++) {
    const mid = (lo + hi) / 2;
    if (inSRGBLinear(oklchToLinearSRGB(L, mid, H))) {
      lo = mid;
    } else {
      hi = mid;
    }
  }

  return lo;
}
```

---

## Why Not RGB Clipping?

Clipping changes channel ratios:

```
R = 1.42 → 1.0
G = 0.61 → 0.61
B = -0.08 → 0.0
```

➡️ This **shifts hue**, producing visibly incorrect colors.

---

## Running the Demo

### Option 1 — Open locally

```bash
open index.html
```

### Option 2 — Serve locally

```bash
npx serve
```

Then visit:

```
http://localhost:3000
```

---

## Use Cases

* Color pickers
* Design tools
* Image processing pipelines
* CSS color engines
* Rendering engines / shaders
* Scientific visualization

---

## Why OKLab / OKLCH?

* Perceptually uniform
* Predictable blending
* Better than RGB / HSL
* Ideal for modern color workflows (CSS Color 4)

---

## Credits

* OKLab by Björn Ottosson
* sRGB standard (IEC 61966-2-1)
* CSS Color 4 Working Draft

---

## License

CC0-1.0 license — use freely.

---

## TL;DR

> **If you care about color accuracy, never clip RGB.
> Reduce chroma in OKLCH instead.**


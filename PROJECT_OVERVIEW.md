# KindredCreations — Project Overview & Handoff

> Handoff doc for continuing this work from Claude Code on desktop.
> Owner: Kindred Creations (kindredcreations.studio) — a startup website-creation studio.
> This repo became an R&D playground for **browser-based, on-device hand-tracking
> gesture apps** — built to explore products the studio could sell.

---

## TL;DR

Four self-contained web apps that run **entirely in the browser on a phone**, using
the device camera for **real-time hand tracking** (Google MediaPipe) + **WebGL/3D**
(Three.js). No app install, no backend, no SDK keys. Everything is static and
deployed free via **GitHub Pages**.

| App | File | Live URL | Purpose | Status |
|-----|------|----------|---------|--------|
| **AIRGRAB** | `index.html` | [/](https://seniorcitizentx.github.io/KindredCreations/) | Pinch in the air to grab/fling floating 3D objects (the "wow" demo) | ✅ Works well |
| **Touchless Kiosk** | `kiosk.html` | [/kiosk.html](https://seniorcitizentx.github.io/KindredCreations/kiosk.html) | Gesture-controlled product showroom (point + pinch-to-select, open-palm-to-go-back) | ✅ Works well |
| **Virtual Try-On** | `tryon.html` | [/tryon.html](https://seniorcitizentx.github.io/KindredCreations/tryon.html) | Put a real 3D ring on your finger via the camera | ⚠️ Partial — see notes |
| **Gesture Reference Board** | `board.html` | [/board.html](https://seniorcitizentx.github.io/KindredCreations/board.html) | One-handed touchless pan/zoom of a reference image (for tattoo artists) | ✅ Working, final polish |

> ⚠️ Browsers cache aggressively. When testing an update, append `?v=N` to the URL
> (bump N) or use a private/incognito tab.

---

## The core capability (the actual asset)

> A **"no-app, no-hardware, runs-in-any-browser" interaction engine that reads a
> human hand in real time.** Just a URL + camera permission. Companies normally need
> native apps or special hardware for this; we deliver ~80% of the magic from a link.

This engine is the reusable thing. The four apps are demonstrations pointing it at
different problems (retail kiosks, jewelry try-on, artist tools).

---

## Tech stack

- **Hand tracking:** [`@mediapipe/tasks-vision`](https://developers.google.com/mediapipe) `HandLandmarker`
  (loaded from jsDelivr CDN). Model: `hand_landmarker.task` (float16) from Google's
  model CDN. 21 landmarks/hand, runs on-device at ~30fps. GPU delegate with CPU fallback.
- **3D / WebGL:** [three.js](https://threejs.org) `0.160.0` via import map + `three/addons/`.
- **Hosting:** GitHub Pages, **served from branch `claude/code-phone-capabilities-jpuuv8`,
  `/ (root)`** (Settings → Pages → Deploy from a branch). `.nojekyll` present.
- No build step, no npm install — everything is a single self-contained HTML file
  loading dependencies from CDNs.

### Common code patterns (shared across all four apps)
- `getUserMedia({ facingMode:'user' })` → front camera, mirrored via CSS `scaleX(-1)`.
- A `screen(landmark)` / `v3(landmark)` helper maps MediaPipe normalized coords →
  screen pixels using **object-fit: cover** math, with **x mirrored** (`1 - x`) to
  match the flipped video.
- Pinch detection: `dist(thumbTip[4], indexTip[8]) / dist(wrist[0], indexMCP[5])`
  (normalized by hand size so it's distance-invariant).
- A small camera "PIP" preview with a hand-skeleton overlay so users see tracking.

---

## App-by-app detail

### 1. `index.html` — AIRGRAB ✅
Floating neon 3D objects (Three.js). **Pinch** (thumb+index) to grab the nearest
object, move to drag, release to fling (keeps velocity). Two-handed (2 objects at
once). Live hand-skeleton overlay. Works well.

### 2. `kiosk.html` — Touchless Interactive Display ✅
**The most sellable concept.** A premium product showroom you control by gesture:
- **Point** (index fingertip) moves a cursor.
- **Pinch** to select a product (opens detail view).
- **Open palm** (hold ~0.6s) to go back.
- Products are a **swappable JS array** (`ITEMS`) → reskin per client (restaurant menu,
  real-estate listings, museum, car showroom). Business model: $5–25k custom
  installations + maintenance retainers.

### 3. `tryon.html` — Virtual Jewelry Try-On ⚠️ (hardest, partial)
Loads a **real Draco-compressed glTF ring model** (`models/ring.glb`) and anchors it
to the ring finger (landmarks 13/14) using MediaPipe + Three.js. Style chips re-tint
the metal. There's a `CFG` block at the top with all tuning knobs, and a **debug
skeleton overlay** (`DEBUG=true`) that draws landmarks + a magenta target circle.

**What we learned the hard way (important for continuing):**
- **Tracking & placement are correct** (confirmed via the debug overlay — green/blue
  dots land on the ring finger).
- The source `ring.glb` is actually a **tray of multiple pieces** (a `gold` band, a
  `silver` ring, loose gems). We isolate **only the `gold` mesh** on load.
- **Occlusion is the crux of "looks worn."** Without it, the ring looks like a flat 2D
  sticker on the hand. The current occluder (`OCCLUDE` flag in `CFG`) is a cylinder
  pinned to the ring's depth so the finger hides the back of the band — but this still
  needs visual verification/tuning. Earlier a mis-depthed occluder hid the *whole* ring.
- **Biggest open problem:** making it convincingly *worn* (occlusion + depth) without
  live iteration is slow. Recommended next step: tune occluder on desktop with a webcam,
  OR consider a purpose-built AR try-on SDK. **Rings are the hardest jewelry case** —
  earrings/necklaces (face tracking) and watches (wrist) are far easier wins.
- A nasty bug we hit & fixed: an element `id="dbg"` created a global `window.dbg` that
  collided with MediaPipe's internal `dbg` and crashed tracking. **Avoid element ids
  that shadow library globals.**

### 4. `board.html` — Gesture Reference Board ✅ (tattoo artists)
Touchless pan/zoom of a reference image. **Designed for ONE hand** (the artist holds
the tattoo machine in the other hand). Final tuning stage.
- **Pan:** pinch (thumb+index) + move. Tracks the **palm (landmark 9)**, not the
  fingertips, so opening the fingers to release doesn't "kick"/drag the image.
- **Zoom:** make a **fist**, move hand **closer to camera = zoom in, farther = out**
  (uses apparent hand size as a distance proxy).
- **Calibrated pinch thresholds (measured on the owner's hand/camera):**
  open hand `dI≈0.31`, firm pinch `dI≈0.10` → **engage pan at `dI<0.17`, release at
  `dI>0.27`** (hysteresis). These are real measured values, not guesses.
- Tappable `＋ / − / Reset` buttons and a `Load` button for the artist's own image.
- Has a **debug readout** (`#diag`, shows `dI=` + mode) still on screen — **remove it
  once the grab feel is confirmed.**

**Immediate next steps for board.html:**
1. Confirm pinch grab feels right; nudge `0.17 / 0.27` if needed.
2. Remove the `#diag` debug readout.
3. Add tattoo extras: **mirror/flip** (for stencils), **brightness/contrast** on the
   reference, a **"lock" gesture** so it can't move mid-tattoo, maybe a grid/ruler.

---

## Deployment / ops notes

- Repo is **public** (required for free GitHub Pages).
- Pages serves from branch **`claude/code-phone-capabilities-jpuuv8`** at root.
  (We removed an earlier GitHub Actions Pages workflow because it conflicted with the
  branch-based build — native branch deploy is simpler and what's in use now.)
- After a push, Pages rebuilds in ~1 min. Cache-bust with `?v=N` when testing.
- Everything is static; to run locally: `npx serve .` then open on the same machine
  (phones need the deployed https URL — camera APIs require https or localhost).

## Suggested priorities to continue
1. **Finish `board.html`** (closest to done) → a genuinely shippable tool.
2. **Productize `kiosk.html`** into a vertical-specific demo (pick one industry) → the
   clearest revenue path for the studio.
3. **Decide on try-on:** either invest in proper occlusion tuning with a webcam, pivot
   to easier jewelry (earrings/necklace via face tracking, watch via wrist), or use a
   dedicated AR SDK.

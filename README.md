# AIRGRAB — Gesture-Controlled Reality

Reach into your phone screen. AIRGRAB tracks your hand in real time and lets you
**pinch your fingers in the air to grab, move, and fling floating 3D objects** —
all running in the browser, no app to install.

![tech](https://img.shields.io/badge/hand%20tracking-on--device-27e7ff)
![tech](https://img.shields.io/badge/3D-WebGL%20%2F%20Three.js-ff2bd6)

## What makes it tick

- **On-device hand tracking** — Google MediaPipe `HandLandmarker` runs a neural net
  locally on your phone (21 joints per hand, up to 2 hands, ~30fps). No video ever
  leaves the device.
- **WebGL scene** — Three.js renders a field of glowing neon objects with a live
  hand-skeleton overlay composited over the camera feed.
- **Pinch physics** — thumb-to-index distance (normalised by hand size, so it works
  near or far) triggers a grab; release mid-motion and the object keeps your velocity
  and flings across space.

## How to use it

1. Open the link on your phone (see below) in **Chrome or Safari**.
2. Tap **Enter** and **allow camera access**.
3. Raise your hand to the front camera — you'll see a glowing skeleton lock on.
4. **Pinch your thumb and index finger together** to grab the nearest object, move
   your hand to drag it, and **open your fingers** to let go (flick to throw).
5. Use **both hands** to grab two objects at once.

> Requires a camera and an **HTTPS** connection (camera APIs are blocked on plain http).
> GitHub Pages serves over https automatically.

## Live link

Once the deploy workflow runs, the site is live at:

**https://seniorcitizentx.github.io/KindredCreations/**

(First deploy can take a minute or two. Check the **Actions** tab for status, or
**Settings → Pages** for the published URL.)

## Run it locally

It's a single self-contained `index.html`. Serve it over https/localhost, e.g.:

```bash
npx serve .
# then open the printed http://localhost:3000 on the same machine
```

(Phones need the deployed https link — localhost won't reach your phone.)

## Tech

| Piece | Library |
|-------|---------|
| Hand tracking | [@mediapipe/tasks-vision](https://developers.google.com/mediapipe) |
| 3D / WebGL | [three.js](https://threejs.org) |
| Hosting | GitHub Pages (auto-deploy via Actions) |

No build step, no dependencies to install — everything loads from CDNs.

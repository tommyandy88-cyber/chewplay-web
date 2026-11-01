# ChewPlay Web V0.1 — Inactivity watchdog regression
Date: 2026-11-01

## Preserved baseline
- Original V0.1 `index.html` Git blob SHA: `7386f6cb98b4a2d1bd88e21ffdfd0d915844d114`
- Detection algorithm `ChewDetector`, MediaPipe model URL, interface, video controls, defaults and KNN Flutter reference were not changed.
- Existing GitHub Pages website URL remains `https://tommyandy88-cyber.github.io/chewplay-web/`.

## Reported bug
Video continues playing while the user is not chewing or moving.

## Confirmed implementation gap
The original no-chew timeout was called only from `detectLoop()`, which was active only while camera tracking was ready. It could not stop playback while camera was off or model still loading; the inactivity clock was also initialized conditionally during tracking/calibration.

## Minimal repair
- Start inactivity time when media starts playing; reset it when switching sources.
- Add an independent 200ms watchdog that pauses media after the configured inactivity timeout (default: 4 s) whether or not the camera/face tracker is ready.
- Leave actual resume tied to a detected chew and existing browser / iOS autoplay restrictions.

## Local verification
- Node.js simulated control cases: 11/11 checks passed, including reproducing the original bug, YouTube mock pause/resume, local video pause, camera-off, model loading, no face, manual pause, and auto-control disabled.
- Local headless Chromium mobile-layout smoke: correct controls, local-video playback mock, and pause after ~4 s without camera.
- **Not yet accepted on an actual iPhone**. Safari, real YouTube IFrame API, front camera, and real Face Landmarker / false positives must be tested by the user.

## Deployment notes
Never rerun the same failing GitHub Pages workflow attempt repeatedly: a re-run may create duplicate `github-pages` artifacts. Use a fresh workflow run when necessary.

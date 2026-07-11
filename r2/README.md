# Round 2 — Permission State-Machine Harness

Primitive 2: one permission's grant/deny/pause silently mutating **another** permission's state, or a permission/stream **surviving a lifecycle event** it shouldn't. Scope: camera, microphone, geolocation.

Single page (`index.html`) + `helper.html` (bfcache staging). Host both on GitHub Pages (HTTPS required). Add to Home Screen to test PWA standalone mode.

## The oracle

Three pills at top: CAM / MIC / GEO. They turn **RED** when the resource is live. A bug is any time a pill goes RED — or `permissions.query` flips to `granted` — **after you denied/stopped it**, caused only by acting on a *different* permission or a lifecycle event. Screen-record + film the phone with a 2nd camera for the media pills.

## What to run

**Shape 1 — Pair matrix.** Six ordered pairs. For each: deny the first prompt, allow the second, watch if the denied pill lights up or re-grants. This is the generalized 31192 — Apple patched CAM↔MIC; CAM↔GEO and MIC↔GEO are unaudited.

**Shape 2 — Lifecycle survival.** Grant CAM/MIC, then: bfcache (nav to helper.html + Back), reload, background the tab (Home button 10s), or PWA relaunch. Bug = stream/permission outlives the context. The bfcache button auto-logs whether tracks survived `pageshow persisted=true`.

**Shape 3 — Prompt collision.** Fire two/three prompts together. Bug = answering one grants another, or a pill lights without an explicit allow.

## Tracker

| # | Shape | Cell | iOS | Repro? | Notes |
|---|-------|------|-----|--------|-------|
| 1 | Pair | deny CAM → allow MIC → CAM | | | patched path (control) |
| 2 | Pair | deny CAM → allow GEO → CAM | | | NEW |
| 3 | Pair | deny MIC → allow CAM → MIC | | | NEW |
| 4 | Pair | deny MIC → allow GEO → MIC | | | NEW |
| 5 | Pair | deny GEO → allow CAM → GEO | | | NEW |
| 6 | Pair | deny GEO → allow MIC → GEO | | | NEW |
| 7 | Life | bfcache nav + Back | | | ties to OE1964092112370 |
| 8 | Life | reload keeps stream | | | |
| 9 | Life | background tab 10s | | | watch status-bar dot |
| 10 | Life | PWA relaunch | | | standalone, no URL bar |
| 11 | Collide | CAM + MIC | | | |
| 12 | Collide | CAM + GEO | | | |
| 13 | Collide | CAM + MIC + GEO | | | |

## Hypotheses / where the bug likely hides

- The 31192 fix was almost certainly scoped to the camera/mic pair. **GEO shares the permission-prompt UI but a different backend** — a cross-flip through GEO may not honor a prior media deny.
- **bfcache** on iOS aggressively caches pages; a live `MediaStream` restored from bfcache is a known-messy area and directly extends your original teardown finding.
- **Collision**: iOS serializes prompts into a queue; a race in that queue could misattribute an allow. Low probability, high payout.

Fill the tracker, keep the copy-log output for any RED-after-deny event — that log + two-camera video is a submittable report.

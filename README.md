# Round 1 — Fix-Bypass Re-tests (iOS Safari / WebKit)

Goal: find **second-order bypasses** of your four patched CVEs. This is the fastest path to a new CVE — you already proved it with CVE-2025-43356 (bypass of 31192).

## Hosting (iOS-only workflow)

Permissions require a **secure context (HTTPS)**. Easiest free option: push this folder to a **GitHub Pages** repo (Settings → Pages → deploy from branch). You get `https://<user>.github.io/<repo>/`.

For the origin-spoof test (PoC 02) and picker test (PoC 03) you want **two different origins**. Use a second GitHub Pages repo, a different subdomain, or a cheap VPS with a real domain for the "domain B" target.

## How to observe (critical for iOS reports)

1. **Screen-record** on the iPhone (Control Center → Record) to capture URL bar, permission prompts, and the orange/green privacy dots in the status bar.
2. **Film the phone with a second phone** to prove the physical camera/mic is active independent of any software indicator. This two-angle proof is what sold CVE-2025-31192.
3. Each PoC has a **Copy log** button — paste the timestamped log straight into the report.

## The oracle for each PoC

| PoC | CVE | Bug confirmed if... |
|-----|-----|---------------------|
| 01 cam/mic state machine | 31192 / 43356 | Banner turns **RED (CAMERA LIVE)** after you **denied** camera in a prompt |
| 02 geo origin spoof | 30466 | Location prompt shows the **wrong origin** (domain B) or **blank**, not domain A |
| 03 picker overlay | 43228 | Page **navigates / popup opens** while the picker or camera is **still on screen** |

## Test tracker

Fill this in as you run each cell. `Repro?` = Y/N/partial.

| # | PoC | Test cell | iOS ver | Browser | Repro? | Notes |
|---|-----|-----------|---------|---------|--------|-------|
| 1 | 01 | Seq A: deny video → allow audio | | Safari | | classic 31192, should be patched |
| 2 | 01 | Seq B: re-click video after deny (43356) | | Safari | | should be patched |
| 3 | 01 | Seq C: deny video → allow motion → re-request video | | Safari | | NEW cross-permission flip |
| 4 | 01 | Seq C variant: deny video → allow geo → re-request | | Safari | | NEW |
| 5 | 01 | Seq D: deny plain video → env-facing video | | Safari | | NEW constraint flip |
| 6 | 01 | Seq D: deny plain video → exact-deviceId video | | Safari | | NEW constraint flip |
| 7 | 01 | video+audio combined after video-only deny | | Safari | | NEW |
| 8 | 02 | assign / href transition | | Safari | | original 30466 vector |
| 9 | 02 | window.open popup transition | | Safari | | NEW |
| 10 | 02 | anchor[download] transition | | Safari | | NEW |
| 11 | 02 | meta refresh transition | | Safari | | NEW |
| 12 | 02 | form submit cross-origin | | Safari | | NEW |
| 13 | 02 | iframe navigation | | Safari | | NEW |
| 14 | 02 | fullscreen enter + geo | | Safari | | NEW |
| 15 | 03 | file picker → top nav | | Chrome iOS | | original 43228 vector |
| 16 | 03 | camera picker → top nav | | Chrome iOS | | |
| 17 | 03 | file picker → popup | | Chrome iOS | | NEW |
| 18 | 03 | any picker in **standalone PWA** | | PWA | | NEW, no URL bar |
| 19 | 03 | Web Share sheet → nav | | Safari | | NEW overlay variant |

## Notes / hypotheses

- **PoC 01** — the fix likely special-cases the mic→camera path. Cross-permission flips (motion, geo) and constraint changes (facingMode, deviceId) may hit a different code path that still ignores the prior deny.
- **PoC 02** — the fix likely corrects origin display for the download-redirect case specifically. Other transition types that also defer navigation (popup, meta refresh, iframe) may still mis-attribute.
- **PoC 03** — third-party browsers can't close the picker themselves, so the WebKit-level fix is the only guard. Standalone PWA mode removes the URL bar entirely, raising severity of any residual navigation.

Keep every PoC minimal and free of "gift card" social-engineering wrappers — Apple triagers respond better to a clean primitive.

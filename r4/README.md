# Round 4 — PWA Scope Escape / Chrome-less Spoofing

Standalone (Home Screen) web apps run with **no URL bar**. The only guard against an installed "app" impersonating another site is: (a) does iOS show browser chrome when you navigate cross-origin or outside the manifest `scope`? If it doesn't, a page can draw its own fake URL bar and there's nothing to contradict it.

Pure UI-trust bug class — no payment backend, no permissions. Directly extends the "no origin signal" theme from your picker CVE (43228).

## Setup

1. Host this folder on GitHub Pages (e.g. `poc/r4/`). Needs `index.html` + `manifest.json` together.
2. On iPhone Safari open `https://<user>.github.io/poc/r4/`.
3. **Share → Add to Home Screen.** Note it installs as "Wallet" (spoofed app name via `apple-mobile-web-app-title` / manifest name).
4. **Open it from the Home Screen icon** — the banner must say STANDALONE. Testing in a Safari tab proves nothing.

## Oracle / tests

**Test 1 — cross-origin nav.** Set the URL field to a different origin and try each navigation method. Watch: does iOS pop a URL bar / origin banner, kick you to Safari, or silently navigate with no chrome? Silent cross-origin nav in standalone = the core bug.

**Test 2 — scope escape.** `scope` is `./` (this folder). The button navigates to `../` (parent, out of scope). Per spec, out-of-scope navigation should surface browser chrome. Does it?

**Test 3 — chrome-less spoof demo.** Paints a fake `apple.com 🔒` bar. In standalone there's no real bar to contradict it. This is the impact demo for a report: installed "Wallet" app showing a fake Apple URL.

## What makes this reportable

The strong version = **Test 1 or 2 shows NO chrome on cross-origin / out-of-scope navigation.** Then: user installs an innocent-looking app, it silently loads an attacker or lookalike origin full-screen with a painted fake URL bar → credential phishing with the OS's own trusted app frame. Screen-record the whole flow (install → open → navigate → fake bar).

If iOS *does* show chrome on every cross-origin nav, this is closed — note it and we pivot to app-scheme auto-invocation.

## Tracker

| # | Test | Method | iOS | Chrome shown? | Notes |
|---|------|--------|-----|---------------|-------|
| 1 | cross-origin | location.href | | | |
| 2 | cross-origin | location.replace | | | |
| 3 | cross-origin | anchor | | | |
| 4 | cross-origin | window.open | | | |
| 5 | cross-origin | meta refresh | | | |
| 6 | cross-origin | form submit | | | |
| 7 | out-of-scope | ../ | | | |
| 8 | fake chrome | paint bar | | | impact demo |

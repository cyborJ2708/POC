# Round 3 — Apple Pay Sheet Spoofing

New direction: point your origin-spoofing skill (30466, 43228) at the highest-value UI in iOS — the Apple Pay payment sheet. Bug classes: **origin misattribution** (sheet shows wrong/blank origin), **label spoofing** (attacker text on the OS-trusted sheet), **availability/card-state leak** (no-gesture privacy signal), **3rd-party-context reachability** (iframe can invoke payment it shouldn't).

## The setup reality (read first)

`session.begin()` shows the sheet, THEN fires `onvalidatemerchant`. So the sheet has a real display window even without a completed payment. Two tiers of testing:

**Tier 1 — free, no registration (start here).** These need no merchant ID:
- `ApplePaySession.canMakePayments()` — leaks that Apple Pay is available, no gesture. Test if it works inside a **cross-origin iframe** / 3rd-party context = a silent tracking/fingerprint signal.
- `canMakePaymentsWithActiveCard(merchantId)` — may leak whether the user has a **provisioned card**. Strong privacy angle if it returns anything useful without a real merchant.
- `new ApplePaySession(3, request)` + `begin()` — **empirically check whether the sheet visually flashes** on an unregistered domain before validation fails. If it does, you can observe origin + labels for free. This is the first thing to test.

**Tier 2 — needs Apple Developer Program ($99/yr).** To make the sheet fully render/persist you place a real association file at:
`/.well-known/apple-developer-merchantid-domain-association`
GitHub Pages serves `/.well-known/` fine (put the file at repo root → `poc/.well-known/...`). Only worth paying if Tier 1 shows the sheet flashes and origin handling looks weak.

## Files & hosting

- `applepay.html` — the probe. Host on **Domain B**.
- `wrapper.html` — Domain A page that iframes the probe. Edit its `src` to your Domain-B URL. Tests cross-origin sheet-origin attribution. Two domains needed here (unlike R2).
- Test both **with** and **without** `allow="payment"` on the iframe.

## Oracle / what to look for

1. Trigger the sheet from `wrapper.html` (Domain A embedding Domain B). Screen-record. **Which domain does the sheet show?** Top (A) is correct per spec. If it shows B, or blank, or the iframe can pay without `allow="payment"` → bug.
2. Deceptive labels: does "Free — verify identity" / "Apple Store" render verbatim on the trusted sheet next to a different real total? Label-vs-total mismatch on an OS sheet is a phishing surface.
3. `canMakePayments()` returning true inside a cross-origin iframe with no user gesture → privacy/tracking leak.
4. `sheet then redirect` / `after history push` — does the sheet stay up showing the OLD origin after the page navigates? (Directly parallels your 30466 transition trick.)

## Tracker

| # | Test | Context | iOS | Result | Notes |
|---|------|---------|-----|--------|-------|
| 1 | canMakePayments() | top-level | | | baseline |
| 2 | canMakePayments() | cross-origin iframe | | | leak if true, no gesture |
| 3 | canMakePaymentsWithActiveCard | top-level | | | card-presence leak |
| 4 | begin() sheet flash | unregistered domain | | | does sheet appear at all? |
| 5 | sheet origin | wrapper A embeds B, allow=payment | | | A vs B vs blank |
| 6 | sheet origin | wrapper A embeds B, NO allow | | | should be blocked |
| 7 | label spoof | deceptive label vs total | | | mismatch on trusted sheet |
| 8 | sheet + redirect | begin then location=apple.com | | | stale origin persists? |
| 9 | sheet + history push | begin then pushState | | | |

If test 4 shows the sheet flashing on an unregistered domain, tests 5–9 are all free. If not, decide whether the $99 account is worth it based on how the availability leaks (tests 1–3) behave.

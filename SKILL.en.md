---
name: store-screenshots
description: Standard pipeline for producing App Store / Play Store listing screenshots — staged real captures + code-rendered frames + double QA gate + store upload. Triggers on requests like "make store screenshots" or "make app listing images."
---

# store-screenshots — App Store Listing Image Pipeline

> Field-tested: refined over 7 rounds of a quality self-review loop (48 → 91 score) on a real shipping app.
> **Core principle: screenshot quality equals the quality of the state on the device screen.** The frame (headline + device mockup) is the easy 20%. The score is decided by what's actually on screen — data, scroll position, language, empty states.

## ⛔ 4 Hard Bans (all derived from real regressions)
1. **No pixel post-processing.** No fades, crops, pillarbox compositing, debanding. Every fix in this direction creates a new seam or hard cut — a whack-a-mole structure that caused a 91 → 78 score regression. Fix clipping/whitespace by **staging the capture itself** (scroll position, row count, data), not by editing the pixels afterward.
2. **No compositing phone screenshots into a tablet canvas.** The moment you fit a phone screenshot into a tablet frame, you get pillarbox seams and density collapse. **Capture directly from a tablet simulator/emulator.** If the app is universal, the store requires tablet screenshots anyway, and capturing for real surfaces tablet-layout bugs early.
3. **No false claims.** Don't depict unshipped features, promise copy the screen doesn't match, or highlight numbers that are logically inconsistent.
4. **Don't remove debug banners with pixel edits.** Disable them at the framework level instead (e.g. Flutter's `debugShowCheckedModeBanner: false`).

## Procedure (7 steps)

### 0. Preconditions
Confirm a visual-token doc (colors, type, sizes) and copy/ASO doc exist, boot the latest build on both phone and tablet simulators, and confirm debug banners are disabled in code.

### 1. Lock the copy (after approval, freeze the wording)
- 6–8 screens, one headline (≤2 lines) + one subline per screen. **Screens 1–3 decide the outcome** (store-scroll behavior).
- One emphasized/colored word per screen max. Whatever the copy promises, that screen's actual UI must show it (e.g. if the copy says "searchable," that screen must show a real search state).

### 2. Design the staging — "zero embarrassing state" checklist
Lay out a table of {screen, data state, scroll position, search term} per slide. Pass criteria:
- [ ] No empty/sparse data (no "owns 1 item," "0 results," "collecting 0%" states)
- [ ] No **standalone** text in a non-target language (secondary/supporting labels are fine)
- [ ] Viewport completeness: no half-cut rows or label-less slivers — pick a row count/search term that ends naturally (changing a search term beats post-processing)
- [ ] Numeric consistency (price, ratio, P&L are staged so they don't contradict each other)
- [ ] Zero contact with production servers/DB — staging hits a local simulator DB only, block real-server sync in capture mode

### 3. Capture — integration-test staging harness
- Automate DB seeding + navigation + screenshot capture with an integration test (e.g. Flutter's `integration_test`) so it's reproducible for the next version.
- **Full-screen capture (including tab bars/chrome)** — this is the store-standard convention. Run separately for phone and tablet simulators.
- Example sizes: iPhone 6.9" 1290×2796 / iPad 13" 2064×2752, resampled to 2048×2732 — always re-check the platform's current accepted dimensions before relying on these numbers.

### 4. Render the frame — one code path, no variants
- Render in code (e.g. Python PIL): background (a consistent light tone works well) + headline/subline + device mockup, with the screenshot going through a **single** `resize → paste` path (don't grow multiple ad-hoc transform functions).
- Third-party screenshot-framing tools often can't do fine copy control like inline word emphasis — if that's blocking, a small in-house render pipeline is the right call.

### 5. Double QA gate (visual pass ≠ score pass)
- **Gate 1 — visual inspection** (vision model / design reviewer): check for overlap, clipping, tofu/broken glyphs, seams, watermarks, blank tiles.
- **Gate 2 — scored review** (anchored rubric, capped at 2 rounds): reviewer persona = "a target user scrolling the store, deciding to download in 3 seconds." Example 5-axis rubric = message-to-screen match 30 / target-user resonance 25 / visual polish 20 / thumbnail (10%-scale) legibility 15 / accuracy 10.
- **Explicitly write into the rubric** (to prevent reviewer variance): "Natural clipping of a horizontally scrolling chip row, a list continuing past the viewport, and a standard modal dim overlay are normal UI conventions, not defects. A defect is something that reads as a rendering bug, a compositing artifact, or unfinished work."
- If the score drops across rounds, post-processing has crept back in — check hard-ban #1 first.

### 6. Upload to the store (API)
- App Store: use the App Store Connect API to find/create the screenshot set → clear existing assets → reserve → chunked upload via presigned URL → commit checksum → verify every asset reaches COMPLETE state → reorder.
- Play Store: apply the same principles via the Play Developer API's image upload endpoint.
- Never print API keys or secrets to logs.

### 7. Reusable implementation (copy-paste starting point — adapt paths per project)
- Staging harness: in the app repo's integration-test directory, a `store_stage`-style file (seeding/navigation logic) plus a `store_capture_test`-style file (capture mode: screen overlay rendering, blocking real-server sync).
- Renderer: a frame-assembly script (zero post-processing — background + text + mockup compositing only).
- Store-upload helper: JWT signing (App Store Connect uses ES256) + screenshot-set API calls. Keep signing keys in a secure local path — never commit them to the repo.

## Side effects (expect and harvest them)
The staging walkthrough is a free end-to-end test. Running this loop on a real app surfaced real product bugs — an exposed debug banner, a tablet layout that wasn't adapted, a crash on a specific screen, UI clipping, and unpromoted seed-data versions. Bugs found during capture should be **fixed in the product and recaptured**, not hidden in the screenshot.

## Exit criteria
Score ≥95 (or a final call after 2 rounds) + every store asset verified COMPLETE. If you ship below that bar, record exactly what's still missing and why (screenshots are an asset you can swap out in the next version submission — recognize the point where perfectionism hits steeply diminishing returns).

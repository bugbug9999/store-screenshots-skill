# App Store & Google Play Screenshot Generator — Claude Code Skill for ASO Listing Images

**store-screenshots** is a [Claude Code](https://claude.com/claude-code) skill that automates the full pipeline for producing **App Store Connect** and **Google Play Console** screenshots (app listing images, marketing screenshots, ASO creatives): staged real device captures → code-rendered frames with headline copy → a double QA scoring gate → automated upload via the App Store Connect API / Play Developer API.

If you've ever hand-cropped screenshots in Figma or Photoshop for an app store submission, tried `fastlane frameit` and hit its inline-text-emphasis limits, or gotten a **rejection for misleading screenshots / inconsistent tablet layout**, this skill exists to replace that workflow with a reproducible, code-driven one.

**Keywords:** app store screenshots, google play screenshots, ASO, app store optimization, screenshot generator, app store connect api, play console api, mobile app marketing, listing images, app store submission, app rejection, fastlane frameit alternative, claude code skill, AI agent skill.

Field-tested and refined over multiple quality-review rounds (score 48 → 91) on a real shipping iOS/Android app.

## What it does

- **Stages real app states** for capture instead of faking them in post — no empty data, no half-cut lists, no wrong-language text leaking into a screenshot.
- **Captures screenshots via integration tests** (device simulators/emulators, phone + tablet) so the process is reproducible for every app version.
- **Renders frames in code** (headline, subline, device mockup) with zero pixel post-processing — no fades, crops, pillarboxing, or debanding that create seams.
- **Runs a two-gate QA check**: a visual-defect pass (overlap, clipping, broken glyphs, watermarks) plus a scored review against a rubric anchored to "would a user scrolling the store download this in 3 seconds?"
- **Uploads directly to the store** via API (App Store Connect screenshot sets / Play Developer API image endpoints), verifying every asset reaches a committed state.

## Files

- 🇺🇸 [`SKILL.en.md`](./SKILL.en.md) — English version
- 🇰🇷 [`SKILL.ko.md`](./SKILL.ko.md) — Korean version (original)

## Usage

Copy `SKILL.en.md` (or `SKILL.ko.md`) into your Claude Code skills directory and rename it to `SKILL.md`, e.g.:

```
~/.claude/skills/store-screenshots/SKILL.md
```

Claude Code picks it up automatically whenever you ask it to produce App Store or Play Store screenshots.

## License

MIT — use it, fork it, adapt it to your own app pipeline.

# 🌑 UMBRA — Rotate the Shadow

A calm 3D shadow-silhouette puzzle. A light throws a glowing construct's **shadow** onto a back wall; a faint target silhouette is marked there. Drag to rotate the construct until its cast shadow snaps into the target — the closer you get, the brighter it glows. Then it doubles: later levels add a **second light**, so one object must cast **two different shadows that match two targets at the same time**. Inspired by the *aha* of Shadowmatic, free and instant in your browser.

**▶ Live:** https://quangle1997.github.io/umbra/ · part of [QUANG ARCADE](https://quangle1997.github.io/arcade/)

## Features
- **15 hand-authored levels** of rising difficulty — recognizable silhouettes (cat, key, bird, bottle, fish, sailboat, umbrella, rabbit, anchor…) plus abstract glyphs.
- **Honest match detection:** the shadow is rendered to an offscreen target and compared to the goal by **IoU** (intersection-over-union) — so any correct angle counts, and the "closeness" you feel is real.
- **Dual-light finale:** four late levels project two shadows at once that you must solve simultaneously (a single arrow that is also a disc; a tangle that is a cat *and* a key).
- **Calm by design:** no timers, no fail state. Soft shadows, subtle bloom, dust and light-shafts, a periwinkle palette and an ambient synth pad that swells as you get close.
- **Star rating** by match quality (1–3★), saved per level, with a level-select map and sequential unlocks.
- Controls: **drag** to rotate, **two-finger twist** / ⟲ ⟳ buttons / **Q–E** to roll; keyboard and touch.
- Audio: synthesized WebAudio pad + SFX, mute toggle. Single `index.html`, zero build, mobile-first portrait, deployed on GitHub Pages.

## Run locally
```bash
python3 -m http.server 8773
# open http://localhost:8773
```
(ES modules / importmap need an HTTP origin — `file://` won't work.)

See [`DOCS.md`](DOCS.md) for the full technical reference (level model, difficulty, scoring, IoU constants, and how to add a level).

---
Built by [QuangLe1997](https://github.com/QuangLe1997) · crafted with ♥ & Claude Code.

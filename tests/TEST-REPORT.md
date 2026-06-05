# UMBRA — Test Report (QA Evidence)

- **Date:** 2026-06-05
- **Build:** single `index.html` · Three.js r0.169 · served on local http (:8796)
- **Harness:** headless Chrome via puppeteer-core (`.qa-harness/umbra.mjs`) with SwiftShader GL flags
- **Viewports:** desktop 1280×800 · mobile 390×844
- **Console errors:** **0** across every run (favicon ignored)

## UI / flow + level coverage (visual evidence)

| ID | Category | Test | Viewport | Shot | Result |
|----|----------|------|----------|------|--------|
| T01 | UI | Main menu (title, stars total) | desktop | [01-menu-d.jpg](screenshots/01-menu-d.jpg) | ✅ PASS |
| T02 | UI | How-to-play onboarding | desktop | [02-how-d.jpg](screenshots/02-how-d.jpg) | ✅ PASS |
| T03 | UI | Level select — locks, stars, dual badge | desktop | [03-select-d.jpg](screenshots/03-select-d.jpg) | ✅ PASS |
| T04 | Level | L1 Halo — scrambled, single-axis | desktop | [04-play-halo-d.jpg](screenshots/04-play-halo-d.jpg) | ✅ PASS |
| T05 | Logic | Near-match glow — L3 Cat @ 87% | desktop | [05-near-cat-d.jpg](screenshots/05-near-cat-d.jpg) | ✅ PASS |
| T06 | Logic | Solved + 3★ — L4 Key @ 100% | desktop | [06-solved-key-d.jpg](screenshots/06-solved-key-d.jpg) | ✅ PASS |
| T07 | Level | L6 Bottle — two-axis | desktop | [07-bottle-d.jpg](screenshots/07-bottle-d.jpg) | ✅ PASS |
| T08 | Level | L9 Umbrella — three-axis + roll buttons | desktop | [08-umbrella-roll-d.jpg](screenshots/08-umbrella-roll-d.jpg) | ✅ PASS |
| T09 | Level | L11 Anchor — three-axis, hardest single | desktop | [09-anchor-d.jpg](screenshots/09-anchor-d.jpg) | ✅ PASS |
| T10 | Dual | L12 Disc & Bar — two shadows | desktop | [10-dual-disc-d.jpg](screenshots/10-dual-disc-d.jpg) | ✅ PASS |
| T11 | Dual | L13 Plus & Bar — dual near-match | desktop | [11-dual-near-plus-d.jpg](screenshots/11-dual-near-plus-d.jpg) | ✅ PASS |
| T12 | Dual | L14 Arrow & Disc — arrow + disc | desktop | [12-dual-arrowdisc-d.jpg](screenshots/12-dual-arrowdisc-d.jpg) | ✅ PASS |
| T13 | Dual | L15 Cat & Key — finale (cat + key) | desktop | [13-finale-catkey-d.jpg](screenshots/13-finale-catkey-d.jpg) | ✅ PASS |
| T14 | UI | Pause overlay (settings) | desktop | [14-pause-d.jpg](screenshots/14-pause-d.jpg) | ✅ PASS |
| T20 | Responsive | Menu mobile | mobile | [20-menu-m.jpg](screenshots/20-menu-m.jpg) | ✅ PASS |
| T21 | Responsive | L1 gameplay mobile | mobile | [21-play-halo-m.jpg](screenshots/21-play-halo-m.jpg) | ✅ PASS |
| T22 | Responsive | Near-match mobile (Cat) | mobile | [22-near-cat-m.jpg](screenshots/22-near-cat-m.jpg) | ✅ PASS |
| T23 | Responsive | Solved + 3★ mobile (Bird) | mobile | [23-solved-bird-m.jpg](screenshots/23-solved-bird-m.jpg) | ✅ PASS |
| T24 | Responsive | Dual-light mobile (Plus & Bar) | mobile | [24-dual-plus-m.jpg](screenshots/24-dual-plus-m.jpg) | ✅ PASS |
| T25 | Responsive | Level select mobile | mobile | [25-select-m.jpg](screenshots/25-select-m.jpg) | ✅ PASS |
| T26 | Responsive | Umbrella + roll buttons mobile | mobile | [26-umbrella-roll-m.jpg](screenshots/26-umbrella-roll-m.jpg) | ✅ PASS |

## Automated logic checks (no screenshot)

| ID | Test | Method | Result |
|----|------|--------|--------|
| T30 | **All 15 levels solvable & not pre-solved** | `umbra.mjs verify`: load each level, render silhouettes, compute IoU at solution pose vs scramble | ✅ PASS — every level **solved IoU = 1.000** on all lights; start IoU 0.07–0.61 (< 0.88, none pre-solved); all target masks non-degenerate (area > 2000 px) |
| T31 | **End-to-end play through real UI + input** | `umbra.mjs interact` | ✅ PASS — boot→menu, first-run How shown, Play→level 1; **pause/resume** work; real mouse **drag rotates** the construct & changes closeness; small drag does not false-solve; real loop **detects the solution** → solved overlay → **3★ awarded & saved** (`umbra.stars`); **Next** → level 2; reload **remembers stars** ("3 / 45"); **level 2 unlocked** in select |
| T32 | Console errors | console + pageerror listeners over all runs | ✅ PASS — 0 errors |

## Summary
- **Test cases:** 24 · **PASS:** 24 · **FAIL:** 0
- **Coverage:** menu / how / select / pause UI · all 4 difficulty tiers (single-axis, two-axis, three-axis+roll, dual-light) · near-match glow · solved + star award · every dual-light level incl. finale · scoring & persistence & unlock (T31) · all-levels solvability sweep (T30) · mobile + desktop.
- **Console errors:** 0
- **DOCS.md matches code:** ✅ (15 levels, axes/scramble per level, IoU/star constants in §14 verified against `CONFIG`)

## ✅ VERDICT: **PASS — GAME DONE**

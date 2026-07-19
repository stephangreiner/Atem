# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

**Atem** ("Physiologisches Atmen") is a tiny, dependency-free breathing PWA
(Progressive Web App). It guides breathing with a pulsing circle and haptic
(vibration) cues, and offers a minimalist minute-tick timer for training a
sense of elapsed time. It is designed primarily for small screens — a phone
(Pixel 4) and a smartwatch (Pixel Watch) — and works offline once installed.

The UI language is **German**. All user-facing text and code comments are in
German; keep it that way.

## Architecture

There is no build step, no framework, no package manager, no dependencies.
The entire application is a handful of static files served as-is (e.g. via
GitHub Pages).

| File                    | Role |
|-------------------------|------|
| `index.html`            | The whole app — HTML, CSS (in `<style>`), and JS (in `<script>`) inline in one file. |
| `sw.js`                 | Service worker: makes the app installable and offline-capable (network-first, cache fallback). |
| `manifest.webmanifest`  | PWA manifest (name, icons, colors, `display: standalone`, portrait). |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | App icons referenced by the manifest and `<head>`. |
| `README.md`             | Effectively empty (title only). |

### Two modes (toggled by the switch at the top)

1. **Minutentakt** (default, switch off/left) — `#timerPanel`. Vibrates on
   every full minute since start. Elapsed minutes are computed from
   `Date.now()`, **not** from counted ticks, so the cadence stays correct even
   if the browser throttles the timer. While running, the screen goes near-black
   (OLED-friendly) with only a dimmed minute count.
2. **Geführt** (guided, switch on/right) — `#guidedPanel`. A circle expands on
   inhale (`Einatmen`) and contracts on exhale (`Ausatmen`), driven by
   CSS `transform: scale()` transitions. Each phase fires a distinct vibration
   pattern (one long pulse for inhale, two short for exhale) so phases are
   distinguishable without looking. Inhale/exhale durations are user-adjustable
   sliders, persisted in `localStorage`.

### Key implementation details

- **Phases** are defined in the `phaseDefs` array; durations come from the
  live `config` object (`inhale`/`exhale` seconds).
- **Persistence**: guided durations are saved to `localStorage` under
  `atem_guided_config`. All `localStorage` access is wrapped in try/catch so a
  missing/blocked store falls back to defaults gracefully.
- **Screen Wake Lock** (`navigator.wakeLock`) keeps the screen on while either
  mode runs, and is re-requested on `visibilitychange`. It is optional —
  everything still works if the API is unavailable.
- **Vibration** uses `navigator.vibrate(...)`, always feature-detected.
- **Responsive scaling** is driven by the CSS custom property `--c` on
  `.stage`, with `@media (max-height: 480px), (max-width: 320px)` overrides for
  watch-sized displays. Running states use `body.running` / `body.timer-running`
  classes to declutter the UI (hide settings, enlarge the circle, dim the timer).
- **Service worker cache** is versioned via the `CACHE` constant in `sw.js`
  (currently `atem-v3`). See caching note below.

## Development workflow

- **Run/preview**: open `index.html` in a browser. For full PWA behavior
  (service worker, install, wake lock) serve over `http://localhost` or HTTPS,
  e.g. `python3 -m http.server` from the repo root, then open the printed URL.
  Service workers do **not** register from `file://`.
- **No build, lint, or test tooling** exists. There is nothing to compile or
  install. Verify changes by hand in a browser, ideally emulating a small
  viewport (DevTools device toolbar) to check watch/phone layouts.
- **Testing haptics/wake lock** requires a real device or a browser that
  supports the APIs; degrade gracefully when they're absent.

## Conventions

- **Language**: German for all UI strings and comments. Match the existing
  explanatory comment style (comments explain *why*, especially small-screen and
  device-specific tradeoffs).
- **Keep it single-file and dependency-free.** Do not introduce a framework,
  bundler, npm, or external CDN assets unless the user explicitly asks. New
  CSS/JS goes inline in `index.html`.
- **Small-screen first.** Any UI change must still fit a Pixel Watch. Prefer the
  `--c` variable and existing media queries over hard-coded sizes.
- **Feature-detect** browser APIs (`vibrate`, `wakeLock`, `serviceWorker`,
  `localStorage`) and wrap fallible calls so the app never breaks when an API is
  missing.
- **Bump the `CACHE` version in `sw.js`** (`atem-v3` → `atem-v4`, …) whenever you
  change `index.html` or other cached assets, so returning users get the update.
  (Fetch is network-first, but bumping guarantees stale caches are purged on
  activate.) If you add or remove a file, update the `ASSETS` list too.

## Git & branching

- Commit messages in this repo are short and descriptive; German or English are
  both used in history. Match the concise, imperative style.
- Work happens on feature branches merged into `main` via pull request (see
  `git log`). Do not commit directly to `main` unless asked.
- Only open a pull request when the user explicitly requests one.
